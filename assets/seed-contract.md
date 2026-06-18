# Seed Contract — the handoff into the SDD flow

Once the user approves at the gate, herald's deliverable is a **seed prompt**: a consolidated, code-grounded base prompt that an SDD/opsx flow can run with. The seed is the entire reason herald exists — it lets `sdd-explore` / `sdd-propose` start from a real, vetted idea instead of a blank page, and it preserves the fact/proposal split so the downstream flow inherits the caveats without re-investigating.

herald does **not** spawn a flow's subagents itself. It returns a structured signal; the consumer fires the **detected** flow's entry (`/sdd-new` for SDD, opsx's entry for opsx, etc.). See [`orchestrator-integration.md`](orchestrator-integration.md) for flow detection, routing, and the pasteable block.

---

## The return signal

On gate success herald emits this structure (to the orchestrator, or inline to the user when standalone). It is a structured-output **convention**, not a typed API: herald writes these fields as text and the consumer — an orchestrator agent, or the user — reads them. The discipline is that herald always emits them in a parseable shape.

```
status:          seed-ready | inline-only | needs-approval | aborted
mode:            ideate | bridge
seed:            <the seed prompt — see structure below>
seed_strength:   solid | partial | thin   (+ one line: why — see below)
grounding_notes: <freshness caveats: stale→re-grounded / unverified / user-trusted>
next_action:     "fire <detected-flow-entry> with <seed>"   (only when seed-ready; e.g. /sdd-new for SDD)
```

- `seed-ready` — a spec-driven flow was detected; the consumer should fire **its** entry (the one named in `next_action` — `/sdd-new` for SDD, opsx's entry for opsx, or a generic orchestrator routing the seed). herald never hard-codes `/sdd-new`.
- `inline-only` — no flow detected (or one herald doesn't recognize); herald has presented the proposal inline and recommends installing a flow. The seed is still emitted so the user can use it manually.
- `needs-approval` — the proposal and seed are ready, but the mandatory gate could not be satisfied (no human approver, e.g. a headless run). herald handed off **nothing**; a human must review and approve before the flow runs. Never auto-approve to avoid this state.
- `aborted` — the user declined at the gate; nothing downstream happens.

---

## Seed strength

`seed_strength` is **computed at the approval gate** and **read off the grounding coverage report** ([`grounding.md`](grounding.md) → Coverage report), not judged separately — so it stays mechanical, not a vibe. It is shown to the user at the gate (see [`consolidation.md`](consolidation.md)) and then emitted **verbatim** in the return signal, so the user and the downstream consumer see the same value:

- **solid** — every source in the coverage is `fresh` or `re-grounded from code`, no `⚠ unverified` / `⚠ user-trusted`, and ≤1 `[open-q]`.
- **partial** — any `⚠ unverified` / `⚠ user-trusted` source, OR a few `[open-q]`, OR a slice that was skipped.
- **thin** — the baseline rests largely on unverified sources or unread code, OR several `[open-q]`, OR a key slice was never grounded.

Always add one line of *why*, lifted from the coverage (e.g. `thin: baseline rests on an unverified KB; 3 open-qs unresolved`). It is a **signal, never a gate** — the user already approved; `seed_strength` only sets expectations downstream.

---

## Seed prompt structure

The seed is plain prose (it becomes a prompt), but it must carry the labeled structure so the downstream flow can read the fact/proposal line:

```
## Idea
<the consolidated idea — one paragraph>

## Factual baseline (grounded, cited)
<the relevant facts with their citations and any ⚠ flags>

## Proposal
<what to build — marked [proposal], with [assumption]s called out>

## Integration / implementation design
<Ideate: the change shape | Bridge: contract, mapping, sync, failure, idempotency>

## Risks, assumptions, open questions
<every [risk] / [assumption] / [open-q] — these MUST survive into the spec flow>

## Suggested first slice (MVP)
<the smallest valuable change, so the SDD flow produces a tight first change>
```

> Why keep the markers in the seed: `sdd-explore` will still do its own exploration, but starting from a seed that already says "this is verified against code" vs "this is an open design question" saves it from re-deriving everything and prevents a spec being built on an unmarked assumption. The seed is a head start, not a replacement for the SDD flow's own rigor.

---

## Standalone behavior

When no orchestrator / SDD flow is present, herald presents the seed inline (`status: inline-only`) and says something like: *"No SDD/opsx flow detected — here's the consolidated seed you can paste into `/sdd-new`, or use as-is. Installing the SDD skills would let me hand this off automatically."* The structure is identical; only the consumer changes.
