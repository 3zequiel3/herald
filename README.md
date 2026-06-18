<div align="center">

# 📯 herald

**Your pre-spec ideation consultant.**
Turns scattered ideas, docs, and real code into a vetted implementation proposal — then hands it off to your spec flow.

`reads code, never writes it` · `separates fact from proposal` · `single-system & cross-system` · `works standalone`

</div>

---

## ⚡ Install

```bash
# add it to the current project
npx skills add 3zequiel3/herald

# or globally, for every project
npx skills add 3zequiel3/herald -g
```

Works with Claude Code, Cursor, Codex, Copilot, Gemini, and any agent that reads the [Agent Skills](https://skills.sh) spec.

---

## 🤔 What is herald?

herald is the **ideation step that comes *before* you write specs**. You give it raw material — existing docs, images, prompts, `.md` files, and source code (read-only) — and it gives you back a **consolidated idea + implementation proposal**: feasibility, gap analysis, integration design, risks and assumptions.

It's the sibling — and the inverse — of [`chronicle`](https://github.com/3zequiel3/chronicle):

| | **chronicle** | **herald** |
|---|---|---|
| Role | 📚 Notary | 💡 Consultant |
| Job | Documents what **exists** | Proposes what **doesn't exist yet** |
| Invents | Nothing | Proposals — but **clearly marked as proposals** |

That last row is the whole point. herald is *allowed* to propose, so the one thing that keeps it honest is a **hard line between cited fact and marked proposal**. It never sells you a guess as a fact.

---

## 🎯 What it actually does

You point herald at one or more systems and ask a question like:

> *"System A handles accreditations and users; System B has its own users module. When someone registers in A, it doesn't reflect in B. Can we implement cross-registration between them?"*

herald reads both (read-only), interviews you on the gaps a machine can't answer, and produces:

- ✅ **A factual baseline** — what each system does *today*, every claim cited to real code/docs.
- 💡 **A proposal** — the integration design, marked speculative, with the assumptions spelled out.
- ⚠️ **Risks & open questions** — the failure modes (what if B is down? are events idempotent?) surfaced, not hidden.
- 🌱 **A seed** — a ready-to-run base prompt for your Spec-Driven Development flow.

It does **not** write code. It produces the thinking that makes the code worth writing.

---

## 🔧 How it works

```
  detect systems ──▶ pick mode (Ideate / Bridge)
        │
  ground in real code  (use a fresh KB if there is one, else read the
        │               relevant slice — delegated, read-only, bounded)
  interview you on the gaps  (the WHY, the boundary, the constraints)
        │
  consolidate  →  idea + feasibility + gaps + design + risks
        │
   ★ APPROVAL GATE ★  you see the full proposal, fact vs proposal split clean
        │             ├─ tweak it → re-consolidate
        │             └─ approve  → continue
        ▼
   hand off the seed  →  /sdd-new   (or just hand it to you, if no SDD flow)
```

**The approval gate is non-negotiable.** herald never hands anything downstream until *you* have seen the proposal and approved it. No black boxes.

---

## 🧭 Two modes

### 🟢 Ideate — one system
*"Propose a referral system for my marketplace."* herald grounds the relevant slice and asks the questions that matter: what problem, what it touches, does it scale, what's the smallest valuable version.

### 🔵 Bridge — two or more systems
*"Integrate A with B."* Everything Ideate does, **plus** the integration questions that quietly sink projects: **source of truth**, sync direction, real-time vs batch, behavior under partial failure, idempotency, eventual consistency, the contract between systems.

herald auto-detects how many systems are in scope and proposes the mode — you confirm.

---

## 🛡️ The discipline: fact vs proposal

Every line herald writes is labeled, so you never have to guess what it's standing on:

```
System A registers users via POST /api/register.   [code · src/auth/register.ts#register]
System B stores users in a `users` table.          [kb · 04_modelo]
The integration doc describes a nightly batch.      [doc · PLAN.md §Sync]  ⚠ unverified
[proposal] On registration in A, emit a `user.created` event that B consumes.
[assumption] B accepts externally-created user IDs without collision.
[risk] If A is down during B's write, the cross-record is lost unless queued.
[open-q] Real-time, or is eventual consistency acceptable?
```

Facts are cited. Proposals are marked and written in proposal voice. Stale or unverified sources are flagged. That labeling rides along into the seed, so your spec flow inherits the caveats instead of rediscovering them.

---

## 🧊 Freshness-aware grounding

herald is **source-agnostic** — it'll use a [`chronicle`](https://github.com/3zequiel3/chronicle) knowledge base, a hand-written one, or just the code. What it cares about is *trust*:

- **Fresh KB?** → use it, cheapest path.
- **Stale, or you say it's stale?** → **the code becomes the source of truth**; herald re-grounds the relevant slice from the real code (delegating to an explorer like `sdd-explore` when available).
- **No freshness ledger?** → it won't assume; it asks you or spot-checks.

Absence of staleness evidence is *not* evidence of freshness. A "fact" from an outdated doc is fiction with a citation — and herald treats it that way.

---

## 🔌 Standalone or orchestrated

**Standalone:** no SDD flow, no orchestrator? herald still runs the whole thing and hands you the seed inline.

**Orchestrated:** drop this into your orchestrator's instruction file and herald slots into the chain `ideation → herald → seed → /sdd-new → sdd-explore → …`:

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
                        normal SDD protocol; carry grounding_notes forward.
- status: inline-only → no SDD flow detected; present the proposal, suggest installing SDD.
- status: aborted     → user declined at the gate; do nothing downstream.

Chain: ideation → herald → seed → /sdd-new → sdd-explore → sdd-propose → …
```

---

## 🚫 What herald never does

- ✋ Modify, refactor, or write source code — **read-only, always.**
- ✋ Present a proposal, assumption, or risk as if it were a verified fact.
- ✋ Hand anything off without your approval at the gate.
- ✋ Spawn spec subagents itself — it returns a seed; the orchestrator runs the protocol.

---

## 🗂️ Structure

```
herald/
├── SKILL.md                          # lean router
└── assets/
    ├── detection-funnel.md           # detect systems + pick the mode (cheap first)
    ├── grounding.md                  # four freshness states, code-as-truth, delegated read
    ├── ideate-interview.md           # single-system question battery
    ├── bridge-interview.md           # cross-system integration battery
    ├── consolidation.md              # proposal structure + approval gate
    ├── seed-contract.md              # seed-prompt structure + return contract
    ├── orchestrator-integration.md   # routing + handshake + pasteable block
    ├── provenance.md                 # fact vs proposal taxonomy + freshness flags
    ├── edge-cases.md                 # conflicts, missing inputs, final self-check
    └── examples.md                   # one worked example per mode
```

---

## 📄 License

Apache-2.0.

<div align="center">
<sub>herald proposes. <a href="https://github.com/3zequiel3/chronicle">chronicle</a> documents. You decide.</sub>
</div>
