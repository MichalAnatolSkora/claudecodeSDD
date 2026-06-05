---
description: Work tasks.md task-by-task, red→green, commit each, then the break-the-code check
argument-hint: [path to the spec folder] (optional)
---

# Implement the trio

Build the feature by working `tasks.md` one task at a time. Spec folder: `$ARGUMENTS`
If no path is given, use the most recently modified folder under `specs/` and tell me which one.

## Before you start

Read `spec.md`, `plan.md`, `tasks.md` in that folder, plus `TESTING.md` and `CLAUDE.md` if present. If the trio clearly hasn't passed `/trio-check` (an unresolved open question, an acceptance criterion with no task), stop and say so — don't build on a broken trio.

## The loop — one task at a time, in order

1. **Red** — write the failing test first, from the acceptance criterion the task maps to. Run it; confirm it fails for the right reason.
2. **Green** — write the *minimum* code to pass (`CLAUDE.md` § Simplicity First — nothing speculative).
3. **Verify** — run the task's ` → verify:` step; it must pass.
4. **Commit** — one commit per green task. A cheap rollback point.

Do one task per cycle. Don't batch tasks or skip ahead.

## If you can't implement a task

Stop — that's a **spec gap, not a cue to improvise.** Name what's missing (an unresolved open question, an underspecified AC, a file not in the plan) and pick one: shrink the task, feed the missing context, or send it back to `/spec-review` / `/tasks-add`. Don't vibe-code past it.

## After the last task — break the code (don't trust green)

1. For each new test, break the implementation on purpose (flip a condition, return a wrong value) and confirm the test goes **red**.
2. Report any test that stayed green — that's false confidence; fix the assertion, not the code.
3. Restore the implementation; confirm the suite is green again.

## Constraints

- Touch only files in `plan.md` § File structure. An unexpected new file → stop; it may be a plan gap.
- Never mark a task done without running its verification.
- End with a short summary: tasks done, commits made, any test that survived break-the-code.
