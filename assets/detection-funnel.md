# Detection Funnel — cheap startup + mode router (3 layers)

herald's startup decides whether the session stays cheap. The rule, inherited from chronicle: **the filesystem tells you WHAT exists (detected); the user tells you WHAT they want (asked).** Never infer intent by reading code — it is impossible and expensive.

The three layers go cheapest → most expensive. Do not move up a layer until the previous one is exhausted. herald adds one thing chronicle does not have: **counting the systems in scope**, because that is what selects Ideate vs Bridge.

---

## Layer 0 — Filesystem footprint (≈0 tokens, WITHOUT reading source code)

Before asking anything and before opening a source file, scan the structure.

### Count the systems in scope (herald-specific)

A "system" is an independent project root. Signals that you have **more than one**:

| Signal | Tells you |
|---|---|
| Multiple sibling dirs each with their own manifest (`package.json`, `go.mod`, `pyproject.toml`…) | Distinct systems → candidate **Bridge** |
| The user names two or more systems ("A and B", "este y el otro sistema") | **Bridge** intent |
| Loose integration docs at a parent level (`*_INTEGRATION_*.md`, `*_SYNC_*.md`, `PLAN_*.md`) | Cross-system material → **Bridge** |
| A single project root, one manifest | One system → candidate **Ideate** |

> Reading a manifest's **dependencies section only** resolves the stack of each system. Do not read source files to "guess" the technology.

### Grounding-source footprint (per system)

For each system in scope, detect — without reading code — what factual material is available:

| Signal | Tells you |
|---|---|
| `knowledge-base/` present | A KB exists (possibly chronicle's). Candidate **fresh / unverifiable** source. |
| chronicle freshness ledger present (the `.json`/ledger chronicle writes) | Freshness is **auto-verifiable** → can detect `fresh` vs `stale`. |
| `knowledge-base/` present but **no ledger** | Freshness **unverifiable** → must ask the user or spot-check. |
| `docs/` with sources, loose `.md`, images, pasted prompts | Additional evidence to cite (`[doc]`, `[img]`). |
| Code present, no KB, no docs | Grounding must come from a **delegated code read**. |

### Stack via manifests (per system)

Same manifest→stack table as chronicle (`package.json` → Node/framework, `go.mod` → Go, `pyproject.toml` → Python, `composer.json` → PHP/Laravel, etc.). Detect by SIGNAL, not by reading source.

**Layer 0 output:** system count (→ proposed mode), per-system stack, per-system grounding-source state, available evidence — all without reading source code.

---

## Layer 1 — Confirm + ask ONLY the gaps

Show what was detected, propose the mode, and confirm. Example:

```
Detected: 2 systems — aura-ai (Next.js + Prisma) and forkmap (Go). Loose
integration docs at the parent (PLAN_INTEGRACION_BIDIRECCIONAL.md). aura-ai has a
knowledge-base/ but no freshness ledger; forkmap has neither.
→ Proposed mode: Bridge (cross-system). Correct?
```

Then ask **only** the questions no file can answer:

- **Intent / the idea**: what do you want to propose or evaluate? (the seed of everything)
- **The WHY**: motivation, the problem behind the request (code never contains intent).
- **Freshness** — only when a source is `unverifiable`: *"aura-ai's KB has no freshness ledger — should I trust it, or spot-check it against the code?"*
- **Trust** — if the user vouches for a non-chronicle KB: record it as `user-trusted`.

Everything Layer 0 resolved (system count, stack, which sources exist) is **confirmed**, not re-asked.

> **Non-invention:** confirming the mode with the user is not optional — Bridge vs Ideate changes the whole question battery. An unconfirmed inference that changes the proposal is marked `**Assumption:**`.

---

## Layer 2 — Deep, bounded reading (delegated)

Reading real source code happens **only when grounding requires it** (a source is `stale`, `unverifiable` and the user wants a check, or absent), and only for the **slice relevant to the proposal**. Never "read everything to understand". Prefer delegation to an existing explorer — full protocol in [`grounding.md`](grounding.md).

This is what makes proposing against a giant system cost nearly the same as a small one: you read the cheap footprint (Layer 0) and then only the slice the idea touches (Layer 2).

---

## Funnel summary

```
Layer 0 (manifests + structure + system count, free)
   → Layer 1 (confirm mode + ask only human gaps: intent, WHY, freshness)
      → Layer 2 (delegated bounded read ONLY of the slice the proposal touches)
```
