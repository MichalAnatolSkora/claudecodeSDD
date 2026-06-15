---
description: Slice an accepted PRD into a prioritized feature list, saved to specs/FEATURES.md once I accept
argument-hint: [path to the PRD] (optional)
---

# Slice the PRD into features

Turn an accepted PRD into a prioritized list of features — each a vertical slice sized to its own spec → plan → tasks trio — and record it in `specs/FEATURES.md` as the project's feature index. PRD: `$ARGUMENTS` (default: the PRD under `docs/prd/`, or `PRD.md`).

## Steps

1. Read the PRD — especially its success criteria / user stories and `Out of scope`.
2. Propose features as **vertical slices** — each independently shippable and user-visible. NOT layers: "DB layer" / "API layer" are not features.
3. Size each to ~a few days (a `spec.md` under ~150 lines). Split anything bigger; fold anything trivial into a neighbour.
4. Flag the **walking skeleton** — the thinnest end-to-end slice that proves the PRD's core — then order the rest by value × dependency and mark dependencies.
5. No feature may violate the PRD's `Out of scope` — flag any proposed slice that would.
6. Lay the result out as the `specs/FEATURES.md` table (structure: `templates/FEATURES.md`) and **show it to me for review.**

## Saving (only after I accept)

Don't write the file before I approve the list. Once I do, save it to `specs/FEATURES.md`:

- **If `specs/FEATURES.md` already exists, merge — never reset progress.** Keep the `Status` of features already listed, append new slices as `planned`, and flag any listed feature missing from this breakdown (renamed or dropped?) rather than deleting it.
- Set `Status` from what's on disk, and **never downgrade**: `shipped` if `specs/<slug>/spec.md` carries `STATUS: shipped`, else `spec'd` if that `spec.md` exists, else `planned`. Leave an `in progress` row untouched.
- Link the `Spec` column to `specs/<slug>/` for features whose folder exists; `—` otherwise.

## Constraints

- **Don't write specs.** This is the breakdown + index only — I'll pick which features become specs, then run `/sdd-3-spec-new <feature>` per one.
- Mark anything you're guessing with `[VERIFY]`.
