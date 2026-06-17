# Bridge Interview — cross-system integration battery

Mode Bridge is for connecting two or more systems. It runs the full Ideate battery ([`ideate-interview.md`](ideate-interview.md)) for the feature itself, **plus** this integration battery. These questions are where most integrations quietly fail — source of truth, failure behavior, idempotency — and they almost never exist in a single-system proposal. Asking them now is what separates a real integration proposal from "just call B's API".

Ask **one at a time**, prefer multiple choice, stop and listen. Ground each system involved before this battery so the questions are concrete ("A's `User` vs B's `Account`") rather than abstract.

---

## The integration battery

### 1. What data crosses the boundary
Which entities/events actually move between systems? Name them concretely against grounded facts: *"A's `Accreditation` and `User` → B's `User`"*. Everything that crosses needs a mapping; everything that doesn't is out of scope — say so.

### 2. Source of truth (per shared entity)
For each shared entity, **which system owns the truth?** This is the single most important question in the whole battery. If both systems can write the same field, you have a conflict-resolution problem that must be designed, not discovered. Surface dual-ownership as a `[risk]` immediately.

### 3. Sync direction
One-way (A → B) or bidirectional (A ↔ B)? Bidirectional is far more than "twice one-way" — it reopens source-of-truth and loop-prevention as `[open-q]`s. Confirm explicitly; never assume.

### 4. Real-time vs batch
Must the change reflect immediately (event/webhook/API call) or is periodic batch acceptable? This drives the entire mechanism (event bus vs cron vs polling) and ties directly to the consistency answer below.

### 5. Behavior under partial failure
If the target system is **down** when the source changes, what happens? Lose it? Queue and retry? Block the source operation? There is no neutral default — an unanswered failure mode is the most expensive `[open-q]` to discover later, so pin it down now.

### 6. Idempotency
If the same event is delivered twice (retries, replays), does the target double-apply it? A safe integration must be idempotent; if it isn't, that is a `[proposal]` (add a dedup key / upsert) plus a `[risk]`.

### 7. Eventual consistency tolerance
How long can the two systems disagree before it's a problem — seconds, minutes, hours? The tolerance determines whether async/eventual is acceptable or strong consistency is required (much costlier). Get a number, even rough.

### 8. The contract between systems
What is the shape of the exchange — event payload, API request/response, shared table? Sketch it from grounded facts where both ends already exist; mark the new shape as `[proposal]`. This becomes the integration design's centerpiece.

---

## When to stop

Stop when you can state: what crosses, who owns each entity, the direction, real-time-or-batch, what happens on failure, whether it's idempotent, the consistency tolerance, and the rough contract shape. Those eight answers ARE the integration design. If any is missing, it is an `[open-q]` the seed must carry forward — do not paper over it with an assumption presented as fact.
