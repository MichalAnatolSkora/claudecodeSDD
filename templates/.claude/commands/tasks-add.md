---
description: Add task(s) to an existing tasks.md (or the Tasks section of a one-file trio)
argument-hint: [what to add] — e.g. "tasks for the Retry-After header"
---

# Add tasks

Append new, verifiable task(s) to a feature's task list without rewriting it. What to add: `$ARGUMENTS`

## Steps

1. Find the target:
   - a `tasks.md` in the relevant `specs/<slug>/` folder, **or**
   - the `## Tasks` section of a one-file trio (`specs/<slug>.md`).
   If you can't tell which feature, ask — or use the most recently modified one and say which you picked.
2. Read the existing `spec.md` / `plan.md` (or the file's Spec/Plan sections) so the new tasks fit the plan's file structure and the spec's acceptance criteria.
3. Insert each new task **in the right execution order** — not blindly at the end. If it must run before an existing task, place it there and renumber.
4. Every task ends with ` → verify: [how to confirm]`. No verify step = incomplete.
5. If a new task proves an acceptance criterion, add or adjust the matching row in the **Verification** section.

## Constraints

- **Append/insert only — don't rewrite or reorder unrelated tasks.** Show me the diff before saving.
- Use only file paths that appear in `plan.md` § File structure; if a task needs a new path, flag it (the plan may need updating first).
- Mark anything you're guessing with `[VERIFY]`.
