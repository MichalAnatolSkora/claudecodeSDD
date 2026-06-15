# Features

> The project's feature index: the prioritized backlog of vertical slices from the PRD(s), and where each one stands.
> Generated at `specs/FEATURES.md` by `/sdd-2-features-from-prd`; re-running it **merges** (never resets) statuses.
> Optional — the lightest setup can skip this file and let `specs/*/` + `git` be the map. Keep it only if a single status view earns its keep.
> Source of truth for *what ships* is still the code + git; this is the map, not the territory.

**Status:** `planned` (sliced, no spec yet) · `spec'd` (`specs/<slug>/spec.md` exists) · `in progress` (being implemented) · `shipped` (implemented & verified; spec stamped `STATUS: shipped` at merge)

| # | Feature | Serves (PRD outcome) | Size | Depends on | Priority | Status | Spec |
|---|---------|----------------------|------|------------|----------|--------|------|
| 1 | [walking skeleton — thinnest end-to-end slice] | [PRD outcome it proves] | [~days] | — | P1 | planned | [`specs/<slug>/`] |
| 2 | [feature] | [PRD outcome] | [~days] | #1 | P1 | planned | — |
| 3 | ... | ... | ... | ... | ... | ... | ... |

> Row 1 is the walking skeleton. Keep rows in execution order (value × dependency). `/sdd-3-spec-new` links the **Spec** column and marks a row `spec'd`; `/sdd-7-implement` advances it to `in progress`, then `shipped`. Re-running `/sdd-2` re-derives status from disk but never downgrades — so the index stays honest whether you run the commands or edit it by hand.
