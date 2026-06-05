# TESTING.md

> Copy to your repo root. These are the conventions the agent reads *before* writing tests —
> so its tests match your project instead of its defaults. Keep it short; fill the brackets.

## Framework & runner

- Test framework: [e.g. xUnit / Jest / pytest]
- Run the suite: [e.g. `dotnet test` / `npm test` / `pytest`]
- [Anything fixed — assertion library, test config, CI command]

## Where tests live

- [Folder + naming, e.g. `tests/Api.Integration/`, `*.Tests.cs`, `*_test.py`]
- Unit vs integration are separated by: [folders / traits / markers]

## Unit vs integration boundary

- **Unit** — [pure logic, no I/O]
- **Integration** — [what needs the real thing, e.g. "repository tests run against Testcontainers Postgres, not mocks"]

## Mocking rules

- Mock: [the edges — outbound HTTP, the clock, third-party SDKs]
- **Never mock:** the type under test, or the database (use a real one in integration tests)

## Test data & fixtures

- Build test data with: [builders / factories / fixtures], shared ones in [location]
- Each test is isolated — no shared mutable state; [reset strategy]

## What "done" means

- Cover **every acceptance criterion** + the risky paths: [money, auth, concurrency, migrations]
- Don't test: [trivial getters, generated code, framework glue]
- Coverage: [a target if you have one — otherwise "every AC has a test that fails when the behaviour breaks"]

## For the agent

- Write the failing test first (red); confirm it fails for the right reason; then implement (green).
- Assert observable behaviour (response, persisted row, return value) — not implementation detail.
- A test that still passes when the feature is broken is worthless — flag or rewrite it.
