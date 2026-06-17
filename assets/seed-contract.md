# Seed Contract — the handoff into the SDD flow

Once the user approves at the gate, herald's deliverable is a **seed prompt**: a consolidated, code-grounded base prompt that an SDD/opsx flow can run with. The seed is the entire reason herald exists — it lets `sdd-explore` / `sdd-propose` start from a real, vetted idea instead of a blank page, and it preserves the fact/proposal split so the downstream flow inherits the caveats without re-investigating.

herald does **not** spawn SDD subagents itself. It returns a structured signal; the orchestrator fires `/sdd-new`. See [`orchestrator-integration.md`](orchestrator-integration.md) for routing and the pasteable block.

---

## The return signal

On gate success herald returns this structure (to the orchestrator, or inline to the user when standalone):

```
status:          seed-ready | inline-only | aborted
mode:            ideate | bridge
seed:            <the seed prompt — see structure below>
grounding_notes: <freshness caveats: stale→re-grounded / unverified / user-trusted>
next_action:     "run /sdd-new with <seed>"   (only when seed-ready)
```

- `seed-ready` — an SDD/opsx flow is available; the orchestrator should fire `/sdd-new` with `seed`.
- `inline-only` — no SDD flow detected; herald has presented the proposal inline and recommends installing SDD. The seed is still emitted so the user can use it manually.
- `aborted` — the user declined at the gate; nothing downstream happens.

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
