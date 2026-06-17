---
description: Draft a new spec.md from a one-paragraph feature idea
argument-hint: [one-paragraph description of the feature]
---

# Draft a spec from an idea

Turn a rough feature idea into a structured `spec.md`. The idea: `$ARGUMENTS`

## Steps

1. Pick a folder: `specs/YYYY-MM-feature-slug/` (use this month's date; derive a short slug from the idea).
2. Draft `spec.md` there using `templates/spec.md` as the structure. Fill in:
   - **Goal** — 1–3 sentences: what problem, for whom, in which system.
   - **In scope** — concrete, observable outcomes.
   - **Out of scope (deliberately, not now)** — at least three things a reader might otherwise assume.
   - **Acceptance criteria** — testable checkboxes; phrase each so it could become a test name.
   - **Impact on existing code** — specific file paths, after scanning `src/`.
   - **Open questions** — do NOT answer them yourself; list what's genuinely undecided.
   - **References** — related ADRs by number, prior specs by slug, integrations if relevant.
3. Mark any guess with `[VERIFY]` — especially file paths you inferred.
4. After the spec is saved, if `specs/FEATURES.md` exists, update the matching feature's row (match by name or its `Spec` link): set `Status` to `spec'd` unless it's already further along, and point the `Spec` column at `specs/<slug>/`. No `FEATURES.md`? Skip this.

## Constraints

- Show me the draft before saving.
- Don't invent answers to Open Questions — surfacing them is the point.
- If the idea is too vague to scope, ask one round of clarifying questions first.
- **Discovery feature (acceptance criteria genuinely unknowable up front)?** Don't invent the AC — spike first (throwaway), let them *emerge* from working code, then come back and fill them in. Until the spike resolves them, leaving AC as Open Questions is fine; note **discovery** at the top of the spec so `/sdd-6-trio-check` waits for the spike instead of failing on open AC. (This is *discovery mode* — see `spec-plan-tasks-guide.md` § "Two modes: delivery and discovery".)
