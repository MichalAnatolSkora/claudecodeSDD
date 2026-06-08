# tasks.md — Retry failed partner deliveries

> Execution checklist for [plan.md](plan.md). Part of `examples/order-export/` (illustrative). Ordered red→green; each step ends in `→ verify:`.

## Tasks

1. Add `RetryOptions` (max attempts, base delay) and bind it from `appsettings.json`. → verify: a unit test reads overridden options from config.
2. Write the DB migration adding nullable `max_attempts`, `base_delay_seconds` to `app.partner`. → verify: migration applies on a Testcontainers DB; columns exist and default to `NULL`.
3. Implement `DeliveryErrorClassifier` (transient vs permanent). → verify: unit test — timeout / reset = transient, auth = permanent (AC4, red→green).
4. Implement `DeliveryRetryPolicy` building the Polly policy from effective options + an injectable clock. → verify: unit test asserts the backoff schedule is 2s, 4s, 8s, 16s with a fake clock (AC1, AC7).
5. Resolve effective options: per-partner override, else global default. → verify: unit test — partner with override wins; partner without falls back (AC5).
6. `DeliveryAttemptRepository`: write one row per attempt; mark the batch `SENT` / `FAILED`. → verify: integration test (Testcontainers) — rows and batch status correct (AC2, AC3).
7. `SftpDeliveryService`: run the upload through the policy; on success mark `SENT` + `OK`, record each attempt, skip already-`SENT` orders. → verify: integration test (stub SFTP) — transient-then-success recovers, one file delivered, orders `SENT` (AC1, AC2, AC6).
8. Dead-letter path: on exhaustion or a permanent error, mark the batch `FAILED` + one alert log line. → verify: integration test — exhausted retries → `FAILED` + single alert; permanent error → immediate `FAILED`, no retries (AC3, AC4).
9. Idempotent re-delivery: re-running on a `FAILED` batch skips `SENT` orders, no duplicate rows. → verify: integration test re-runs delivery; no double-send, no duplicate attempt rows (AC6).
10. Break-the-code check on the new tests. → verify: flip the backoff doubling and disable the `SENT`-skip; the matching tests go red. Restore; suite green.

## Verification (every acceptance criterion has a task)

| AC | Covered by task(s) |
|----|--------------------|
| AC1 retry + backoff | 4, 7 |
| AC2 recovery recorded | 6, 7 |
| AC3 dead-letter on exhaustion | 6, 8 |
| AC4 fail fast on permanent error | 3, 8 |
| AC5 per-partner override | 5 |
| AC6 idempotent re-delivery | 7, 9 |
| AC7 deterministic timing | 4 |

---
*Illustrative tasks. After merge: append `STATUS: shipped (PR #N, date)` to the spec, then it freezes.*
