# Grounding — establishing the factual base (source-agnostic, freshness-aware)

Grounding is how herald earns the right to call something a **fact**. Everything the proposal stands on must be traceable to a real source whose freshness herald can reason about. A "fact" from a stale document is not a fact — it is fiction with a citation, and it will poison the seed handed to the SDD flow downstream.

herald is **agnostic to who produced the material**: chronicle, a human, another tool. It cares about three things only:

1. **Citable** — can I point to where this came from? (see [`provenance.md`](provenance.md))
2. **Freshness-assessable** — can I tell whether it still matches reality?
3. **Trusted** — does the user or a verifiable ledger vouch for it?

---

## The four freshness states

Every grounding source resolves to exactly one state.

### `fresh`
A chronicle `knowledge-base/` is present AND its freshness ledger says the code has not changed since it was documented (git fast-path / fingerprint match).

→ **Use it directly.** Cheapest and most trusted path. Cite claims as `[kb · node]`. No code read needed.

### `stale`
The ledger (or a git fast-path comparison) says the code changed since it was documented, **OR the user declares the source out of date**.

→ **The code is now the source of truth.** Discard the stale claims for the affected slice and **re-ground from real code** (delegated read, below). Re-grounded claims carry `[code · …]` and the flag `⚠ stale→re-grounded`. Do not re-verify the entire KB — only the slice the proposal touches (token economy).

> The user's word wins. If the user says "the KB is outdated", there is no debate — the document is demoted and the idea is generated from the real code.

### `unverifiable`
A KB or docs exist but there is **no freshness ledger** — including any non-chronicle KB, loose docs, or a chronicle KB whose ledger files are missing.

→ **Do not assume fresh.** Absence of staleness evidence is not evidence of freshness. Detect this cheaply in Layer 0 (KB present, ledger absent) and ask the user in Layer 1:

> *"I found the KB but no freshness ledger; I can't verify it's current. Should I treat it as reliable, or spot-check it against the code?"*

- User says **reliable** → use it, but mark every claim `⚠ user-trusted` (it becomes the state below).
- User says **verify** or **I don't know** → run a bounded code spot-check (delegated) on the relevant slice. If code contradicts the KB, the code wins.

Recommend the clean fix without blocking: *"running chronicle would regenerate the ledger and give you verifiable freshness."*

### `user-trusted`
The user explicitly vouches for a source herald cannot auto-verify.

→ **Use it as factual**, but mark each claim `⚠ user-trusted`. This is not distrust — it is traceability. At the approval gate the user sees that the confidence comes from their word, not from a verified ledger.

---

## Code as source of truth — delegated reading

When grounding falls back to real code (`stale`, `unverifiable`-and-checking, or no KB at all), herald does **not** read code at scale itself. It **delegates** the read, in priority order:

1. **An existing exploration subagent** — `sdd-explore`, `opsx-explore`, or whatever explorer is installed. Reuses ecosystem machinery, runs in its own context, returns a compact result. The main session stays lean.
2. **Fallback** — herald's own bounded, read-only sub-agent, scoped strictly to the slice the proposal touches.

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
