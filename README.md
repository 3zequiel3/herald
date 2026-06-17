# herald

**Pre-spec ideation consultant.** herald turns raw material — existing docs, images, prompts, `.md` files, and source code (read-only) — into a **consolidated idea + implementation proposal**, then hands it off as a seed prompt to a Spec-Driven Development flow (SDD / opsx).

It is the sibling and predecessor of [`chronicle`](../chronicle):

- **chronicle** is a *notary* — it documents only what exists, inventing nothing.
- **herald** is a *consultant/architect* — it MAY propose what does not exist yet, but always separates the **factual** (what systems do today, cited) from the **speculative** (the proposal, with assumptions and risks). A proposal is never presented as a fact.

herald sits at the front of the funnel: it feeds `sdd-explore` / `sdd-propose` with an idea already grounded in the real code.

## What it does

- **Two modes**, chosen by how many systems are in scope:
  - **Ideate** — propose a feature/improvement for a single system.
  - **Bridge** — design a cross-system integration (sync, cross-registration, shared entity) between two or more projects.
- **Source-agnostic, freshness-aware grounding** — uses a chronicle KB if present and fresh; if a source is stale, missing a freshness ledger, or the user declares it outdated, the **real code becomes the source of truth** and herald re-grounds via a delegated, read-only bounded read (preferring `sdd-explore` / `opsx-explore`).
- **Mandatory user approval gate** — herald never hands anything to the SDD flow without showing you the proposal first, split clearly between cited fact and proposal.
- **Standalone** — works with or without SDD/opsx skills or an orchestrator. The handoff is the preferred path, never a requirement.

## What it never does

- Modify, refactor, or write source code (read-only, always).
- Persist artifacts by default (conversational + handoff only).
- Present a proposal, assumption, or risk as if it were a verified fact.
- Spawn SDD subagents directly (it returns a seed; the orchestrator runs the protocol).

## The flow

```
detect systems → propose Ideate/Bridge → ground (KB if fresh, else delegate code read)
  → question battery → consolidate (idea + feasibility + gaps + design + risks)
  → ★ approval gate ★ → seed → hand off to /sdd-new   (or inline if no SDD flow)
```

## Orchestrator integration

herald returns `{ status, mode, seed, grounding_notes, next_action }`. On `seed-ready` the orchestrator fires `/sdd-new` with the seed and continues its normal protocol. To wire it in, paste this into your orchestrator's instruction file:

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

## Structure

```
herald/
  SKILL.md                         # lean router
  assets/
    detection-funnel.md            # Layer 0/1/2, system counting, mode router
    grounding.md                   # four freshness states, code-as-truth, delegated read
    ideate-interview.md            # single-system question battery
    bridge-interview.md            # cross-system integration battery
    consolidation.md               # proposal structure + approval gate
    seed-contract.md               # seed-prompt structure + return contract
    orchestrator-integration.md    # routing + handshake + pasteable block
    provenance.md                  # factual vs speculative taxonomy + freshness flags
    edge-cases.md                  # conflicts, missing inputs, final self-check
    examples.md                    # one few-shot per mode
```

## License

Apache-2.0.
