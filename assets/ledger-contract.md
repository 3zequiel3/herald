# `.ledger/` — Shared Ledger Contract (chronicle ⇄ herald)

> Status: contract v1 · Date: 2026-06-17 · aligned with chronicle v2.12
> The `.ledger/` folder is a **shared kernel** between `chronicle` and `herald`. This document is the single source of truth for its layout, schema, ownership, and migration. Both skills must honor it identically.

## Purpose

`chronicle` (notary) and `herald` (consultant) both reason about *the same thing*: the current state of the code, captured as **normalized fingerprints**. Sharing one fingerprint ledger means:

- herald reads freshness from a **stable, predictable path** without knowing who wrote it.
- A single fingerprint is the source of truth for "did the code change?" — no duplicate fingerprinting.
- Either skill can seed it; the other extends it.

The shared surface is kept **minimal on purpose** (only the fingerprint map). Everything else each skill keeps private. This is a Shared Kernel: powerful, but expensive to change — so it stays small.

## Location & structure

One `.ledger/` per **project root** (not nested in `knowledge-base/`, because a sibling skill may run on a project that has no KB).

```
<project-root>/.ledger/
├── fingerprints.json     # ★ SHARED — the only file a sibling skill reads
├── verification.json     # chronicle-private (verdicts)
├── trace-map.json        # chronicle-private (code-citation resolution)
├── checks.json           # chronicle-private (config)
├── last-check.json       # chronicle-private (last run)
├── registry.json         # chronicle-private (append-only stable-ID ledger)
└── herald-grounding.json # herald-private (grounding snapshots per slice)
```

Cross-system (herald Bridge mode) → one `.ledger/` per system root; herald reads/writes N of them.

## `fingerprints.json` schema (the shared surface)

```json
{
  "version": 1,
  "ref": "<git commit at last write, or null>",
  "fingerprints": {
    "path/to/file.ext#symbol": {
      "fingerprint": "<sha256 of the NORMALIZED symbol body>",
      "ref": "<git commit when computed, or null>"
    }
  }
}
```

- **Fingerprint algorithm:** the normalized hash of a symbol's body (formatting and comments collapsed), so a reformat is **not** a change — only real logic is. This algorithm is the contract; it must stay identical across both skills — defined in the next section so herald is self-sufficient.
- **`ref`:** the git commit at fingerprint time, used as the baseline for the git fast-path staleness check. `null` when git is unavailable.

## Version compatibility (hard rule)

`version` is the **first** thing a consumer reads, before interpreting any fingerprint. herald supports a known set of versions (currently `1`).

- **Recognized version** → read and interpret the fingerprints normally.
- **Missing, or newer than herald knows** (e.g. the writer bumped the algorithm to `2`) → herald does **NOT** interpret the fingerprints. It treats every symbol in scope as `unverifiable`, **re-grounds from code**, and surfaces a note: *"the ledger is vN; this herald understands v1 — re-grounding from code. Align versions (update herald, or re-run the writer) to restore fast freshness."*

A Shared Kernel's worst failure is a **silent misread**: comparing fingerprints produced by a different algorithm would let stale code pass as fresh. An unrecognized version is therefore never assumed compatible — it degrades to the safe, expensive path, **loudly**.

## Fingerprint normalization (so herald can seed identically)

herald must produce the same fingerprint chronicle would, or the ledger stops being interoperable. When herald seeds or refreshes an entry, normalize the symbol's body **before** hashing:

