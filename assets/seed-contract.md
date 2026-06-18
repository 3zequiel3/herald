# Seed Contract — the handoff into the SDD flow

Once the user approves at the gate, herald's deliverable is a **seed prompt**: a consolidated, code-grounded base prompt that an SDD/opsx flow can run with. The seed is the entire reason herald exists — it lets `sdd-explore` / `sdd-propose` start from a real, vetted idea instead of a blank page, and it preserves the fact/proposal split so the downstream flow inherits the caveats without re-investigating.

herald does **not** spawn a flow's subagents itself. It returns a structured signal; the consumer fires the **detected** flow's entry (`/sdd-new` for SDD, opsx's entry for opsx, etc.). See [`orchestrator-integration.md`](orchestrator-integration.md) for flow detection, routing, and the pasteable block.

---

## The return signal

On gate success herald emits this structure (to the orchestrator, or inline to the user when standalone). It is a structured-output **convention**, not a typed API: herald writes these fields as text and the consumer — an orchestrator agent, or the user — reads them. The discipline is that herald always emits them in a parseable shape.

```
status:          seed-ready | inline-only | aborted
mode:            ideate | bridge
seed:            <the seed prompt — see structure below>
seed_strength:   solid | partial | thin   (+ one line: why — see below)
grounding_notes: <freshness caveats: stale→re-grounded / unverified / user-trusted>
next_action:     "fire <detected-flow-entry> with <seed>"   (only when seed-ready; e.g. /sdd-new for SDD)
```

- `seed-ready` — a spec-driven flow was detected; the consumer should fire **its** entry (the one named in `next_action` — `/sdd-new` for SDD, opsx's entry for opsx, or a generic orchestrator routing the seed). herald never hard-codes `/sdd-new`.
- `inline-only` — no flow detected (or one herald doesn't recognize); herald has presented the proposal inline and recommends installing a flow. The seed is still emitted so the user can use it manually.
- `aborted` — the user declined at the gate; nothing downstream happens.

---

## Seed strength

`seed_strength` is herald's honest read of how ready the seed is, so the consumer (and the user) can judge before firing the flow:

- **solid** — factual baseline is fresh and cited; few or no open questions; the proposal rests on verified ground.
- **partial** — usable, but some claims are `⚠ unverified` / `⚠ user-trusted`, or a few `[open-q]` remain.
- **thin** — the baseline rests largely on unverified sources or unread code, with several open questions; the flow should expect to do real exploration.

Always add one line of *why* (e.g. `thin: baseline rests on an unverified KB; 3 open-qs unresolved`). It is a **signal, never a gate** — the user already approved; `seed_strength` only sets expectations downstream.

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
