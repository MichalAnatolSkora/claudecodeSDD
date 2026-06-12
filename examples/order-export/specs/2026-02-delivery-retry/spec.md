# spec.md — Retry failed partner deliveries

> Feature 3 from the [PRD](../../docs/prd/2026-01-order-export.md). Part of `examples/order-export/` (illustrative). Trio order: **this → [plan.md](plan.md) → [tasks.md](tasks.md)**.

**Status:** Active · **Serves:** PRD goals G1 (on-time delivery), G3 (resilience)

## Goal

When a partner's SFTP delivery fails for a transient reason, the batch must retry automatically with backoff and recover with no manual work. When it genuinely can't be delivered, the batch is **dead-lettered** — marked `FAILED` and flagged for an operator — never silently lost. Today a failed upload is lost and discovered late; this closes that gap.

## Acceptance criteria

The test contract — every item maps to at least one test (see [TESTING.md](../../TESTING.md)).

- [ ] AC1: **Retry with backoff** — a delivery failing with a *transient* error is retried up to the configured max attempts (default **5**) with exponential backoff (base **2s**, doubling: 2s, 4s, 8s, 16s).
- [ ] AC2: **Recovery is recorded** — when a retry succeeds, the batch's orders are marked `SENT`, one `app.delivery_attempt` row is written per attempt (each failure `FAILED`, the success `OK`), and the partner receives exactly one file (no duplicate from earlier attempts).
- [ ] AC3: **Dead-letter on exhaustion** — if every attempt fails, the batch is marked `FAILED` and exactly one alert-level log line is emitted with the `batch_id` and partner.
- [ ] AC4: **Fail fast on permanent errors** — a non-transient error (auth / permission rejected) is **not** retried: it dead-letters immediately and logs the reason.
- [ ] AC5: **Per-partner override** — a partner may override `max_attempts` and `base_delay_seconds` in `app.partner`; with no override, the global default applies.
- [ ] AC6: **Idempotent re-delivery** — re-running delivery for a dead-lettered batch never re-sends orders already `SENT` and never creates duplicate rows.
- [ ] AC7: **Deterministic timing in tests** — backoff is driven by an injectable clock, so the schedule is verified without waiting in real time.

## Out of scope

- Operator UI and notification emails on dead-letter (just the log line + `FAILED` status here).
- Changing what counts as "due for export" — this is delivery only.
- Month-end file-size limits (separate concern, see PRD risks).

## Open questions

- [x] Which errors are transient? → timeouts, connection resets, and transient SFTP failures. Auth / permission are permanent. *(→ [ADR-015](../../docs/adr/ADR-015-delivery-retry-backoff.md))*
- [x] Retry library or hand-rolled? → **Polly**. *(→ ADR-015)*
- [x] Where do dead-letters live? → reuse `app.delivery_attempt` (final attempt `FAILED`) + batch `FAILED`; no new table.

---
*Illustrative spec. A shipped spec freezes; later changes get a new spec.*
