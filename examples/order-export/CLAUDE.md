# CLAUDE.md — Order Export platform

> The agent's instruction hub. **This example shows the project layer only.** The behavioral layer (sections 1–4: Think Before Coding / Simplicity First / Surgical Changes / Goal-Driven Execution) is copied verbatim from [`multica-ai/andrej-karpathy-skills`](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/CLAUDE.md) and is byte-identical to [`templates/CLAUDE.md`](../../templates/CLAUDE.md) — read it there. Part of `examples/order-export/` (illustrative).

## What this project is

A B2B order export platform. Per partner on a schedule, it exports `RDY` orders to a partner-specific file (XML or CSV) and delivers it over SFTP, with a full audit trail. Used by the operations team and partner-integration engineers. Success = batches delivered on time, every transmission auditable. See [`docs/prd/2026-01-order-export.md`](docs/prd/2026-01-order-export.md).

## Stack

- .NET 8, ASP.NET Core (minimal APIs)
- Dapper 2.x — **not** EF Core (ADR-007)
- MS SQL Server 2022
- Quartz.NET 3.x for scheduling
- Serilog for logging, with the `IBaseHandler<TSelf>` correlation contract
- FluentValidation for input validation
- NUnit + Testcontainers for integration tests — see [`TESTING.md`](TESTING.md)

## Conventions

- Repositories follow `src/Repositories/OrderLogRepository.cs`: SQL in `const string` at the top of the class; methods named by intent (`GetReadyOrders`, not `Get`); no business logic inside.
- Handlers are **pure** (no I/O); I/O lives in repositories and delivery.
- Formatters are pure functions of `(orders, partner) → bytes` — no DB, no network.
- SQL: keywords UPPERCASE, identifiers lowercase snake_case (`app.order_log`, `id`, `created_at`, `partner_id`).
- Every method touching the database logs the operation name plus the order id or batch id.
- Specs live in date-slug folders: `specs/YYYY-MM-feature-slug/`.

## Do NOT

- Do NOT propose Entity Framework, `DbContext`, or LINQ-to-SQL (ADR-007).
- Do NOT add MediatR or CQRS — handlers are called directly.
- Do NOT use `Console.WriteLine` for logging — Serilog only.
- Do NOT put I/O in a handler or a formatter.
- Do NOT create a new top-level folder without an ADR.

## Active decisions (Accepted ADRs)

- **ADR-001** — Repository per aggregate, hand-written SQL in constants
- **ADR-003** — Quartz for scheduling, config in `appsettings.json`
- **ADR-007** — Dapper for data access (not EF Core)
- **ADR-015** — Delivery retry with exponential backoff + dead-letter — [`docs/adr/ADR-015-delivery-retry-backoff.md`](docs/adr/ADR-015-delivery-retry-backoff.md)

> Only **ADR-015** is included in full in this example — it's the decision the `delivery-retry` trio produced. ADR-001/003/007 are listed to show that *Active decisions* is a **curated index of what's in force**, not a copy of every file in `docs/adr/`.

## Documentation map

| Task | Read first |
|------|------------|
| New SQL query | `DOMAIN.md`, `src/Repositories/OrderLogRepository.cs` |
| Export format change | `ARCHITECTURE.md` § Components, the formatter classes |
| Delivery / retry work | `docs/adr/ADR-015-delivery-retry-backoff.md` |
| Writing tests | `TESTING.md` |
| A new feature | `specs/` — one folder per feature |

## What to skip

- Spec folders under `specs/` for *shipped* features describe past work — read for context, not as current instructions.

## How to update this file

- Add a line to **Conventions** the third time you correct the agent on the same thing.
- Update **Active decisions** when an ADR is added or superseded.
- Trim any line older than 6 months you can't justify in one sentence.
- Don't edit the behavioral layer (it's upstream) — see the note at the top.

---
*Illustrative CLAUDE.md. Project layer only; the behavioral layer lives in [`templates/CLAUDE.md`](../../templates/CLAUDE.md).*
