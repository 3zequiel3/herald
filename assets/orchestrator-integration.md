# Orchestrator Integration

herald is the **front of the funnel**. When an orchestrator agent is present, herald must slot into its routing cleanly — feeding the existing SDD flow without bypassing the orchestration protocol (per-phase models, skill paths, execution mode, artifact store). herald produces the idea; the orchestrator runs the machinery.

---

## Routing — when the orchestrator should call herald

Route to herald when the user wants **ideation or a proposal for something not yet specified**:

- "propose feature X", "how would I add Y", "armá una propuesta para…"
- "can we integrate A and B?", "se puede implementar el registro cruzado entre A y B"
- "what would it take to…", "evaluate if X is feasible", "fijate si conviene…"

Do **not** route to herald when:
- The idea is already consolidated and code-grounded → go straight to `/sdd-new`.
- The user wants to document what exists → that is `chronicle`.
- The user wants code written → herald is read-only and never implements.

Single system → herald runs **Ideate**; two or more → **Bridge**.

---

## Spec-flow detection & adapter (herald is not hard-wired to SDD)

herald hands off to whichever spec-driven flow Layer 0 detected — it does **not** assume `/sdd-new`. Resolve the target in priority order:

| Detected | Signal | Entry herald names in `next_action` |
|---|---|---|
| **SDD** | `sdd-*` skills / agents present | `/sdd-new <seed>` |
| **opsx / OpenSpec** | `opsx-*` / `openspec/` present | opsx's entry command |
| **Generic orchestrator** | an orchestrator that declares it consumes seeds | it routes the seed itself |
| **None** | none of the above | `inline-only` — no command emitted |

- **Multiple flows present** → prefer the one the orchestrator has configured; with no orchestrator, ask the user which.
- **An unrecognized flow** → herald does **not** invent its command. It degrades to `inline-only` and says so: *"a spec flow seems present but I don't recognize its entry — here's the seed to feed it manually."*
- herald **never auto-spawns** a flow's subagents (that bypasses model assignment, skill resolution, and the execution-mode/artifact-store questions the orchestrator owns). Standalone, it hands the user the exact command of the detected flow rather than firing it itself.

---

## The handshake (return contract)

herald runs as a skill in the main loop and emits the structured signal defined in [`seed-contract.md`](seed-contract.md) (`status`, `mode`, `seed`, `seed_strength`, `grounding_notes`, `next_action`). The orchestrator acts on `status` and fires the **detected** flow's entry (SDD shown below as the common case):

- **`seed-ready`** → fire `/sdd-new` with `seed` as the base prompt, then continue the normal SDD protocol: ask execution mode / artifact store / delivery strategy, resolve per-phase models, inject skill paths, and delegate to `sdd-explore` / `sdd-propose`. herald does **not** pre-empt these questions — that is the orchestrator's job.
- **`inline-only`** → no SDD flow detected; present herald's proposal and suggest installing the SDD skills.
- **`aborted`** → the user declined at the gate; do nothing downstream.

herald never spawns `sdd-*` subagents directly. Doing so would skip model assignment, skill resolution, and the execution-mode/artifact-store questions the orchestrator owns.

---

## Pasteable orchestrator instruction block

Drop this into the orchestrator's instruction file (e.g. `CLAUDE.md`) to wire herald into the chain:

```markdown
## herald — pre-spec ideation (front of funnel)

Route to herald when the user asks to ideate or propose something not yet spec'd
(single-system feature → Ideate; cross-system integration → Bridge). Do NOT route to
herald when the idea is already consolidated and code-grounded (go to /sdd-new), when
the user wants existing code documented (use chronicle), or when code must be written.

herald grounds in real code (read-only), separates fact from proposal, and STOPS at a
mandatory user approval gate. On approval it returns { status, mode, seed,
grounding_notes, next_action }:

- status: seed-ready  → fire /sdd-new with `seed` as the base prompt, then run the
                        normal SDD protocol (execution mode / artifact store / delivery,
                        per-phase models, skill paths, delegate to sdd-explore/sdd-propose).
                        Carry grounding_notes forward so the flow inherits the caveats.
- status: inline-only → no SDD flow detected; present the proposal, suggest installing SDD.
- status: aborted     → user declined at the gate; do nothing downstream.

Chain: ideation → herald → seed → /sdd-new → sdd-explore → sdd-propose → …
```

---

## Standalone (no orchestrator)

With no orchestrator/SDD present, herald runs the full flow and emits the seed + proposal inline (`status: inline-only`), recommending SDD installation. The return contract is identical; only the consumer changes (the user reads the seed instead of the orchestrator firing `/sdd-new`).
