# Testing in SDD: Tests the Agent Writes (and You Verify)

> How testing fits into spec-driven development — and how to make it work when an AI agent writes most of the tests. The short version: the spec's **acceptance criteria are the test contract**, the agent writes the tests as it implements, and your job shifts from *writing* tests to *verifying the agent's tests actually verify something*. Built for teams of **1–10**.

---

## Table of Contents

1. [What this guide is](#what-this-guide-is)
2. [Where testing lives in SDD](#where-testing-lives-in-sdd)
3. [TESTING.md — the conventions the agent reads](#testingmd--the-conventions-the-agent-reads)
4. [From acceptance criteria to tests](#from-acceptance-criteria-to-tests)
5. [TDD with an agent (red → green)](#tdd-with-an-agent-red--green)
6. [Getting good tests out of the agent](#getting-good-tests-out-of-the-agent)
7. [Don't trust green — verify the agent's tests](#dont-trust-green--verify-the-agents-tests)
8. [What to test (pragmatic coverage)](#what-to-test-pragmatic-coverage)
9. [The agent's run-fix loop](#the-agents-run-fix-loop)
10. [AI-assisted testing prompts](#ai-assisted-testing-prompts)
11. [Anti-patterns](#anti-patterns)
12. [Golden rules](#golden-rules)

---

## What this guide is

In SDD you don't bolt testing on at the end. Every spec already carries **acceptance criteria** — the testable definition of done — and every `TASKS.md` already ends each step with `→ verify:`. Testing is just the discipline of turning those into real, running tests, and (increasingly) letting the agent write them.

This guide covers the part the other guides only gesture at:

- where tests live in the SDD loop,
- the `TESTING.md` file that makes the agent write tests in *your* style,
- how to get **good** tests out of an agent (its tests are the output most likely to be confidently wrong),
- and how to not trust a green suite that proves nothing.

It assumes the agent does most of the typing. Your leverage is in the acceptance criteria you write and the tests you sanity-check — not in hand-writing every assertion.

---

## Where testing lives in SDD

Three places, already in the method:

- **`SPEC.md` § Acceptance criteria = the test contract.** Each AC is phrased so it *could be a test name* (*"POST /api/orders with no `X-Partner-Id` → 400"*). That's not a coincidence — the ACs are the tests, written in English first.
- **`TASKS.md` = where tests get written.** The implementation steps are ordered red→green (failing test first), and the **Verification (against acceptance criteria)** section maps every AC to the task/test that proves it. `/sdd-6-trio-check` runs *before* implementation, so what it checks is that every AC has a **task** (and a Verification row) — the tests themselves arrive later, in the red→green loop of `/sdd-7-implement`.
- **`TESTING.md` = the conventions the agent reads.** So the tests it writes match your framework, layout, and mocking rules instead of its own defaults.

> Command names here are the shipped full names; the `sdd-N-` prefix just makes the files sort in pipeline order.

So "adding testing to SDD" is mostly: write testable ACs, keep the red→green order, and give the agent a `TESTING.md`. The rest of this guide is doing that well.

---

## TESTING.md — the conventions the agent reads

`TESTING.md` is to tests what `CLAUDE.md` is to code: the short, durable file that stops the agent from inventing its own conventions. Without it, the agent picks a framework, a folder, a mocking style, and a naming scheme on every session — and they drift.

Keep it short. It should answer the questions the agent would otherwise guess:

- **Framework + runner** — what you use (xUnit / Jest / pytest / …) and how tests run (`dotnet test`, `npm test`).
- **Where tests live** — folder and naming (`tests/`, `*.Tests`, `*_test.py`), and how unit vs integration are separated.
- **Unit vs integration boundary** — what's a unit test here, what needs the real thing (DB, HTTP). *"Repository tests run against Testcontainers Postgres, not mocks."*
- **What to mock (and never mock)** — the rule that prevents over-mocking. *"Mock outbound HTTP and the clock; never mock the type under test or the database."*
- **Test data / fixtures** — how to build test data (builders, factories, fixtures) and where shared ones live.
- **What "done" means** — coverage expectations (a target, or "cover every AC + the gnarly paths") and what doesn't need a test (trivial getters, generated code).

A copy-pasteable starting point lives at [`templates/TESTING.md`](../templates/TESTING.md). Trigger to write it: the **third time** you correct the agent on a test convention (per the main guide's reactive-files rule).

---

## From acceptance criteria to tests

The cleanest agent task in all of SDD is "turn these acceptance criteria into tests" — because the ACs are already concrete.

- **One AC → at least one test.** If an AC needs three cases (happy, missing field, over-limit), that's three tests, all tracing to the same AC.
- **Phrase ACs as test names.** *"Same partner over N requests in window W → 429 with `Retry-After`"* becomes a test name almost verbatim. Vague ACs (*"rate limiting works"*) produce vague or fake tests — sharpen the AC first.
- **The Verification section is the coverage gate.** `TASKS.md` maps every AC to its test; "ready to merge" means every row is green. You don't chase a coverage *percentage* — you chase *every AC has a test that would fail if the behaviour broke*.

If an AC is hard to write a test for, that's a signal the AC is vague or the design is untestable — fix it in the spec/plan, not by writing a weaker test.

---

## TDD with an agent (red → green)

`TASKS.md` already opens features with a failing test. With an agent, the loop is:

1. **Red** — write the failing test for the next AC *first*. Run it; confirm it fails **for the right reason** (the behaviour is missing — not a typo, not a missing import).
2. **Green** — implement the minimum to pass it. Run; confirm green.
3. **Next** — move to the next AC. Don't write three features then backfill tests.

Why red-first matters with an agent specifically: a test written *after* the code tends to assert whatever the code already does — **including its bugs**. A test written *before* asserts what the AC says *should* happen. The order is a cheap guard against the agent rubber-stamping its own implementation.

**Prompt:**

```text
Implement only AC2 from spec.md. First write the failing test (name it after the AC),
run it, and show me it fails for the right reason. Then write the minimum code to
pass it, and run the suite. Don't touch other ACs.
```

**When NOT to test-first.** Red-first is the default for AC-driven work, not a universal law:

- **Spikes / prototypes** — you're going to throw the code away; tests on throwaway code are waste. Write what you *learned* into the spec instead.
- **Exploratory UI work** — you don't know what right looks like until you see it. Pin the behaviour with tests once it settles.
- **Legacy code without seams** — you can't red-first what you can't instantiate. Start with characterization tests (assert what it does *today*), then refactor toward testability.

---

## Getting good tests out of the agent

The agent's tests are the output **most likely to be confidently wrong** — plausible, green, and proving nothing. Know the failure modes and prompt against them:

- **Tautological / always-green** — asserting on a value the test computed the same way the code does, or `assert x == x`. *Guard:* the test must fail if you break the behaviour (next section).
- **Testing the implementation, not the behaviour** — asserting on private calls, internal state, or exact log strings. *Guard:* "assert on observable behaviour — the response, the persisted row, the return value — not on how it's computed."
- **Over-mocking** — mocking the very thing under test, or mocking so much the test only verifies the mocks. *Guard:* the `TESTING.md` mocking rule — mock the edges (HTTP, clock), never the unit under test or the DB.
- **Happy-path only** — one passing case, no edge or error cases. *Guard:* "for each AC, include the failure and boundary cases, not just the success."
- **Asserting incidental detail** — pinning exact timestamps, ordering, whitespace, full-object equality. *Guard:* assert the part the AC cares about; brittle tests get deleted, not maintained.

**Prompt:**

```text
Review these tests against the spec's acceptance criteria. For each test:
- does it assert observable behaviour, or implementation detail?
- would it still pass if the feature were broken? (if yes it's worthless — flag it)
- does it over-mock — mock the type under test or the database?
- are the error/boundary cases covered, not just the happy path?
List the weak tests and how to fix them. Don't rewrite yet.
```

---

## Don't trust green — verify the agent's tests

A passing suite proves nothing if the tests don't actually exercise the behaviour. The cheapest possible check:

> **Break the code on purpose. The test must go red.**

Comment out the rate-limit check, or return the wrong status, and run the test that's supposed to cover it. If it stays green, the test is theatre — it isn't testing what it claims. This is mutation testing by hand, and at 1–10 it's enough: you don't need a mutation framework, you need ten seconds of *"did this test even fire?"*

Treat the agent's tests like any other agent output: **review them.** The trio guide's rule applies — the agent's plausible-sounding fills are most likely wrong exactly where they matter (the assertions). A green checkmark from the agent is a claim, not a proof.

---

## What to test (pragmatic coverage)

Coverage targets are a trap for small teams. Test by *value at risk*, not by percentage:

- **Always:** every acceptance criterion; the gnarly/risky bits (money, auth, concurrency, data migrations — anything you'd be scared to break).
- **Usually:** integration tests over the real seams (DB, HTTP) where the bugs actually live — for a 1–10 team these catch more than a wall of mocked unit tests.
- **Skip:** trivial getters/setters, framework glue, generated code, and anything a type-checker already guarantees.
- **Don't** chase 100% — the last 20% is mostly testing trivia, and a green 100% of weak tests is worse than 70% of real ones (it lies to you).

The 1–10 default: **a thin layer of integration tests on the ACs and the scary paths, plus unit tests where the logic is dense.** Grow it when a class of bug keeps escaping.

**Integration tests need infrastructure.** Standing up the real thing (e.g. Testcontainers) means Docker has to be available *everywhere the suite runs* — your machine, CI, and the agent's environment; a suite the agent can't run is a loop it can't close. Keep the integration layer thin on purpose: real dependencies are slow, and a fat integration suite turns the run-fix loop into a wait. And put the exact run command (plus any setup, like starting Docker) in `TESTING.md`, so the agent can run the suite unaided.

---

## The agent's run-fix loop

The agent can close the loop itself if you give it a runnable target — this is "Goal-Driven Execution" applied to tests:

1. Write the failing test (red).
2. Implement.
3. **Run the suite.** Read the failure output.
4. Fix. Re-run. Repeat until green.
5. Then stop — don't keep "improving."

**Prompt:**

```text
Work tasks.md from where we are. For each task: write/keep its test, run the suite,
and if it's red, read the failure and fix it — loop until green before the next task.
Show me the diff and the test output at each task boundary, not a wall at the end.
```

Let the agent run the tests (it reads a failure better than it predicts one). Your gate is at task boundaries: review the diff + the test, and run the break-it check on anything important.

---

## AI-assisted testing prompts

Five reusable prompts (the agent types; you judge):

1. **Write tests from ACs** — *"Write tests for every acceptance criterion in `SPEC.md`, following `TESTING.md`. One test per case (happy + each failure/boundary). Name each after the AC. Don't change production code."*
2. **Add the missing edge cases** — *"List the edge and error cases the current tests don't cover (empty, null, boundary, concurrent, malformed). Add a test for each."*
3. **Critique a test for weakness** — the *"would it pass if broken?"* prompt above.
4. **Generate test data / fixtures** — *"Build a test-data builder for `Order` with sensible defaults and overridable fields, per `TESTING.md`'s fixture convention."*
5. **Triage a flaky test** — *"This test is flaky. Find the non-determinism (clock, ordering, shared state, real I/O) and make it deterministic — inject the clock, don't `sleep`."*

---

## Anti-patterns

1. **Tests written after the fact to hit a coverage number.** They assert what the code does, bugs included. Write the test from the AC, ideally first.
2. **The green-but-hollow suite.** Hundreds of passing tests, none of which fail when you break the code. Run the break-it check on anything that matters.
3. **Testing the mock.** So much is mocked the test only proves the mocks were called. Mock the edges, not the unit.
4. **Snapshot-everything.** Giant auto-generated snapshots nobody reads; they pass until they don't, and then everyone just re-blesses them. Use sparingly, for stable output only.
5. **Chasing 100%.** Effort goes to trivia while the scary paths get one happy-path test. Test by risk.
6. **A Verification section that doesn't map to tests.** If *"AC4 → covered"* points at no real test, it's a checkbox, not coverage. `/sdd-6-trio-check` flags vague rows before implementation; the break-the-code check is what proves the test is real.

---

## Golden rules

1. **The acceptance criteria are the tests.** Write them testable; the tests are their translation.
2. **Red before green.** A test written after the code asserts the code's bugs.
3. **A test that can't fail isn't a test.** Break the code; the test must go red.
4. **Review the agent's tests — especially the assertions.** That's where its plausible-wrong output hides.
5. **Mock the edges, never the unit under test or the DB.** Put the rule in `TESTING.md`.
6. **Test by risk, not by percentage.** Every AC + the scary paths; skip the trivia.
7. **`TESTING.md` is the durable home for "how we test here."** Write it the third time you correct the agent.

---

*This guide is the testing companion to [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md) (the `TASKS.md` verify steps and the Verification gate), [`working-with-agents-guide.md`](working-with-agents-guide.md) (the implementation loop and prompting), and [`quality-gates-guide.md`](quality-gates-guide.md) (running the suite as a CI / hook gate). The trio defines what "done" means; this guide is how you prove it — without trusting the agent's green on faith.*
