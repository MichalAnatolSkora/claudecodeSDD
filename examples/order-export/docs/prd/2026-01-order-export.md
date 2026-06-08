# PRD — Order Export platform

> Product requirements, humans-only (the agent reads specs, not this). Lean format — see [`prd-guide.md`](../../../../guides/prd-guide.md). **Illustrative example**, part of `examples/order-export/`. The product is fictional.

**Status:** Accepted · **Date:** 2026-01

## Problem

We deliver daily order data to ~12 partner banks and logistics providers (e.g. `acme-bank`). Today an operator runs a manual export per partner, file formats vary, and a failed SFTP upload is often discovered hours later — sometimes by the partner. We keep no auditable record of what was sent, when, or whether it arrived. As we onboard more partners the manual process doesn't scale, and the missing audit trail is becoming a compliance problem.

## Users

- **Operations team** — trigger and monitor exports, chase failures. *Primary.*
- **Partner-integration engineers** — onboard a new partner, add or adjust a file format.
- *(Indirect)* **Partners** — receive the files; never touch the system.

## Goals / success criteria

1. **On-time delivery** — 99% of daily partner batches delivered within the agreed window, automatically.
2. **Auditable** — every batch has a record: which orders, which partner, when sent, success/failure, how many attempts.
3. **Resilient to transient failure** — a partner's SFTP being briefly down must not lose a batch; it retries and recovers with no manual work.
4. **Fast onboarding** — a new partner (format + schedule + credentials) is added in under a day, with no code change for a format we already support.

## Non-goals (v1)

- No self-service partner portal.
- No real-time / streaming export — batch only.
- No parsing of partner-side acknowledgements (we confirm *delivery*, not downstream processing).

## Solution sketch

A scheduled .NET service that, per partner on a schedule: queries the orders due for export, renders them to the partner's format (XML or CSV), writes one batch file, delivers it over SFTP, and records the outcome. Operators can see batch status and re-trigger. See [`ARCHITECTURE.md`](../../ARCHITECTURE.md).

## Feature breakdown (slices → specs)

Vertical slices, walking-skeleton first. Each becomes one `specs/YYYY-MM-slug/` trio.

| # | Feature | Serves | Size | Depends on |
|---|---------|--------|------|------------|
| 1 | Walking skeleton: export one partner's `RDY` orders to XML, deliver by SFTP, record the batch | G1, G2 | M | — |
| 2 | CSV format option per partner | G4 | S | 1 |
| 3 | **Retry failed deliveries (backoff + dead-letter)** | **G1, G3** | **M** | 1 |
| 4 | Per-partner schedule configuration | G4 | S | 1 |
| 5 | Operator batch-status view | G2 | M | 1 |

This example carries **feature 3** through the full trio (`spec → plan → tasks`) and the ADR it produced — see [`specs/2026-02-delivery-retry/`](../../specs/2026-02-delivery-retry/) and [`ADR-015`](../adr/ADR-015-delivery-retry-backoff.md).

> This list is the slice **snapshot** captured when the PRD was accepted — it freezes with the PRD. Which features are shipped or in flight lives in `specs/` and your tracker, **not here**: a PRD never takes per-feature status edits (the "freeze after v1" rule — see [`prd-guide.md`](../../../../guides/prd-guide.md)).

## Risks / open questions

- Partner SFTP reliability varies — the retry policy must be tunable per partner. *(Resolved in feature 3 / ADR-015.)*
- Order volume spikes at month-end — batch file sizes must stay within partner limits. *(Tracked; out of scope for feature 3.)*

---
*Illustrative PRD for the SDD examples set.*
