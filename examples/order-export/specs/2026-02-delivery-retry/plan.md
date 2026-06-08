# plan.md — Retry failed partner deliveries

> The *how* for [spec.md](spec.md). Part of `examples/order-export/` (illustrative). Next: [tasks.md](tasks.md).

## Technical decisions

- **Polly** builds the retry policy (exponential backoff + small jitter). Chosen over hand-rolled loops and a message queue — see [ADR-015](../../docs/adr/ADR-015-delivery-retry-backoff.md).
- **Transient vs permanent** classification lives in a pure `DeliveryErrorClassifier`; the policy retries only transient errors (AC1, AC4).
- **Dead-letter = existing tables.** Reuse `app.delivery_attempt` (one row per attempt; final row `FAILED`) and set the batch to `FAILED`. No new table — keeps the schema small (AC3).
- **Injectable clock.** `SftpDeliveryService` takes a `TimeProvider`; Polly's sleep uses it, so tests control time (AC7).
- **Per-partner overrides.** `max_attempts` and `base_delay_seconds` are nullable columns on `app.partner`; null → fall back to global `RetryOptions` from `appsettings.json` (AC5).
- **Idempotency.** Before sending, skip orders already `SENT`; the attempt write is keyed by `(batch_id, attempt_no)` (AC6).

## File structure

```
src/
├── Delivery/
│   ├── SftpDeliveryService.cs        # orchestrates upload + retry policy + recording
│   ├── DeliveryRetryPolicy.cs        # builds the Polly policy from effective options
│   └── DeliveryErrorClassifier.cs    # transient vs permanent (pure)
├── Config/
│   └── RetryOptions.cs               # global defaults (max attempts, base delay)
└── Repositories/
    └── DeliveryAttemptRepository.cs  # write attempts; mark batch SENT / FAILED
db/migrations/
└── 2026-02-add-partner-retry-overrides.sql   # nullable max_attempts, base_delay_seconds on app.partner
tests/
├── Delivery/
│   ├── DeliveryRetryPolicyTests.cs        # backoff schedule (unit, fake clock)
│   ├── DeliveryErrorClassifierTests.cs    # transient vs permanent (unit)
│   └── SftpDeliveryServiceTests.cs        # retry→recover, dead-letter, idempotency (integration, stub SFTP)
└── Repositories/
    └── DeliveryAttemptRepositoryTests.cs  # attempt rows + batch status (integration, Testcontainers)
```

## Constraints

- Dapper only — no EF (`CLAUDE.md` Do NOT, ADR-007).
- Logging via Serilog with the `IBaseHandler<TSelf>` contract; the dead-letter alert is one line, not per-attempt noise.
- No I/O in the classifier or the policy builder — those stay pure.
- No new top-level folder (`Delivery/`, `Config/`, `Repositories/` already exist).

---
*Illustrative plan. The file paths here are the only ones tasks.md may touch.*
