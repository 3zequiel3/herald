# `.ledger/` — Shared Ledger Contract (chronicle ⇄ herald)

> Status: contract v1 · Date: 2026-06-17
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

- **Fingerprint algorithm:** the normalized hash of a symbol's body (formatting and comments collapsed), so a reformat is **not** a change — only real logic is. This algorithm is the contract; it must stay identical across both skills. (chronicle already defines it in `assets/checker-spec.md §4`; herald reuses it verbatim.)
- **`ref`:** the git commit at fingerprint time, used as the baseline for the git fast-path staleness check. `null` when git is unavailable.

## Ownership & discipline (hard rules)

1. **Co-owned.** Either skill may create and extend `.ledger/` and `fingerprints.json`. Whoever runs first seeds it; whoever runs later extends it. The shared schema is what makes this safe.
2. **Tooling writes, the model never hand-edits.** Fingerprint values come from the hashing tool (`sha256sum`/`shasum` or equivalent), **never from the model's memory**. This is the same anti-fabrication rule chronicle enforces on its ledger, and it is what prevents drift. A fingerprint invented from memory is a defect.
3. **Private files stay private.** A consumer reads only `fingerprints.json`. It must ignore the other skill's private files.
4. **Gitignored.** `.ledger/` is tooling/infra state, not a deliverable — add it to the project's `.gitignore`. (For herald this is also consistent with its master rule: no proposal is persisted, only the cache that makes re-grounding cheap.)

## When chronicle was never used (herald-only)

herald does **not** depend on chronicle. If `.ledger/` does not exist:

1. herald **seeds it** the first time it grounds from real code: it creates `<project-root>/.ledger/`, computes fingerprints (via the hashing tool) for the symbols in the slice it read, and writes them to `.ledger/fingerprints.json` using this exact schema and algorithm.
2. herald writes its grounding snapshots to `.ledger/herald-grounding.json`.
3. herald ensures `.ledger/` is gitignored in that project.
4. On later runs, herald recalls from its own seeded ledger — same staleness machinery.

Because herald seeds with the **shared schema**, if chronicle is later installed and runs, it finds a valid `.ledger/fingerprints.json` already there and simply extends it (adding its own `verification.json`, `trace-map.json`). No conflict, no migration — they co-own from that point on.

**If writes are not possible** (read-only filesystem, or the user declines persistence): herald degrades to **in-session recall only** — it does not create `.ledger/`, and re-grounds each session. Correctness is unaffected (the staleness check always runs against the real repo); only the cross-session speedup is lost.

## Migration (chronicle legacy KBs)

Existing chronicle KBs store tooling state at `knowledge-base/.chronicle/`. The migration is one-time and must not break them:

- If legacy `knowledge-base/.chronicle/` exists and `.ledger/` does not → migrate once: move the files to `.ledger/`, and **derive `.ledger/fingerprints.json`** by lifting the fingerprints out of the old `verification.json`.
- If both exist → prefer `.ledger/`.
- If only legacy exists and migration can't run → keep reading it (backward-compat) and recommend migrating.

## Instruction brief for the chronicle agent

Paste this to the agent that maintains `chronicle` (herald does **not** edit chronicle):

> **Task: unify chronicle's tooling storage under a project-root `.ledger/`, with a shareable fingerprint map.**
>
> 1. Move tooling storage from `knowledge-base/.chronicle/` to **`.ledger/`** at the **project root** (siblings without a KB need it too).
> 2. Extract the fingerprint map to its own file at the **root** of `.ledger/`: **`.ledger/fingerprints.json`** (flat, NOT `.ledger/chronicle/fingerprints.json`). Use the schema in this contract (`version`, `ref`, `fingerprints{ "path#symbol": { fingerprint, ref } }`).
> 3. Your other private files (`verification.json`, `trace-map.json`, `checks.json`, `last-check.json`) also move to `.ledger/`, but the fingerprint map stays **separate** so a consumer reads only `fingerprints.json`.
> 4. **Do not change the normalized fingerprint algorithm** — it is the shared contract (same normalization, same SHA-256).
> 5. Keep the **ledger-ownership rule**: `fingerprints.json` and the ledger are written **only by the mechanical checker**, never hand-edited by the LLM.
> 6. **Migration:** if legacy `knowledge-base/.chronicle/` exists and `.ledger/` does not, migrate once and derive `fingerprints.json` from the old `verification.json`. Prefer `.ledger/` when both exist; keep reading legacy if migration can't run.
> 7. Add `.ledger/` to the project `.gitignore`.
> 8. Update all assets referencing the old paths (`checker-spec.md`, `verification.md`, `staleness.md`, anything mentioning `.chronicle`/`knowledge-base/.chronicle/`).
> 9. Bump chronicle's version and document the change + migration in the README.
> 10. Don't change standalone behavior: no git → no fast-path; nothing present → first run seeds the ledger.
