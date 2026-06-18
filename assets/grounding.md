# Grounding — establishing the factual base (source-agnostic, freshness-aware)

Grounding is how herald earns the right to call something a **fact**. Everything the proposal stands on must be traceable to a real source whose freshness herald can reason about. A "fact" from a stale document is not a fact — it is fiction with a citation, and it will poison the seed handed to the SDD flow downstream.

herald is **agnostic to who produced the material**: chronicle, a human, another tool. It cares about three things only:

1. **Citable** — can I point to where this came from? (see [`provenance.md`](provenance.md))
2. **Freshness-assessable** — can I tell whether it still matches reality?
3. **Trusted** — does the user or a verifiable ledger vouch for it?

---

## The shared `.ledger/` (freshness + recall)

herald and `chronicle` share one well-known location per system root: **`.ledger/`**. The single file herald depends on is **`.ledger/fingerprints.json`** — a tooling-written map of `path#symbol → normalized fingerprint (+ git ref)`. It is the *shared kernel* between the two skills: chronicle writes it when it documents/verifies; herald reads it to assess freshness and to power recall.

The contract herald relies on:

- **Location:** `<system-root>/.ledger/fingerprints.json` — flat, at the root of `.ledger/`. A stable, predictable path; herald never needs to know who wrote it.
- **Ownership & fingerprint:** the map is **written by tooling, never hand-edited by the model**; herald only ever obtains a fingerprint from the hashing tool, never from memory. The normalized-hash algorithm and the ownership rules live in **one** place — [`ledger-contract.md`](ledger-contract.md) — so they cannot drift between the two skills.
- **herald's own state:** herald stores its grounding snapshots at `<system-root>/.ledger/herald-grounding.json` (its extracted facts per slice, each pointing at the fingerprint keys it relied on). This is tooling/infra state — gitignored, never a deliverable. It is consistent with the master rule: no proposal is persisted, only the cache that makes re-grounding cheap.
- **Standalone:** if `.ledger/` is absent, herald may seed it (compute fingerprints for the slice it reads); if writes are not allowed, it degrades to in-session recall only. Cross-system → one `.ledger/` per system root.

> Full schema (`fingerprints.json`), ownership rules, the herald-seeds-it-when-chronicle-is-absent path, and migration: [`ledger-contract.md`](ledger-contract.md). Load it whenever herald has to **read or seed** the ledger.

---

## The four freshness states

Every grounding source resolves to exactly one state.

### `fresh`
`.ledger/fingerprints.json` is present AND a cheap staleness check (git fast-path / fingerprint match) says the symbols in the relevant slice have not changed since they were fingerprinted.

→ **Use the cached grounding directly.** Cheapest and most trusted path: reuse the facts from `herald-grounding.json`, or cite the chronicle KB as `[kb · node]`. No code read needed.

### `stale`
The ledger (or a git fast-path comparison) says the code changed since it was documented, **OR the user declares the source out of date**.

→ **The code is now the source of truth.** Discard the stale claims for the affected slice and **re-ground from real code** (delegated read, below). Re-grounded claims carry `[code · …]` and the flag `⚠ stale→re-grounded`. Do not re-verify the entire KB — only the slice the proposal touches (token economy).

> The user's word wins. If the user says "the KB is outdated", there is no debate — the document is demoted and the idea is generated from the real code.

### `unverifiable`
A KB or docs exist but there is **no `.ledger/fingerprints.json`** to check them against — including any non-chronicle KB, loose docs, or a chronicle KB whose ledger is missing.

→ **Do not assume fresh.** Absence of staleness evidence is not evidence of freshness. Detect this cheaply in Layer 0 (KB present, `.ledger/` absent) and ask the user in Layer 1:

> *"I found the KB but no freshness ledger; I can't verify it's current. Should I treat it as reliable, or spot-check it against the code?"*

- User says **reliable** → use it, but mark every claim `⚠ user-trusted` (it becomes the state below).
- User says **verify** or **I don't know** → run a bounded code spot-check (delegated) on the relevant slice. If code contradicts the KB, the code wins.

Recommend the clean fix without blocking: *"running chronicle would regenerate the ledger and give you verifiable freshness."*

### `user-trusted`
The user explicitly vouches for a source herald cannot auto-verify.

→ **Use it as factual**, but mark each claim `⚠ user-trusted`. This is not distrust — it is traceability. At the approval gate the user sees that the confidence comes from their word, not from a verified ledger.

---

## Grounding recall (don't re-discover what hasn't changed)

For teams working the same systems every day, re-grounding from scratch on every run is wasted work. herald recalls via the shared `.ledger/`:

1. **Recall first.** For the slice the proposal touches, look its symbols up in `.ledger/fingerprints.json` and in herald's `herald-grounding.json`.
2. **Cheap staleness check.** Git fast-path (`git diff <ref> --name-only`, which also catches uncommitted changes) or a per-symbol fingerprint compare. Only the changed files are candidates.
3. **Reuse or re-ground.** Symbols whose fingerprint matches → reuse the cached facts **for free** (no read). Symbols that changed or were never seen → re-ground only those (delegated read), then update the fingerprint (via the hashing tool) and the snapshot.

This is the same fingerprint machinery chronicle uses for staleness — herald just consumes it. Without git, fall back to content-hashing the slice; without any `.ledger/`, re-ground (and optionally seed the ledger). **Correctness never depends on the cache:** the staleness check runs against the real repo, so herald never grounds on something stale without noticing — the cache only removes redundant reads, never adds blind trust.

---

## Code as source of truth — delegated reading

When grounding falls back to real code (`stale`, `unverifiable`-and-checking, or no KB at all), herald does **not** read code at scale itself. It **delegates** the read, in priority order:

1. **A read-only exploration subagent** — the generic `Explore` agent, or herald's own bounded read-only sub-agent, scoped strictly to the slice the proposal touches. It runs in its own context and returns a compact result, so the main session stays lean. This is the default, always-available path.
2. **Reuse an SDD explorer only when it fits** — if `sdd-explore` / `opsx-explore` is installed AND can take an ad-hoc, change-less scope, prefer it (it reuses ecosystem machinery). Do **not** assume it does: those are SDD *phase* agents with their own lifecycle, so lean on them only when they genuinely accept a one-off read; otherwise stay with option 1.

Why this matters: **double win.** Token economy (isolated context, compact return — the main session does not inflate) and reuse (no reinvented explorer; herald leans on the same tooling the SDD flow already ships). The fallback is what keeps herald standalone when no explorer exists.

Always scope the delegated read to the **slice relevant to the proposal** — the entities, routes, or modules the idea actually touches. Never "read the whole system to understand it".

---

## Non-chronicle KBs

Any reliable KB is a valid grounding source. chronicle is only *preferred* because it ships two things for free: provenance citations already inside the KB, and a freshness ledger. It is never required — this is what keeps herald genuinely standalone.

Two consequences for a non-chronicle KB:

- It usually has **no freshness ledger** → it lands in `unverifiable` by default (ask the user / spot-check).
- It usually has **no internal citations** → herald treats it as a **document** (`[doc · file]`), not as pre-cited facts it can forward blindly. herald cannot inherit citations that do not exist. For higher confidence on a specific claim, spot-check that claim against the code.

---

## Coverage report (always)

After grounding, state plainly what happened — never cut off silently:

- Which sources were used and in which state (`fresh` / `re-grounded from code` / `user-trusted` / `unverified`).
- Which slices were read from code (and via which path: explorer vs fallback sub-agent).
- What was **not** grounded and why (out of scope, unavailable, user deferred).

This report travels into the seed as `grounding_notes` so the downstream SDD flow inherits the caveats.
