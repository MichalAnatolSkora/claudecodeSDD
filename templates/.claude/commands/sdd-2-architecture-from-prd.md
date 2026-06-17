---
description: Greenfield — Q&A the foundational choices (hosting, datastore, shape) from a PRD into ARCHITECTURE.md plus a stub ADR per hard decision
argument-hint: [path to the PRD] (optional)
---

# Architecture from a PRD

Turn an accepted PRD into the project's **structural foundation**: a lean `ARCHITECTURE.md` plus a stub ADR for each hard-to-reverse decision. The PRD is deliberately tech-free, so the system-level choices it leaves open get decided **here, by Q&A** — don't invent them. PRD: `$ARGUMENTS` (default: the PRD under `docs/prd/`, or `PRD.md`).

Run this **once per project**, when you commit to building — not before it's earned. Existing codebase? Use `/sdd-2-architecture-from-code` instead.

## Steps

1. **Read the PRD** — especially `Constraints`, `Success metrics`, and `Out of scope`. Constraints often *force* the answer (e.g. "EU data residency" → region / on-prem; a tight p99 → no cold-start serverless). Note what the PRD already settles before asking anything.
2. **Ask me** the foundational decisions still open — grouped, a few at a time, then wait. Only the ones you genuinely can't start without:
   - **Runtime / hosting** — cloud (which) vs on-prem vs serverless vs containers; managed vs self-run.
   - **Datastore** — relational / document / key-value / none; which engine; managed?
   - **Shape** — one service vs several; sync API vs scheduled batch vs event-driven.
   - **External integration** — how it talks in and out (REST, SFTP, queue, webhook).
   - **AuthN / AuthZ** — who calls it, and how identity is established.
   Skip anything the PRD or my answers already settle. Don't ask what a feature's spec/plan can decide later.
3. **Draft `ARCHITECTURE.md`** (repo root) from `templates/ARCHITECTURE.md` — keep it **structural and short**: components (one line each), boundaries, the main data flow. No feature detail, no runbook material.
4. **Stub an ADR** in `docs/adr/` for each *hard-to-reverse* choice from step 2 (hosting, datastore, shape…), using `templates/ADR.md`. In `Context`, cite the PRD constraint that drove it; set `Status: Proposed`. Number sequentially from the highest existing `ADR-NNN`.
5. Mark every guess `[VERIFY]`.

## Saving (only after I accept)

Show me the drafts first. Once I approve: save `ARCHITECTURE.md` and the ADR files, flip each ADR to `Accepted — <today>`, and add one line per ADR to the **Active decisions** list in `CLAUDE.md`.

## Constraints

- **Lean, not big-design-up-front.** Decide only the unavoidable foundation; per-feature detail is the trio's job (`/sdd-3-spec-new` →). If a question can wait for a spec, don't ask it here.
- Hard decisions become **ADRs**, not prose buried in `ARCHITECTURE.md`. The file shows the resulting *structure*; the ADR holds the *why*.
- Don't invent the answers — **I make the calls.** You draft, ask, and record.