1. **Extract by symbol**, not by line range — the function/class/method body the citation points to.
2. **Strip comments and docstrings** (line and block).
3. **Collapse whitespace**: every run of spaces, tabs, and newlines becomes a single space; trim the ends.
4. **Touch nothing else**: identifiers, literals, and operator order are left exactly as-is. So a reformat or a new comment is **not** a change, but any real logic change is.
5. `fingerprint = sha256(normalized_body)`, computed with a SHA-256 hashing **library** from the runtime — **never** by shelling out to `sha256sum`/`shasum` with interpolated input (chronicle's security rule §5: argv-arrays only, defense against command injection via a crafted path/symbol), and **never** from the model's memory. Without a hashing library, fall back to a normalized structural signature, and say so.

This mirrors chronicle's algorithm (its `checker-spec.md §4`); this section is the copy herald keeps in sync, so it never needs to load a chronicle asset to seed correctly.

> **Validate against the shared golden.** chronicle ships the canonical fingerprint fixture at `assets/conformance/fingerprint/` (a `sample` + its `expected` normalized string and SHA-256). It is the cross-skill golden that **both** producer and consumer must pass — a one-byte difference makes the same symbol hash differently and the shared map silently lies. herald's seeding must reproduce that fixture's output exactly.

## Ownership & discipline (hard rules)

1. **Co-owned.** Either skill may create and extend `.ledger/` and `fingerprints.json`. Whoever runs first seeds it; whoever runs later extends it. The shared schema is what makes this safe.
2. **Tooling writes, the model never hand-edits.** Fingerprint values come from a SHA-256 hashing **library** (never by shelling out to `sha256sum`/`shasum`, and never from the model's memory). This is the same anti-fabrication rule chronicle enforces on its ledger, and it is what prevents drift. A fingerprint invented from memory is a defect.
3. **Private files stay private.** A consumer reads only `fingerprints.json`. It must ignore the other skill's private files.
4. **Gitignored.** `.ledger/` is tooling/infra state, not a deliverable — add it to the project's `.gitignore`. (For herald this is also consistent with its master rule: no proposal is persisted, only the cache that makes re-grounding cheap.)
5. **Concurrent-safe writes.** Two agents or terminals may write `.ledger/` at once. The writer must (a) write **atomically** — temp file + `rename` — so a reader never sees half-written JSON, and (b) **read-merge-write** — load the current `fingerprints.json`, union your new `path#symbol` entries, then write. Last-write-wins per key is safe because a fingerprint is a pure function of the code, so concurrent reads of the same symbol produce the same hash. A lost update can **never** cause a wrong freshness verdict — the staleness check always runs against the real repo — it only costs a redundant re-ground next run.

## When chronicle was never used (herald-only)

herald does **not** depend on chronicle. If `.ledger/` does not exist:

1. herald **seeds it** the first time it grounds from real code: it creates `<project-root>/.ledger/`, computes fingerprints (via a SHA-256 hashing library) for the symbols in the slice it read, and writes them to `.ledger/fingerprints.json` using this exact schema and algorithm.
2. herald writes its grounding snapshots to `.ledger/herald-grounding.json`.
3. herald ensures `.ledger/` is gitignored in that project.
4. On later runs, herald recalls from its own seeded ledger — same staleness machinery.

Because herald seeds with the **shared schema**, if chronicle is later installed and runs, it finds a valid `.ledger/fingerprints.json` already there and simply extends it (adding its own `verification.json`, `trace-map.json`). No conflict, no migration — they co-own from that point on.

**If writes are not possible** (read-only filesystem, or the user declines persistence): herald degrades to **in-session recall only** — it does not create `.ledger/`, and re-grounds each session. Correctness is unaffected (the staleness check always runs against the real repo); only the cross-session speedup is lost.

## Migration (chronicle legacy KBs) — chronicle's job, not herald's

Older chronicle KBs stored tooling state at `knowledge-base/.chronicle/`. Migrating it is **chronicle's** responsibility — its mechanical checker does it automatically (copy → verify → clean, never a blind `mv`) as of chronicle v2.12. herald **never touches chronicle's private files**.

What herald does when it meets a legacy layout:

- Legacy `knowledge-base/.chronicle/` exists, `.ledger/` does not → herald does **not** migrate it. It treats freshness as `unverifiable` (or re-grounds the slice from code) and recommends running chronicle, which migrates and seeds `.ledger/` itself.
- Both exist → read `.ledger/` (chronicle prefers it and leaves legacy untouched).
- Only `.ledger/` → the normal path.

## Interop status

chronicle ≥ **v2.12** implements this contract natively: the same project-root `.ledger/`, the same `.ledger/fingerprints.json` public projection, the same `checker-spec.md §4` algorithm + conformance fixture, version gating, union-merge atomic writes, and automatic legacy migration. **No action is pending on chronicle's side** — both skills already honor one shared ledger. A chronicle older than v2.12 predates the shared ledger; herald then falls back to `unverifiable` / re-ground and recommends updating chronicle.
