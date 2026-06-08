# ADR-015 — Delivery retry with exponential backoff and dead-letter

**Status:** Accepted · **Date:** 2026-02

> Illustrative ADR, part of `examples/order-export/`. Produced by the [`delivery-retry` trio](../../specs/2026-02-delivery-retry/). Format: [`adr-guide.md`](../../../../guides/adr-guide.md).

## Context

Partner SFTP endpoints fail transiently — timeouts, dropped connections, brief outages. Today a failed delivery is lost and noticed late (see the PRD problem statement). We need automatic recovery for transient failures and a safe, auditable end state for the ones we can't recover — without losing batches or double-sending files.

## Decision

Wrap each SFTP delivery in a **Polly** retry policy: exponential backoff (base 2s, doubling) with small jitter, up to a configurable max attempts (default 5). A pure `DeliveryErrorClassifier` decides what is retryable — transient errors retry, permanent ones (auth / permission) fail fast. On exhaustion or a permanent error the batch is **dead-lettered**: marked `FAILED` with a single alert log line, recorded in the existing `app.delivery_attempt` table (no new table). Backoff sleeps go through an injectable `TimeProvider`, so the schedule is testable without real waiting.

## Consequences

- **Good:** transient failures self-heal; every attempt is auditable; permanent failures surface fast instead of burning retries; timing is deterministic in tests.
- **Cost:** a new dependency (Polly); the retry policy and the classifier are now part of the delivery path and must track what partners actually return.
- **Follow-up:** per-partner tuning is supported via `app.partner` overrides; if partners later need wildly different policies, revisit whether config alone is enough.

## Alternatives considered

- **Hand-rolled retry loop** — fewer dependencies, but we'd reinvent backoff, jitter, and classification and test them ourselves. Rejected.
- **Message queue with redelivery** — robust, but heavy for ~12 partners on a daily cadence; adds infrastructure to run and reason about. Rejected for now; revisit at higher volume.
- **Log-and-drop, no dead-letter** — simplest, but loses batches and fails the audit goal. Rejected.

---
*Illustrative ADR. In a real repo it lives in `docs/adr/` and is referenced from `CLAUDE.md` Active decisions — exactly as it is here.*
