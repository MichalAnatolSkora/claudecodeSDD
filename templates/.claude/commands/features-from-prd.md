---
description: Draft a vertically-sliced, prioritized feature list from an accepted PRD
argument-hint: [path to the PRD] (optional)
---

# Slice the PRD into features

Turn an accepted PRD into a prioritized list of features — each a vertical slice sized to its own spec → plan → tasks trio. PRD: `$ARGUMENTS` (default: the PRD under `docs/prd/`, or `PRD.md`).

## Steps

1. Read the PRD — especially its success criteria / user stories and `Out of scope`.
2. Propose features as **vertical slices** — each independently shippable and user-visible. NOT layers: "DB layer" / "API layer" are not features.
3. Size each to ~a few days (a `spec.md` under ~150 lines). Split anything bigger; fold anything trivial into a neighbour.
4. Flag the **walking skeleton** — the thinnest end-to-end slice that proves the PRD's core — then order the rest by value × dependency and mark dependencies.
5. Keep every feature within the PRD's `Out of scope`.

## Output

A prioritized table: *feature | the PRD outcome it serves | rough size | depends on | P1/P2/P3*. Each row is a candidate `specs/YYYY-MM-slug/`.

## Constraints

- **Don't write specs.** This is the breakdown only — I'll pick which features become specs, then run `/spec-new <feature>` per one.
- Mark anything you're guessing with `[VERIFY]`.
