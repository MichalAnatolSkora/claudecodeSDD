---
name: trio-author
description: Author a full feature trio (spec → plan → tasks) for a non-trivial change, with a human review gate between each artifact. Use when starting a new feature that warrants the full trio, or when the user asks to "write the spec/plan/tasks" for a change.
---

# Trio author

Drive the standard feature flow: draft each artifact in order, stopping for human review between steps. Order is the discipline — never skip ahead.

## Procedure

1. **Spec.** Draft `specs/YYYY-MM-feature-slug/spec.md` from the user's idea (see the `/spec-new` command for the section-by-section prompt). List Open Questions; do not answer them yourself.
   - **Stop. Ask the user to review** the spec and resolve Open Questions before continuing. Do not draft the plan until the spec is reviewed.
2. **Plan.** Once the spec is reviewed, draft `plan.md` in the same folder (see `/plan-from-spec`). Cite active ADRs; carry "Out of scope" into Constraints.
   - **Stop for review.** Optionally run the `/plan-validate` checks against `docs/adr/` and `ARCHITECTURE.md`.
3. **Tasks.** Once the plan is reviewed, draft `tasks.md` (see `/tasks-from-plan`). Map every acceptance criterion to a task; order red→green; every task gets a ` → verify:` step.
4. **Gate.** Run a final trio consistency check (see `/trio-check` or the `trio-consistency` skill) before any implementation begins.

## Rules

- One artifact at a time, each human-gated. The agent does the typing; the human judges.
- Mark guesses with `[VERIFY]`.
- Compress for small changes: a bugfix may need only a short spec; a small feature may fit all three sections in one file. Don't pad three files to look thorough.
