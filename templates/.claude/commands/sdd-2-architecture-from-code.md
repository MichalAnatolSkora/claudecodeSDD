---
description: Reverse-engineer ARCHITECTURE.md from existing code (components, boundaries, data flow) plus an ADR per load-bearing decision already in force
argument-hint: [path to scan] (optional, default: src/)
---

# Architecture from existing code

Build `ARCHITECTURE.md` for a codebase that already exists — the foundation step when retrofitting SDD (see `guides/legacy-to-sdd-migration-guide.md`). You describe **what's actually there**, not what should be. Scan root: `$ARGUMENTS` (default: `src/`, or the repo's main source dir — tell me which you used).

## Steps

1. **Scan** the source: entry points (main, API routes, schedulers, queue consumers), the main modules/packages, the data layer (which store, how it's accessed), and how the system talks to the outside. Follow imports to find the real boundaries — not the folder names.
2. **Map the components** — name each real unit and what it does in one line. Identify which layers hold I/O vs pure logic, and the cross-cutting concerns (logging, validation, auth) and where they live.
3. **Trace the main data flow** — the dominant path through the system, from trigger to outcome, including the failure path.
4. **Draft `ARCHITECTURE.md`** (repo root) from `templates/ARCHITECTURE.md`. Keep it structural and short; describe, don't redesign.
5. **Stub an ADR** in `docs/adr/` for each *load-bearing decision already in force* that you can read off the code (the data-access/ORM choice, the scheduler, the service shape, a strong pattern). Use `templates/ADR.md` with `Status: Accepted — <today>` — these are *reactive* ADRs recording what's already true — and in `Context` note the evidence in code. Number sequentially from the highest existing `ADR-NNN`.
6. Mark every inference `[VERIFY]` — anything you read off the code but couldn't confirm is intentional.

## Saving (only after I accept)

Show me the drafts first. Once I approve: save `ARCHITECTURE.md` and the ADR files, and add one line per ADR to the **Active decisions** list in `CLAUDE.md`.

## Constraints

- **Describe reality, don't refactor it.** If the code does something questionable, record it (flag it `[VERIFY]`) — don't document the version you wish existed. Cleanups are separate specs.
- Don't drown it in detail — one screen of structure beats an exhaustive inventory. Per-module depth lives in the code and in feature specs.
- A reverse-engineered ADR captures *what is*, not a fresh decision. If a choice should actually change, that's a new ADR — not this one.
