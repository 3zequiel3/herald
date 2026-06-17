# Examples — one few-shot per mode

Load only the section for the active mode. These show the *shape* of a good herald run — the fact/proposal discipline, not a script to copy verbatim.

---

## Ideate — single system

**User:** "Quiero que cuando un usuario se registra en aura-ai se le mande un mail de bienvenida. ¿Se puede?"

**herald (abridged):**

> Layer 0: one system (`aura-ai`, Next.js + Prisma). `knowledge-base/` present, no freshness ledger → `unverifiable`.
> Layer 1: "Found aura-ai's KB but no freshness ledger — trust it or spot-check the register flow against code?" → user: "spot-check it."
> Grounding (delegated read of the auth slice):
> - Registration handled by `register()`. `[code · src/auth/register.ts#register ~L20]`
> - No mailer is called in the flow today. `[code · src/auth/register.ts#register ~L20]`
> - A `MailService` exists but is only used for password reset. `[code · src/mail/MailService.ts#sendReset ~L14]`
>
> Battery → problem (onboarding), trigger (on successful register), surface (`auth/`, `MailService`), fit (reuse existing `MailService`), MVP (one templated email, no preferences).
>
> **Approval gate:**
> - ✅ FACTUAL: register flow at `register() ~L20`; `MailService` exists, used only for reset.
> - 💡 PROPOSAL: after a successful register, call `MailService` with a new `welcome` template.
> - ⚠ RISK: email send failure shouldn't block registration — `[proposal]` fire-and-forget / queued. `[open-q]` retwill on failure?
>
> User approves → seed-ready → orchestrator fires `/sdd-new`.

---

## Bridge — cross-system (the motivating case)

**User:** "Sistema A tiene acreditaciones y usuarios; Sistema B tiene usuarios. Si alguien se registra o acredita en A, no se refleja en B. Fijate si se puede implementar el registro/acreditación cruzado entre A y B."

**herald (abridged):**

> Layer 0: two systems → **Bridge**. A (`aura-ai`, Next.js + Prisma, KB + ledger → can verify), B (`forkmap`, Go, no KB). Loose `PLAN_INTEGRACION_BIDIRECCIONAL.md` at the parent.
> Layer 1: confirm Bridge. Ledger check on A: code unchanged → A's KB is `fresh`. B has no KB → delegate a bounded read of B's user module.
> Grounding:
> - A registers users via `POST /api/register`; emits nothing external. `[kb · 06_funcionalidades]` (fresh)
> - A's `Accreditation` belongs to a `User`. `[kb · 04_modelo]`
> - B stores users in a `users` table, created only via B's own signup. `[code · internal/users/service.go#Create ~L31]` (delegated read)
> - The integration doc proposes a nightly batch. `[doc · PLAN_INTEGRACION_BIDIRECCIONAL.md §Sync] ⚠ unverified`
>
> Integration battery → crosses: A.User (+ Accreditation) → B.User. Source of truth: **A owns identity** (user: confirmed). Direction: one-way A → B for now. Real-time vs batch: user wants near-real-time. Failure: must not lose the cross-record if B is down. Idempotency: B's `Create` is not idempotent today `[code · …#Create]` → `[risk]`. Consistency: minutes acceptable.
>
> **Approval gate:**
> - ✅ FACTUAL: A's register endpoint; A owns identity (user); B's `Create` (not idempotent); doc's batch idea is `⚠ unverified`.
> - 💡 PROPOSAL: on register/accredit in A, emit `user.upserted`; B consumes and **upserts** by A's user id.
> - 💡 PROPOSAL: add a dedup/upsert key in B so redelivery is safe.
> - ⚠ RISK: B down during A's write → `[proposal]` durable queue + retry. `[assumption]` B accepts external ids without collision. `[open-q]` bidirectional later?
>
> User: "approve, but make bidirectional an explicit open question" → re-present → approve → seed-ready → orchestrator fires `/sdd-new` with the seed (markers preserved).

---

Notice in both: every capability is cited before it's relied on, every new piece is `[proposal]`, every failure mode is `[risk]` or `[open-q]`, and freshness flags ride along. That is the discipline the examples exist to demonstrate.
