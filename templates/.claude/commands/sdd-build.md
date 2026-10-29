---
description: Build a trio — work tasks.md red→green, commit each task, then the break-the-code check
argument-hint: [path to the spec folder or one-file trio] (optional)
---

# Build the trio

Build the feature by working `tasks.md` one task at a time. Target: `$ARGUMENTS`
If no path is given, use the most recently modified spec under `specs/` — a folder **or** a one-file trio (`specs/<slug>.md`) — and tell me which one.

## Before you start

Read `spec.md`, `plan.md`, `tasks.md` in that folder (for a one-file trio: its Spec / Plan / Tasks sections), plus `TESTING.md` and `CLAUDE.md` if present. If the trio clearly hasn't passed `/sdd-check` (an unresolved open question, an acceptance criterion with no task), stop and say so — don't build on a broken trio.

## The loop — one task at a time, in order

1. **Red** — write the failing test first, from the acceptance criterion the task maps to. Run it; confirm it fails for the right reason.
2. **Green** — write the *minimum* code to pass; nothing speculative (per `CLAUDE.md` § Simplicity First, if your `CLAUDE.md` carries the behavioral layer).
3. **Verify** — run the task's ` → verify:` step; it must pass.
4. **Commit** — one commit per green task. A cheap rollback point.

Do one task per cycle. Don't batch tasks or skip ahead.

## If you can't implement a task

Stop — that's a **spec gap, not a cue to improvise.** Name what's missing (an unresolved open question, an underspecified AC, a file not in the plan) and pick one: shrink the task, feed the missing context, or send it back to `/sdd-spec`. Don't vibe-code past it.

If an acceptance criterion turns out **wrong** (not missing): the spec isn't merged yet, so it isn't frozen — fix it in place, add a dated line to the trio's Notes saying why, update the plan and tasks to match, re-run `/sdd-check`, then continue.

## After the last task — break the code (don't trust green)

1. **Confirm each acceptance criterion is actually asserted** — not just that *some* test goes green. Walk the spec's AC list: each one must map to a test whose assertion would fail if that AC were violated. An AC with no asserting test is uncovered even when the suite is green.
2. For each new test, break the implementation on purpose (flip a condition, return a wrong value) and confirm the test goes **red**.
3. Report any test that stayed green — that's false confidence; fix the assertion, not the code.
4. Restore the implementation; confirm the suite is green again.
5. If building this taught you something about the next slice — a new one, a now-unnecessary one, a re-ordering — say so; `/sdd-prd` re-slices the PRD's Feature breakdown before the next trio.

## Constraints

- Touch only files in `plan.md` § File structure. An unexpected new file → stop; it may be a plan gap.
- Never mark a task done without running its verification.
- End with a short summary: tasks done, commits made, any test that survived break-the-code.
