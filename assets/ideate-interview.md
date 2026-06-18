# Ideate Interview — single-system question battery

Mode Ideate is for proposing something new inside ONE system. The value of herald is not the proposal text — it is asking the questions that turn a vague wish into something a spec flow can actually build. Ask **one at a time**, prefer multiple choice, and stop after each to listen. Skip a question the user already answered (in the prompt or the grounding); never interrogate for the sake of it.

The goal of this battery is to fill, with the user, the gaps the code cannot answer — intent, boundary, and constraints — so consolidation has real material instead of guesses.

---

## Draft-first (Express track)

When the complexity router (see [`detection-funnel.md`](detection-funnel.md)) selected the **Express track** — a small, nameable change in a single system — invert the order. Do **not** run the full battery up front:

1. Ground the slice — from the cached `.ledger/` grounding if it is fresh, otherwise from a **single** delegated bounded read of just the named slice — then **draft the proposal immediately**, marking every inferred premise as `[assumption]`.
2. Ask **only the blocking gaps** — the questions whose answer would change the proposal's shape (usually just #1 problem/WHY and #6 MVP boundary). Skip anything grounding already settled.
3. Take the draft to the approval gate, where the user corrects the assumptions in one pass.

The gate and the fact/proposal split stay **mandatory** in Express — draft-first changes the *order* and the *depth* of questioning, never the discipline. Anything large, ambiguous, stale, or Bridge runs the full battery below.

---

## The battery

Ask in roughly this order; adapt to what grounding already revealed.

### 1. Problem / motivation (the WHY)
What problem does this solve, for whom? Code never contains intent — this is pure human knowledge and it anchors the whole proposal. A feature without a clear problem is a `[risk]` worth surfacing, not a proposal worth seeding.

### 2. Trigger / desired behavior
In one sentence: when X happens, the system should do Y. This becomes the spine of the consolidated idea and the first line of the seed.

### 3. Surface — modules and entities touched
Based on grounding, name the modules, routes, and entities the idea touches. Confirm with the user: *"this looks like it touches `auth/` and the `User` entity — anything else?"* Anything touched but not grounded becomes a delegated-read target or an `[open-q]`.

### 4. Fit with what exists
Does this extend an existing pattern, or introduce a new one? Reusing an existing pattern is cheaper and lower-risk; a new pattern needs justification. If the system already does something similar, cite it (`[code · …]`) and frame the proposal as an extension.

### 5. Scalability / load
Will this run once, per-request, per-user, or at high volume? The answer changes the design (sync vs queued, cached vs computed) and surfaces `[risk]`s early. Don't over-engineer — just size it.

### 6. MVP vs post-MVP boundary
What is the smallest version that delivers the value, and what is explicitly deferred? A sharp boundary keeps the seed focused so the SDD flow produces a tight first change instead of a sprawling one. Defer aggressively; mark deferred items as `[open-q]` or future scope.

---

## When to stop

Stop asking when you can state, in your own words: the problem, the trigger, the surface it touches, the smallest valuable version, and the main risk. If you can write those five with cited grounding underneath, you have enough to consolidate. More questions past that point spend the user's patience without improving the seed.
