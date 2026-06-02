---
name: trio-consistency
description: Check that a feature's spec.md, plan.md, and tasks.md agree before implementation. Use proactively whenever spec/plan/tasks files are edited together, or before starting to implement a spec — even if the user didn't explicitly ask for a consistency check.
---

# Trio consistency

Audit a feature trio for internal agreement. Read-only — surface gaps; never modify the files.

## When to run

- Before implementation of any spec begins.
- Whenever you notice `spec.md`, `plan.md`, or `tasks.md` were edited in the same change.

## Checks

For the target `specs/<slug>/` folder, read all three docs and verify:

1. Every acceptance criterion in `spec.md` maps to at least one task (and a Verification entry).
2. Every file in `plan.md` § File structure has a task that creates or modifies it.
3. Every "Out of scope" item in `spec.md` is respected — no task touches it.
4. Every Open Question in `spec.md` is resolved (`[x]`) or moved to an ADR.
5. No `plan.md` technical decision contradicts an Accepted ADR in `docs/adr/`.
6. `tasks.md` uses only file paths from `plan.md` § File structure.
7. The three docs use consistent vocabulary for the same concept.

## Output

A markdown table: *check | pass/fail | evidence (cite specific lines)*. End with a one-line verdict: ready for implementation, or the specific gaps to fix first.
