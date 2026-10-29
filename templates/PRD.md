# PRD: [Project / Product name]

> Starting artifact. Captures *what* and *why* before any code exists.
> **PRD is for humans — the agent never implements from it.** It reads this only to draft, review, or slice it into features; code comes from `SPEC.md`.
> Freeze after this era's initial release ships (all but the Feature breakdown); archive to `docs/prd/YYYY-MM-<era-slug>.md`.
> New direction → new PRD (per the era pattern), never edits to this one.
>
> Lean PRD format below. For the one-pager and the PR-FAQ trick for big bets,
> two complete worked examples, AI-assisted authoring, and the review process,
> see [`guides/prd-guide.md`](../guides/prd-guide.md).

## Problem

[What problem does this solve? Why now? 2-5 sentences.]

## Users

[Who is this for? Be specific — primary user, secondary user, who is NOT a user.]

## Success criteria

[Measurable, time-bound. "Users adopt it" is not a criterion. "60% of active accounts use feature X within 3 months" is.]

- ...
- ...

## In scope (v1)

- ...
- ...

## Out of scope (deliberately, not now)

- [What we are NOT building, even if asked]
- ...

## Constraints

- **Technical:** [stack, integrations we must use, systems we cannot touch]
- **Legal / regulatory:** [GDPR, RODO, contractual obligations]
- **Time:** [deadline and why]
- **Team / budget:** [headcount, skills available]

## Feature breakdown

> Vertical slices, walking skeleton first — each becomes one `specs/YYYY-MM-slug/` trio when picked up (`/sdd-prd` drafts this).
> The era's backlog: the one section that stays editable after the PRD freezes — re-order, add, drop as shipped slices teach you.
> No status column: what's shipped is what's merged under `specs/`.

| # | Feature | Serves (PRD outcome) | Size | Depends on |
|---|---------|----------------------|------|------------|
| 1 | [walking skeleton — thinnest end-to-end slice] | [success criterion it proves] | [~days] | — |
| 2 | [feature] | [success criterion] | [~days] | #1 |

## Risks and unknowns

- [What could make this fail? What do we not yet know?]
- ...

## References

- [Linked research, market analysis, prior art, related ADRs]
