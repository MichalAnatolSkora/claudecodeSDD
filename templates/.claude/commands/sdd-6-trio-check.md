---
description: Final consistency audit across spec.md + plan.md + tasks.md (read-only)
argument-hint: [path to the spec folder] (optional)
---

# Trio consistency check

The "ready for implementation" gate. Target: `$ARGUMENTS`
If no path is given, use the most recently modified spec under `specs/` — a folder **or** a one-file trio (`specs/<slug>.md`) — and tell me which one.

## Checks

Read all three of `spec.md`, `plan.md`, `tasks.md` (for a one-file trio: its Spec / Plan / Tasks sections), then verify:

1. Every **acceptance criterion** in `spec.md` has at least one task in `tasks.md` (and an entry in the Verification section).
2. Every file in `plan.md` § File structure has at least one task that creates or modifies it.
3. Every **Out of scope** item in `spec.md` is respected — no task touches it.
4. Every **Open Question** in `spec.md` is `[x]` resolved or moved to an ADR; none still `[ ]`.
5. `plan.md` technical decisions don't contradict any Accepted ADR.
6. `tasks.md` uses only file paths listed in `plan.md` § File structure.

## Output

Return a markdown table: *check | status (pass / fail) | evidence (cite specific lines)*.
**Don't modify any files** — surface gaps only; I'll decide what to fix before implementation.
