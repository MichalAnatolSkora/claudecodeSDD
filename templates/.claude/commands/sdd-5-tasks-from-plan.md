---
description: Draft tasks.md (ordered, verifiable) from spec.md + plan.md
argument-hint: [path to the spec folder] (optional)
---

# Draft tasks from a plan

Turn a plan into an ordered, verifiable task list. Spec folder: `$ARGUMENTS`
If no path is given, use the most recently modified folder under `specs/` and tell me which one.

## Steps

1. Read `spec.md` and `plan.md` in that folder.
2. Draft `tasks.md` using `templates/tasks.md` as the structure.
3. For each acceptance criterion in `spec.md`, identify which task(s) prove it — the **Verification** section must map every AC.
4. Order tasks red→green: an early task lets you write a failing test, later tasks make it pass. Group under headings if there are more than ~8.
5. Every task ends with ` → verify: [how to confirm]`. A task without a verification step is incomplete.

## Constraints

- Show me the draft. Mark anything you're guessing with `[VERIFY]`.
- Use only file paths that appear in `plan.md` § File structure.
