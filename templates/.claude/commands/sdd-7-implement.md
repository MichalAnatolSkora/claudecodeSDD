---
description: Work tasks.md task-by-task, red→green, commit each, then the break-the-code check
argument-hint: [path to the spec folder] (optional)
---

# Implement the trio

Build the feature by working `tasks.md` one task at a time. Target: `$ARGUMENTS`
If no path is given, use the most recently modified spec under `specs/` — a folder **or** a one-file trio (`specs/<slug>.md`) — and tell me which one.

## Before you start

Read `spec.md`, `plan.md`, `tasks.md` in that folder (for a one-file trio: its Spec / Plan / Tasks sections), plus `TESTING.md` and `CLAUDE.md` if present. If the trio clearly hasn't passed `/sdd-6-trio-check` (an unresolved open question, an acceptance criterion with no task), stop and say so — don't build on a broken trio. If `specs/FEATURES.md` exists, flip this feature's row to `in progress` before the first task.

## The loop — one task at a time, in order

1. **Red** — write the failing test first, from the acceptance criterion the task maps to. Run it; confirm it fails for the right reason.
2. **Green** — write the *minimum* code to pass; nothing speculative (per `CLAUDE.md` § Simplicity First, if your `CLAUDE.md` carries the behavioral layer).
3. **Verify** — run the task's ` → verify:` step; it must pass.
4. **Commit** — one commit per green task. A cheap rollback point.

Do one task per cycle. Don't batch tasks or skip ahead.

## If you can't implement a task

Stop — that's a **spec gap, not a cue to improvise.** Name what's missing (an unresolved open question, an underspecified AC, a file not in the plan) and pick one: shrink the task, feed the missing context, or send it back to `/sdd-3-spec-review` / `/sdd-5-tasks-add`. Don't vibe-code past it.

If the gap is a *wrong* acceptance criterion (not a missing one), that's the code→spec loop: while the spec is still **Active** (pre-merge) edit it in place with a dated `CHANGED during implementation: <what and why>` note, update `plan.md`/`tasks.md`, re-run `/sdd-6-trio-check`, then continue. The freeze starts at merge — until then the spec is allowed to change.

## After the last task — break the code (don't trust green)

1. For each new test, break the implementation on purpose (flip a condition, return a wrong value) and confirm the test goes **red**.
2. Report any test that stayed green — that's false confidence; fix the assertion, not the code.
3. Restore the implementation; confirm the suite is green again.
4. If `specs/FEATURES.md` exists, set this feature's row to `shipped` (the spec's own `STATUS: shipped` marker is added later at merge, per `tasks.md` Post-merge).
5. If building this taught you something about the next slice — a new one, a now-unnecessary one, a re-ordering — say so, and re-run `/sdd-2-features-from-prd` to merge it into the backlog before the next trio. Shipping a slice routinely re-ranks what comes next.

## Constraints

- Touch only files in `plan.md` § File structure. An unexpected new file → stop; it may be a plan gap.
- Never mark a task done without running its verification.
- End with a short summary: tasks done, commits made, any test that survived break-the-code.
