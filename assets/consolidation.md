# Consolidation + Approval Gate

Consolidation is where herald turns the grounded facts and the interview answers into the proposal the user reviews. The approval gate is where the user sees it, split clean between fact and proposal, and decides. Nothing is handed off before the gate passes — this is non-negotiable (it is the difference between herald as a trustworthy consultant and herald as a black box).

---

## The consolidated proposal structure

Present these sections, scaled to the idea's complexity. Every factual line carries a citation (see [`provenance.md`](provenance.md)); every speculative line carries a marker. Do not blur the two.

### 1. Consolidated idea
One tight paragraph: the problem, the trigger, and the proposed behavior. This is the spine that becomes the seed.

### 2. Factual baseline (what the systems do today)
Bullet list of the relevant **facts**, each cited. For Bridge, group by system. This is the ground the proposal stands on — if it's thin or full of `⚠ unverified` flags, say so plainly; the user needs to know the foundation before judging the building.

### 3. Feasibility analysis
Can it be done with what exists? Separate the *existing capabilities* (factual, cited) from the *judgment that they suffice* (`[proposal]` / `[assumption]`). State the confidence honestly.

### 4. Gap analysis
What is missing between today and the idea. Current state cited as fact; missing pieces as `[proposal]` / `[open-q]`. This is the most actionable section for the SDD flow.

### 5. Integration / implementation design
- **Ideate:** the shape of the change — what to add, where it plugs into existing patterns (cite the patterns).
- **Bridge:** the contract, the data mapping, the sync mechanism, the failure/idempotency handling — the eight answers from the bridge battery, rendered as a design. New wiring is `[proposal]`; its premises `[assumption]`.

### 6. Risks & assumptions
Every `[risk]` and `[assumption]` collected, in one place, so the user can weigh them before committing. Never hide a risk to make the proposal look cleaner — a surfaced risk is the proposal doing its job.

### 7. Open questions
Every `[open-q]` the SDD flow must resolve. These travel into the seed so nothing is silently dropped.

---

## The approval gate (mandatory)

After presenting the proposal, stop and ask the user to decide. Make the fact/proposal split and the freshness flags impossible to miss — that visibility is the whole point of the gate.

Present it roughly as:

```
Here's the consolidated proposal. Before I hand anything to the SDD flow:
- ✅ FACTUAL (cited, grounded): … [with any ⚠ flags shown]
- 💡 PROPOSAL (what I'm suggesting, not yet built): …
- ⚠ RISKS / ASSUMPTIONS / OPEN QUESTIONS: …

Do you want to (a) approve and hand off, (b) change something, or (c) stop here?
```

- **Change requested** → incorporate and re-present (loop). Do not hand off a stale version.
- **Approved** → proceed to the seed (see [`seed-contract.md`](seed-contract.md)).
- **Stop** → return `status: aborted`; do nothing downstream.

> If the factual baseline rests heavily on `⚠ unverified` / `⚠ user-trusted` sources, flag it at the gate explicitly: *"note that most of this stands on an unverified KB — want me to spot-check against the code before we commit?"* The user must approve with eyes open.
