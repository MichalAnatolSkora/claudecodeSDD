# TESTING.md — Order Export platform

> The test conventions the agent reads before writing tests. Part of `examples/order-export/` (illustrative). The practice behind it: [`testing-guide.md`](../../guides/testing-guide.md).

## Framework & layout

- **NUnit** for all tests. **Testcontainers** spins up a real MS SQL Server for integration tests.
- Tests live in `tests/` mirroring `src/` (`src/Delivery/SftpDeliveryService.cs` → `tests/Delivery/SftpDeliveryServiceTests.cs`).
- Name tests `MethodOrFeature_Scenario_ExpectedResult` (e.g. `Deliver_PartnerSftpDown_RetriesThenDeadLetters`).

## What's tested where

- **Unit** — pure logic: formatters (HDR / DTL rendering), the retry policy's backoff schedule, validation rules. No DB, no network.
- **Integration** — anything touching the DB or SFTP: repositories (against Testcontainers SQL) and delivery (against a stub SFTP server).

## Rules

- **Don't mock the database.** Use Testcontainers — mocking Dapper hides the bugs that matter.
- Mock only true externals: the partner SFTP endpoint, and the system clock (so backoff is testable without real waiting).
- Each acceptance criterion in a `spec.md` maps to at least one test — **the AC list is the test contract.**

## "Done"

A task is done when its `→ verify:` step passes **and** the break-the-code check holds: break the implementation on purpose and the relevant test must go red. A test that stays green is false confidence — fix the assertion, not the code.

---
*Illustrative TESTING.md. See [`templates/TESTING.md`](../../templates/TESTING.md) for the blank starting point.*
