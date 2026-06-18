# Provenance — the line between fact and proposal

herald's master rule lives or dies here. Because herald is allowed to propose what does not exist, the ONLY thing that keeps it trustworthy is a rigid, visible separation between what is **factual** (traceable to a real source) and what is **speculative** (the agent's proposal). The user must be able to glance at any sentence and know which side it is on.

> Hard rule: **a factual claim without a citation is a defect.** Either it cites a source, or it is reframed as speculative (`[proposal]` / `[assumption]`) — there is no unmarked middle ground.

> Hard rule: **a proposal is never written as a fact.** "System B exposes a webhook" (if read from code) is factual. "System B could expose a webhook" is a proposal. Never collapse the second into the first.

This is the inverse of chronicle: chronicle only ever produces facts; herald produces both, and must label them.

---

## Citation format

Rendered as **inline code** (visually distinct and greppable):

```
[<type> · <anchor>]
```

### Factual citations (the claim is traceable to a real source)

| `type` | Meaning | `anchor` form |
|---|---|---|
| `code` | read from real code (delegated read / spot-check) | `path/file.ext#symbol ~Lnn` |
| `doc` | from a source document or a non-chronicle KB | `path/file.ext §section` |
| `kb` | from a chronicle KB node (carries its own provenance) | `NN_node` (e.g. `04_modelo`) |
| `img` | read from an image (diagram, screenshot, mockup) | `path/file.png` |
| `user` | declared by the user | `user` (optional: round/date) |

**`code` anchor:** the **symbol** is the source of truth (survives refactors); the line number (`~Lnn`) is a navigation hint, never canonical. A `code` citation is only valid if the symbol was actually read in the delegated grounding — never fabricated from memory. Because herald itself does not read the code, a delegated `code` citation must be **backed by the symbol's fingerprint** returned by the reader (cross-checked against `.ledger/` when present); a claim herald cannot back with a fingerprint is second-hand and downgrades to `⚠ unverified` (see [`grounding.md`](grounding.md) → Close the trust boundary).

### Freshness flags (appended to a factual citation)

A fact can be true-but-uncertain. Flags make that visible:

| Flag | Meaning |
|---|---|
| `⚠ unverified` | source could not be freshness-checked (no ledger) |
| `⚠ user-trusted` | the user vouched for it; not verified against code |
| `⚠ stale→re-grounded` | source was stale; this claim was re-derived from real code |

### Speculative markers (the claim is NOT a fact)

| Marker | Meaning |
|---|---|
| `[proposal]` | something herald is proposing to build; does not exist yet |
| `[assumption]` | a premise herald is assuming to make the proposal work; should be confirmed |
| `[risk]` | a way the proposal could fail or a cost it carries |
| `[open-q]` | a question that must be answered before/within the SDD flow |

> A proposal's uncertainty is already carried by the markers above: a low-confidence proposal is one surrounded by `[assumption]` / `[risk]` / `[open-q]`, which say *why* it is uncertain — concretely, not as a vibe. Overall seed confidence is `seed_strength` (mechanically derived from the coverage report; see [`seed-contract.md`](seed-contract.md)). There is no separate per-proposal confidence tag.

---

## Examples

```markdown
- System A registers users via `POST /api/register`. [code · src/auth/register.ts#register ~L20]
- System B stores users in a `users` table. [kb · 04_modelo]
- The integration doc describes a nightly batch sync. [doc · PLAN_INTEGRACION.md §Sync] ⚠ unverified
- System A is the source of truth for credentials. [user]
- [proposal] On registration in A, emit a `user.created` event that B consumes.
- [assumption] B can accept externally-created user IDs without collision.
- [risk] If A is down during B's registration, the cross-record is lost unless queued.
- [open-q] Is the sync expected to be real-time or is eventual consistency acceptable?
```

> Notice: every factual line points to a source; every speculative line is marked and is written in proposal voice ("emit", "could", "if"). A reader never has to guess.

---

## What carries what

- **Feasibility analysis** — mixes both: the *capabilities that exist* are factual (cited); the *judgment that they suffice* is `[proposal]` or `[assumption]`.
- **Gap analysis** — the *current state* is factual (cited); the *missing pieces* are `[proposal]` / `[open-q]`.
- **Integration design** — the *touchpoints that exist today* are factual; the *new wiring* is `[proposal]`, its premises `[assumption]`, its failure modes `[risk]`.
- **Risks / assumptions** — always speculative by nature; never dressed as fact.

---

## Why this is the backbone

The seed handed to the SDD flow inherits these markers verbatim. That means `sdd-explore` / `sdd-propose` downstream can tell, with zero re-investigation, which parts of the idea are already verified against code and which are open design decisions. A blurred fact/proposal line at this stage compounds into a spec built on sand. Keeping the line sharp here is the single highest-leverage discipline in the whole skill.
