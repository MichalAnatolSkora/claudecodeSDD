---
description: Once per project — ARCHITECTURE.md plus an ADR per hard-to-reverse decision, read off existing code or decided by Q&A from the PRD
argument-hint: [path to scan, or path to the PRD] (optional)
---

# Architecture — once per project

Lay the project's **structural foundation**: a lean `ARCHITECTURE.md` at the repo root (structure: `templates/ARCHITECTURE.md`) plus an ADR in `docs/adr/` (structure: `templates/ADR.md`) for each hard-to-reverse decision. Run this **once per project**, when it earns its place — not before. Input: `$ARGUMENTS`

Pick the mode and tell me which:

- **From code** — a codebase already exists (retrofitting SDD, or a kept PoC). Scan root: `$ARGUMENTS`, default `src/` or the repo's main source dir.
- **From the PRD** — greenfield: little or no code yet, an accepted PRD under `docs/prd/` (or `PRD.md`).

## From code — describe what's there

1. **Scan** the source: entry points (main, API routes, schedulers, queue consumers), the main modules, the data layer (which store, how it's accessed), and how the system talks to the outside. Follow imports to find the real boundaries — not the folder names.
2. **Map the components** — name each real unit and what it does in one line. Note which layers hold I/O vs pure logic, and where the cross-cutting concerns (logging, validation, auth) live.
3. **Trace the main data flow** — the dominant path from trigger to outcome, including the failure path.
4. **Draft `ARCHITECTURE.md`** — structural and short; describe, don't redesign.
5. **An ADR per load-bearing decision already in force** that you can read off the code (the data-access choice, the scheduler, the service shape, a strong pattern), with `Status: Accepted — <today>` — these are *reactive* ADRs recording what's already true. In `Context`, note the evidence in code.
6. Mark every inference `[VERIFY]`.

**Describe reality, don't refactor it.** If the code does something questionable, record it (flag it `[VERIFY]`) — don't document the version you wish existed; cleanups are separate specs. A choice that should actually change is a new ADR, not this one.

## From the PRD — decide by Q&A

The PRD is deliberately tech-free, so the system-level choices it leaves open get decided **here, by asking me** — don't invent them.

1. **Read the PRD** — especially `Constraints`, `Success metrics`, and `Out of scope`. Constraints often *force* the answer (e.g. "EU data residency" → region / on-prem; a tight p99 → no cold-start serverless). Note what the PRD already settles.
2. **Ask me** only the foundational decisions still open — grouped, a few at a time, then wait: **runtime / hosting**, **datastore**, **shape** (one service vs several; sync API vs scheduled batch vs event-driven), **external integration** (REST, SFTP, queue, webhook), **authN / authZ**. Skip anything the PRD settles or a feature's spec can decide later.
3. **Draft `ARCHITECTURE.md`** — components (one line each), boundaries, the main data flow. No feature detail, no runbook material.
4. **An ADR per hard-to-reverse choice** from step 2, with `Status: Proposed`; in `Context`, cite the PRD constraint that drove it.
5. Mark every guess `[VERIFY]`.

**Lean, not big-design-up-front.** Decide only what the first slice genuinely can't start without; let the rest surface as you build, captured as ADRs when a real choice forces it. When unsure whether something is foundational or a feature's call, leave it to the feature.

## Saving (only after I accept)

Show me the drafts first. Once I approve: save `ARCHITECTURE.md` and the ADRs (numbered from the highest existing `ADR-NNN`), flip any `Proposed` ADR to `Accepted — <today>`, and add one line per ADR to the **Active decisions** list in `CLAUDE.md`.

- Hard decisions become **ADRs**, not prose buried in `ARCHITECTURE.md` — the file shows the resulting *structure*; the ADR holds the *why*.
- One screen of structure beats an exhaustive inventory; per-module depth lives in the code and in feature specs.
- You draft, ask, and record — **I make the calls.**
