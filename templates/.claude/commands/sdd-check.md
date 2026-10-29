---
description: Read-only audit of a PRD or a trio — whatever exists so far — and the gate before /sdd-build
argument-hint: [path to a PRD, a spec folder, or a one-file trio] (optional)
---

# Check

Audit without modifying anything. Target: `$ARGUMENTS`
If no path is given, use the most recently modified spec under `specs/` — a folder **or** a one-file trio (`specs/<slug>.md`) — or the PRD under `docs/prd/` (or `PRD.md`) if that's newer, and tell me which you picked.

Run the checks for **what exists**: a PRD gets the PRD checks; a spec alone gets the spec checks; a spec + plan adds the plan checks; a full trio gets all three plus the trio checks. For a one-file trio, read its Spec / Plan / Tasks sections.

## PRD

1. Are the **users** specific (job title + company size or equivalent) — or generic ("users")?
2. Does the **problem** describe a specific behavior or cost today, not just "users want X"?
3. Are **success metrics** measurable (numbers and timeframe) — or vague ("users adopt it")?
4. Is **Out of scope** at least 5 items long, with some a reader might assume are in scope?
5. Does it stay free of **implementation detail** (languages, frameworks, databases, class names)? That belongs in specs.
6. Are **risks** named with consequences ("if X, then Y") — or generic?
7. Any **contradictions between sections** (an out-of-scope item reappearing as a constraint or feature)?
8. Is it still **a page or two**, factual and skimmable — claims, numbers, decisions, not marketing prose or filler?
9. If it has a **Feature breakdown**: vertical slices (not layers), walking skeleton first, nothing that violates Out of scope?

## Spec

1. Is the **Goal** phrased as a problem, not a feature?
2. Does **Out of scope** exclude at least three things a reader might assume?
3. Are the **acceptance criteria** specific enough to write tests from?
4. Does **Impact on existing code** list real file paths (verify against `src/`)?
5. Are the **Open questions** genuine uncertainties — or decisions the author should already have made?
6. Are the **references** current (ADR numbers exist in `docs/adr/`)?
7. Does anything **in scope contradict an Accepted ADR**?

## Plan

1. For each **technical decision**: does an Accepted ADR speak to it — and does the plan match or contradict it? Quote the ADR text.
2. Does the plan cross a boundary `ARCHITECTURE.md` establishes?
3. Does every **File structure** path fit the existing layout in `ARCHITECTURE.md`?

## Trio — the gate before code

1. Every **acceptance criterion** has at least one task, and a row in the Verification section.
2. Every file in the plan's File structure has a task that creates or modifies it.
3. Every **Out of scope** item is respected — no task touches it.
4. Every **Open question** is `[x]` resolved or moved to an ADR — none still `[ ]`.
5. No task uses a file path outside the plan's File structure.

## Output

One markdown table per section you ran: *check | pass / fail | evidence (cite specific lines)*. End with one line — **ready for `/sdd-build`**, or what blocks it.
**Don't modify any files** — surface gaps only; I decide what to fix.
