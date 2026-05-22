# Spec-Driven Development with AI Coding Agents

> A practical guide to structuring your repository so AI coding agents (Claude Code, Copilot, Cursor) produce consistent, maintainable code instead of drifting away from your intent.

---

## Table of Contents

1. [Core Philosophy](#core-philosophy)
2. [The PRD-First Approach](#the-prd-first-approach)
3. [Writing a Good PLAN.md](#writing-a-good-planmd)
4. [The Three-Layer Documentation Model](#the-three-layer-documentation-model)
5. [Workflow for Changes](#workflow-for-changes)
6. [Architecture Decision Records (ADR) in Depth](#architecture-decision-records-adr-in-depth)
7. [Additional Files Worth Adding](#additional-files-worth-adding)
8. [Repository Organization](#repository-organization)
9. [Working with the Agent: Practical Commands](#working-with-the-agent-practical-commands)
10. [Golden Rules](#golden-rules)

---

## Core Philosophy

**Spec-Driven Development (SDD)** means writing requirements, decisions, and constraints down *before* asking an AI agent to generate code. Every minute spent on specification saves an hour of refactoring agent-generated drift.

The core insight: **AI agents fill gaps with assumptions.** The more they assume, the further the code drifts from what you actually want. Good specs eliminate the gaps.

Related terms you'll encounter:
- **Context engineering** — preparing complete context before generation
- **PRD-first approach** — starting from a Product Requirements Document
- **Spec-kit workflow** — GitHub's formalization: `spec.md` → `plan.md` → `tasks.md`

These are facets of the same idea. The name matters less than the discipline.

---

## The PRD-First Approach

A **PRD (Product Requirements Document)** is your starting artifact. It captures *what* you're building and *why*, before any code exists.

A good PRD answers:
- What problem does this solve?
- Who is the user?
- What does success look like (measurable)?
- What is explicitly out of scope?
- What are the constraints (technical, legal, time)?

**Critical insight:** The PRD is a *starting artifact*, not a living document. Once the project ships its first version, the PRD freezes as history. Subsequent changes live in feature-level specs, not in PRD edits.

Treating PRD as living causes two problems:
- It becomes a dumping ground for every change request
- The original intent gets diluted over time

Keep the original PRD pristine. Archive it in `docs/prd/` with a date. New direction → new PRD or roadmap entry, not edits to the old one.

---

## Writing a Good PLAN.md

A good `PLAN.md` answers questions the agent would otherwise guess at. Every guess is a potential drift from intent.

### Principles That Work

- **Concrete decisions, not options.** Don't write "consider MediatR or services" — pick one. A plan is not a brainstorm.
- **Separate what (requirements) from how (architecture).** Mixing them produces code that mixes layers.
- **Explicit exclusions.** An "out of scope" section prevents the agent from adding features you don't want now.
- **Testable acceptance criteria.** "Works correctly" → bad. "POST /api/x with payload Y returns 201 and a row in T_LOG" → good.
- **Open questions in a separate section.** Forces decisions before the agent starts coding (and before it fabricates an answer).
- **File structure upfront.** Otherwise the agent invents its own and you refactor.

### Template

````markdown
# [Feature/Project Name]

## Goal
[1-3 sentences: what problem, for whom, in which system]

## Scope
### In scope
- ...
### Out of scope (deliberately, not now)
- ...

## Technical Decisions
- Stack: .NET 8, ASP.NET Core, Dapper, MS SQL Server
- Data access pattern: repository + Dapper, no EF
- DI: ...
- Logging: Serilog, contract matching `IBaseHandler<TSelf>`
- Naming conventions: `SCH_X.T_Y`, `PK_T_Y`, ...

## Data Model
### Tables (DDL diff)
```sql
-- new table / change
```
### DTOs / contracts
- `XRequest`: fields...
- `XResponse`: fields...

## File Structure
```
src/
  Domain/
    ...
  Infrastructure/
    Repositories/XRepository.cs
  Application/
    Handlers/XHandler.cs
  Api/
    Controllers/XController.cs
tests/
  Integration/
    XHandlerTests.cs   // Testcontainers + NUnit
```

## Tasks (in execution order)
1. DDL migration script for `SCH_X.T_Y`
2. `XRepository` + unit tests for queries
3. `XHandler` (pure logic, no I/O in tests)
4. `POST /api/x` endpoint + FluentValidation
5. Integration tests with Testcontainers
6. Quartz job registration (if applicable)

## Acceptance Criteria
- [ ] `POST /api/x` with valid payload → 201 + row in `T_Y`
- [ ] Missing required field → 400 with specific message
- [ ] Integration test covers happy path + 2 error cases
- [ ] Each test runs against an isolated database

## Constraints
- Do NOT use EF Core
- Do NOT add CQRS/MediatR — handlers called directly
- Maintain consistency with `OneXmlOneFileStrategyMerge` pattern

## Open Questions
- [ ] Does `T_Y` use soft-delete or hard?
- [ ] Retry policy for SFTP — Polly or custom?

## References
- `src/.../FileImportTablesRepository.cs` — repo pattern to follow
- `docs/adr/ADR-007-dapper.md`
````

### Practical Tips

Keep `PLAN.md` short — if it exceeds ~200 lines, split into `PLAN.md` (what + acceptance) and `ARCHITECTURE.md` (how). AI agents work better with several focused files than one monolith.

After each iteration, move items from "Open Questions" to "Technical Decisions." The plan is alive, but decision history must stay visible — so the agent (and you, in a month) know *why* something is the way it is.

---

## The Three-Layer Documentation Model

Once you ship v1, project life unfolds across three documentation layers that evolve at different speeds.

### Layer 1: Stable (rarely changes, agent always reads)

- `CLAUDE.md` — conventions, stack, what NOT to do
- `ARCHITECTURE.md` — high-level structure, layers, boundaries
- `DOMAIN.md` — domain model, terminology glossary

These define the "physics" of your codebase. Change them only when fundamentals shift.

### Layer 2: Per-feature (created, implemented, frozen)

- `specs/2026-01-campaign-retry/spec.md` + `plan.md` + `tasks.md`
- After shipping, append `STATUS: shipped (PR #123, 2026-01-15)`
- Never edit retroactively — this is history

Each feature gets its own folder. The folder name encodes date + slug. After merge, the folder becomes documentation of *how* this feature came to exist.

### Layer 3: Decisions (immutable, append-only)

- `docs/adr/ADR-001-dapper-instead-of-ef.md`
- `docs/adr/ADR-002-quartz-for-scheduler.md`

An old decision never changes. If you change your mind, write a new ADR with `Supersedes: ADR-001`. The original stays as historical record.

---

## Workflow for Changes

### Small change (bugfix, minor feature)

New folder in `specs/`, one `spec.md`, implementation, done. Don't touch `ARCHITECTURE.md`.

### Medium change (new module, endpoint, integration)

Full `spec.md` + `plan.md` + `tasks.md` in `specs/`. If it changes a layer contract — update `ARCHITECTURE.md`. If it adds a domain concept — update `DOMAIN.md`.

### Large change (refactor, stack change, new pattern)

First write an ADR (why, alternatives rejected, consequences). Only then write an implementation spec that references the ADR.

### Concrete Example

Say you have a deployed `OneXmlOneFileStrategyMerge` and want to add ZIP compression per batch:

```
specs/2026-02-campaign-zip-per-batch/
├── spec.md
│   # Goal: ZIP per batch instead of per campaign
│   # Affects: OneXmlOneFileStrategyMerge, FlagFileWriter
│   # Does NOT affect: SFTP delivery, T_CAMPAGINES_LOG schema
├── plan.md
│   # Decisions: SharpZipLib (already in solution), streaming, not in-memory
│   # Files to modify: ...
│   # Files to add: ZipBatchWriter.cs
└── tasks.md
    # 1. Integration test for new behavior (red)
    # 2. ZipBatchWriter + unit tests
    # 3. Integration in MergeFilesIntoOneXml
    # 4. Quartz config migration (if interval changes)
```

In `CLAUDE.md` you add one line in the "Conventions" section: *"Batch compression: ZipBatchWriter, never manually ZipArchive."* That's it.

### Practical Workflow Tips

**Context per task.** Don't load all specs into the agent. For a new feature provide: `CLAUDE.md` + `ARCHITECTURE.md` + relevant `DOMAIN.md` fragment + 1-2 old specs of similar features as templates. The rest is noise.

**Snapshot before large refactors.** Before starting a major rewrite, generate `docs/snapshots/2026-02-pre-refactor.md` describing the current state. The agent (and you, three months later) will know where you started.

**Write ADRs after the fact when you must.** Ideally an ADR is written before the decision. In practice you often discover a pattern after 3 similar implementations — then write the ADR retroactively and link future specs to it.

**"Impact on existing code" section in every spec.md.** Forces thinking about side effects before the agent starts generating. This is where you catch "ah, this breaks the `IBaseHandler<TSelf>` contract" *before* four hours of debugging.

**Link spec to PR.** In the PR description: `Implements: specs/2026-01-campaign-retry/`. After merge, append the PR number to spec.md. After a year you have full traceability: feature → spec → code → decision.

---

## Architecture Decision Records (ADR) in Depth

ADRs were introduced by Michael Nygard in 2011. The premise: **code shows *what* you did, but not *why*. ADRs fill that gap.**

### When to Write an ADR

Write an ADR when a decision:
- is hard to reverse (ORM choice, message broker, authorization approach)
- has consequences beyond a single module (logging convention across the system)
- was made against the obvious choice ("why Dapper when .NET standard is EF?")
- keeps resurfacing in discussions ("maybe EF after all?") — an ADR ends this permanently

### When NOT to Write an ADR

- Variable naming, formatting, code style (goes to `CLAUDE.md` or `.editorconfig`)
- Decisions inside a single feature (goes to `spec.md`)
- Things obvious from the stack ("we use async/await" — no thanks)

**Practical test:** if a year from now someone asks "why the hell did we do it this way?" — that's an ADR.

### Structure (Nygard's format, lightly modified)

```markdown
# ADR-007: Dapper as the Data Access Layer

## Status
Accepted — 2026-01-15

## Context
The system processes campaigns with CHD/SKD files, where critical needs include:
- batch query performance (10k+ records per campaign)
- full control over SQL (legacy SCH_FILE_IMPORT schema, many joins)
- debugging simplicity — team knows T-SQL better than LINQ

EF Core would be the natural choice for a new .NET 8 project, but:
- EF migrations don't align with our existing DDL-diff process
- LINQ-to-SQL produces unpredictable plans on complex queries
- our repositories would write raw SQL via FromSqlRaw anyway

## Decision
We use Dapper as the sole data access layer.
Repositories are hand-written, one class per aggregate (`FileImportTablesRepository`,
`CampaignLogRepository`). SQL lives in constants in the repo class, not in .sql files.

## Consequences
**Positive:**
- full control over generated SQL
- easier code review (you see exactly what hits the database)
- no hidden N+1
- smaller dependency graph

**Negative:**
- manual DTO ↔ record mapping (boilerplate)
- no change tracking — updates require explicit SQL
- integration tests matter more than with EF (less "free" type validation)

**Neutral:**
- DDL migrations handled by separate tooling (DbUp or custom scripts)
- `Dapper.Contrib` only for simple CRUD, never for joins

## Alternatives Rejected
- **EF Core 8** — see Context
- **NHibernate** — abandoned by community, younger devs don't know it
- **Linq2Db** — interesting, but smaller community and less familiar to team

## References
- Spike: PR #87 (EF vs Dapper comparison on T_CAMPAGINES_LOG query)
- Discussion: architecture meeting notes 2026-01-10
```

For smaller decisions, use **MADR** (Markdown ADR) — a trimmed template with just Status / Context / Decision / Consequences.

### ADR Lifecycle and the Supersedes Pattern

Standard statuses:
- **Proposed** — proposal, not yet accepted, under discussion
- **Accepted** — in force, we follow it
- **Deprecated** — discouraged, but no replacement yet
- **Superseded by ADR-XXX** — replaced by a newer decision

**Key rule: you never edit the content of an old ADR.** You change only the status header and add a link. The full history must remain reconstructible.

Example: imagine ADR-002 said *"Quartz.NET with configuration in appsettings.json"*. A year later you need per-environment configuration from a database. You create **ADR-014**:

```markdown
# ADR-014: Quartz Configuration from Database Instead of appsettings

## Status
Accepted — 2026-05-20
Supersedes: ADR-002

## Context
ADR-002 (2025-03-10) established Quartz configuration in appsettings.json.
Since then new requirements emerged:
- different schedules per environment (DEV/TEST/PROD)
- ability to change cron expressions without deployment
- audit trail of schedule changes

## Decision
[...]

## Migration
- Existing jobs stay on config until end of Q3 2026
- New jobs go to database immediately
- Full migration: spec 2026-Q3-quartz-db-config
```

In parallel, you edit **only the header of ADR-002**:

```markdown
# ADR-002: Quartz.NET with Configuration in appsettings.json

## Status
Superseded by ADR-014 — 2026-05-20

## Context
[unchanged — history must stay]
...
```

### ADR Anti-Patterns

1. **ADR as post-mortem documentation.** ADRs should be written *at the moment of decision*, not six months later "because we should document it." Late ADRs are okay as exceptions, not the rule.

2. **ADR as a manual.** The "Decision" section should be short — 3-5 sentences. You don't explain how Dapper works. You explain that you choose Dapper, and why *here specifically*.

3. **ADR without an "Alternatives Rejected" section.** This is the most valuable part for future readers. Without it, the ADR reads like "we did X because we did X."

4. **Editing old ADRs.** The temptation is huge ("this is outdated, let me fix it"). Resist it. Create a new ADR with Supersedes.

5. **Numbering with gaps.** Number sequentially from ADR-001. No ADR-002.1, no ADR-002-v2. Decision changes → new number.

6. **ADR for reversible things.** "We use Serilog instead of NLog" — okay, ADR makes sense, changing loggers is work. "Our DTO classes have a `Dto` suffix" — no, that's a convention, goes to `CLAUDE.md`.

### How ADRs Work with AI Agents

ADRs are the **source of truth for the agent** when it makes decisions the spec file doesn't cover.

Practical setup:
- In `CLAUDE.md`, have a section: *"Before introducing an architectural decision, check `docs/adr/`. Active decisions: ADR-001, ADR-003, ADR-007, ADR-014."* (only Accepted, no Superseded)
- When the agent proposes something contradicting an ADR — redirect it to the specific file: *"Read ADR-007 and change your approach"*
- When the agent has a justified suggestion to change direction — that's a signal it might be time for a new ADR with Supersedes

### Tooling

- **adr-tools** (Node/Bash) — simplest, generates numbered skeletons: `adr new "Database choice"`
- **log4brains** — generates a static site from ADRs, nice to browse
- **MADR template** — Markdown template alone, no tooling, closest to how most teams actually work

For a solo or small team — a `docs/adr/` folder and numbering convention is enough. Add tooling only when you have 30+ ADRs and need search.

**Golden rule:** a short ADR written today is better than a perfect one written never. Your first five ADRs will be imperfect. By the tenth you'll have developed a style that fits how you work.

---

## Additional Files Worth Adding

Beyond what we've covered (`CLAUDE.md`, `ARCHITECTURE.md`, `DOMAIN.md`, ADRs, specs, PRD), consider these categories — from most important to "nice to have."

### Critical (should exist early)

**`RUNBOOK.md` / `OPERATIONS.md`** — what to do when things break. How to restart Quartz jobs, how to check SFTP state, where the logs live, how to diagnose a stuck campaign. This is the document that saves you at 3 a.m. The agent also uses it when you ask for diagnostic scripts.

**`SECURITY.md`** — how to report vulnerabilities (for public repos or projects with clients), but also internally: where secrets live, how we rotate them, what we *never* commit. Essential for projects with GDPR/RODO obligations.

**`.env.example` + `CONFIG.md`** — complete list of environment variables with descriptions and allowed values. Without this, onboarding a new developer (or agent) takes hours instead of minutes.

**`CONTRIBUTING.md`** — branch naming, commit format (Conventional Commits?), squash vs merge, PR checklist. These are the rules an AI agent can lean on when generating commit messages.

### Very Useful (second tier)

**`GLOSSARY.md`** — separately or as a section of `DOMAIN.md`. Dictionary of business terms: CHD, SKD, FLG, batch, campaign, broker. Without this, the agent mixes terminology and you spend time on corrections. This is the document that *most improves* generated code quality, because variable and class names start being consistent.

**`API.md` or OpenAPI/Swagger spec in the repo** — contract with the outside world. If you have a public API, generate `openapi.yaml` and keep it in the repo. The agent reads it before adding an endpoint and doesn't invent new conventions.

**`TESTING.md`** — testing strategy. What's tested in unit, what in integration, where Testcontainers, naming conventions, coverage expectations. Stops the agent from writing tests in its "own" style.

**`CHANGELOG.md`** — in Keep a Changelog format. Client, developer, agent — everyone sees what changed between versions. Written manually or generated from Conventional Commits.

**`ROADMAP.md`** — what you plan in the coming quarters. Not details, just direction. Gives the agent (and new joiners) context on *where* we're going, so proposed solutions don't conflict with future plans.

### Domain-Specific

**`docs/integrations/`** — one file per external integration. `sftp-velobank.md`, `inpost-shipx.md`, `gemini-image-gen.md`. Each contains: endpoints, auth, limits, quirks, sample payloads, partner-side contacts. Invaluable when the agent generates integration code — without it, it improvises based on general API knowledge.

**`docs/data-flows/`** — diagrams (Mermaid in markdown) of main flows: how a campaign flows from CHD file to delivery, how brokerage mailing works. Mermaid renders natively in GitHub/GitLab; the agent can also read it.

**`docs/schemas/`** — DDL of current schema + ERD. `SCH_FILE_IMPORT.sql`, `SCH_CAMPAGINES.sql`. Plus diff migrations in a separate folder. The agent reading the current schema won't invent column names.

**`docs/decisions-log.md`** (or `docs/journal.md`) — quicker than ADR notes: *"2026-05-15: tried Polly for SFTP retry, works, but waiting for permanent decision."* These are in-flight decisions not yet worth a full ADR, but you want to remember them. Review quarterly and either promote to ADR or discard.

### For Team and Process

**`ONBOARDING.md`** — from `git clone` to a working environment. Step by step. If a new person needs more than a day, this document is bad. The agent uses it when you ask for setup scripts.

**`docs/templates/`** — PR templates, issue templates, spec.md template, ADR template. Anything you repeat on every change becomes a template.

**`docs/postmortems/`** — incident analyses. *"2026-03-12: SFTP went down, how we diagnosed it, lessons learned."* Gold for the agent, because it sees what mistakes to *avoid*.

### Nice to Have

**`LICENSE`** — obvious, but often skipped in commercial projects.

**`docs/research/`** — your notes from spikes, experiments, library comparisons. Often contain conclusions that never reach the code but remain valuable. *"PDF generation library comparison, 3 options, 2026-02."*

**`docs/external-resources.md`** — links to partner documentation, articles that inspired decisions, talks. Source attribution = easier verification in the future.

---

## Repository Organization

Practical layout for a mid-sized .NET project:

```
/
├── README.md
├── CLAUDE.md                    # agent always reads
├── ARCHITECTURE.md              # agent always reads
├── DOMAIN.md                    # agent always reads
├── CONTRIBUTING.md
├── CHANGELOG.md
├── SECURITY.md
├── ROADMAP.md
├── LICENSE
├── .env.example
├── docs/
│   ├── adr/
│   │   ├── ADR-001-dapper.md
│   │   └── ADR-002-quartz.md
│   ├── prd/                     # archive, reference only
│   │   └── 2025-12-original-prd.md
│   ├── runbooks/
│   ├── integrations/
│   ├── data-flows/
│   ├── schemas/
│   ├── postmortems/
│   ├── research/
│   ├── templates/
│   ├── snapshots/
│   ├── GLOSSARY.md
│   ├── TESTING.md
│   ├── ONBOARDING.md
│   ├── CONFIG.md
│   └── decisions-log.md
└── specs/
    ├── _template/
    │   └── spec.md
    ├── 2026-01-campaign-retry/
    │   ├── spec.md
    │   ├── plan.md
    │   └── tasks.md
    └── 2026-02-brokerage-csv-validation/
        └── ...
```

---

## Working with the Agent: Practical Commands

All the documentation in the world is useless if you don't actually use it when prompting the agent. This section shows concrete prompts for common situations.

### Starting a New Session

Begin every non-trivial session by anchoring the agent in the relevant context. Don't assume it remembers anything from before — even in long-running tools like Claude Code, fresh context windows are the norm.

**For a new feature:**

```
Read CLAUDE.md, ARCHITECTURE.md, and DOMAIN.md.
Then read specs/_template/spec.md to understand our spec format.
We're starting a new feature: [short description].
Before writing any code, draft specs/[date-slug]/spec.md based on the template.
List open questions at the end — don't fill them in yourself.
```

**For modifying an existing feature:**

```
Read CLAUDE.md and ARCHITECTURE.md.
We're modifying [feature]. Read its original spec at specs/[folder]/spec.md
and the relevant code at [paths].
I want to: [change description].
Before changing anything, summarize:
1. Which files will need to change
2. Which conventions or ADRs apply
3. What you would do — wait for my approval before writing code.
```

The pattern is the same: **load context → state intent → propose plan → wait for green light**. Don't let the agent jump straight to code.

### After Adding a New ADR

When you add a new ADR (especially Accepted or one that Supersedes another), the agent doesn't magically know. Tell it explicitly.

**For a new Accepted ADR:**

```
I just added docs/adr/ADR-014-quartz-db-config.md.
Read it carefully. Then:
1. Update the "Active decisions" list in CLAUDE.md (add ADR-014)
2. Check if any other docs reference the old approach and need updating
3. Tell me what code in the current codebase contradicts this ADR
   — do NOT change code yet, just list the locations.
```

**For an ADR that Supersedes another:**

```
ADR-014 supersedes ADR-002. I've already updated ADR-002's status header.
Please:
1. Update CLAUDE.md's "Active decisions" list — remove ADR-002, add ADR-014
2. Grep the codebase and specs/ for references to ADR-002
3. For each reference, recommend whether it stays (historical context)
   or needs updating to point to ADR-014.
```

**For documenting a decision after the fact:**

```
We just decided to use [X] instead of [Y] for [reason].
Draft an ADR following the template in docs/adr/_template.md.
Use the next available number. Status: Accepted, today's date.
Fill in Context, Decision, Consequences, and Alternatives Rejected.
Leave References empty — I'll add them.
Show me the draft before creating the file.
```

### After Creating a Spec — Starting Implementation

A spec is ready. Now you want the agent to implement it. Don't just say "code it up."

```
We're implementing specs/2026-02-campaign-zip-per-batch/.
Read spec.md, plan.md, and tasks.md in that folder.
Also read CLAUDE.md and the referenced source files in plan.md.

Work task by task from tasks.md:
1. Before each task, restate what you're about to do
2. Write the test first (if applicable per TESTING.md)
3. Implement
4. Wait for me to confirm before moving to the next task

Do NOT skip ahead. Do NOT add features not in the spec.
If something in the spec is unclear, stop and ask — don't guess.
```

For smaller specs you can compress this, but the principle holds: **task by task, with checkpoints**.

### After Implementing — Updating Documentation

This is the step most teams skip, and it's why repos rot. After merging code, the agent should help maintain the docs that just became outdated.

```
We just merged PR #123 implementing specs/2026-02-campaign-zip-per-batch/.
Please:
1. Append "STATUS: shipped (PR #123, 2026-05-21)" to spec.md
2. Check if ARCHITECTURE.md needs updating (new component, changed boundary?)
3. Check if DOMAIN.md needs updating (new term introduced?)
4. Check if CLAUDE.md conventions section needs a line about ZipBatchWriter
5. Show me proposed diffs — I'll approve each one separately.
```

This 5-minute habit prevents documentation drift over months.

### After a Major Refactor

Refactors invalidate large parts of context. Help the agent re-orient.

```
We just completed a major refactor of the campaign processing pipeline.
Before we continue work:
1. Read the current state of src/Campaigns/ and src/Infrastructure/
2. Compare against ARCHITECTURE.md — list discrepancies
3. Propose updates to ARCHITECTURE.md to reflect reality
4. Create docs/snapshots/2026-05-post-refactor.md describing the new state
   (for future reference and rollback context).
```

Snapshots are cheap insurance. You almost never need them, but when you do they're priceless.

### When the Agent Drifts (Proposes Something Against Your Conventions)

This will happen. Often. The fix is simple but requires discipline.

**Surgical correction:**

```
Stop. What you're proposing contradicts ADR-007 (we use Dapper, not EF).
Re-read ADR-007 and revise your approach.
```

**When the agent persists:**

```
You're still suggesting EF patterns. Two possibilities:
1. You forgot ADR-007 — re-read it now and confirm you understand
2. You think ADR-007 should change — if so, make the case for a new ADR
   that supersedes it. Don't sneak EF in via the back door.
```

**When the agent has a point:**

```
You're right that ADR-007 doesn't cover this case.
Draft an ADR-015 that extends ADR-007 for [specific scenario].
Don't supersede ADR-007 — extend it. Show me the draft.
```

The pattern: **named documents in your prompts**. "Per our conventions" is weak. "Per ADR-007 section Consequences" is strong.

### Asking the Agent to Maintain Documentation Proactively

Train the agent to flag documentation gaps as it works.

Add this to `CLAUDE.md`:

```markdown
## Documentation Maintenance Rules
When working on any task, if you encounter:
- A convention not documented in CLAUDE.md → propose adding it
- A decision made implicitly that should be an ADR → flag it
- Terminology used inconsistently → propose a GLOSSARY entry
- An integration without docs in docs/integrations/ → propose creating one

Always propose, never edit docs without explicit approval.
```

Then in sessions, occasionally ask:

```
Based on our work today, what documentation gaps did you notice?
List them with priority (critical / nice-to-have).
```

### Ending a Session

The last 60 seconds of a session matter more than the first 10 minutes.

```
We're wrapping up. Please produce a session summary:
1. What we accomplished (with PR/spec references)
2. What's left in current tasks.md (with status)
3. Any decisions made that aren't yet in ADRs or specs
4. Any documentation drift you noticed and didn't address
5. Suggested next session starting prompt.

Save it as specs/[current-spec]/session-notes-[date].md
```

This becomes the perfect starter context for your next session — by you or by the agent.

### A Few Universal Prompting Patterns

These work across all situations:

**"Plan before code":**
```
Don't write code yet. First, outline what you'll do in 5-10 bullet points.
I'll approve or correct before you start implementing.
```

**"Cite your source":**
```
For each decision in your proposal, cite the document that supports it
(CLAUDE.md section, ADR number, spec path).
If a decision isn't grounded in any document, mark it as [ASSUMPTION]
and we'll discuss before proceeding.
```

**"Diff, don't replace":**
```
Show me proposed changes as diffs against the current file,
not as a full rewrite. I want to see exactly what changes.
```

**"Question before assumption":**
```
If anything in this task is ambiguous, list your questions before starting.
I'd rather answer 5 questions now than refactor 50 lines later.
```

**"Stay in scope":**
```
The spec is specs/2026-02-x/. Do not modify any file outside the paths
listed in plan.md. If you think a change outside scope is necessary,
stop and propose extending the spec — don't silently expand.
```

### Anti-Patterns in Agent Interaction

What NOT to do:

1. **"Just figure it out."** The agent will. The result will not match your conventions.

2. **"Use best practices."** Whose best practices? Yours are in CLAUDE.md. Reference them.

3. **"Make it production-ready."** Meaningless. Say what production-ready means in your context (logging, error handling, tests, observability) — or point to TESTING.md and CLAUDE.md.

4. **Loading every doc on every prompt.** Wastes tokens, dilutes attention. Pick relevant context per task.

5. **Skipping the post-implementation doc update.** This is how repos rot. The discipline is: spec → code → docs update, every time.

6. **Letting the agent write ADRs unsupervised.** ADRs are decisions you own. The agent can draft. You approve.

7. **Asking the agent to "remember" something across sessions.** It can't. Write it down in CLAUDE.md, or it doesn't exist.

---

## Golden Rules

1. **Don't create documents on spec.** Add the next one when:
   - You explain the same thing to the agent for the third time (→ this should be a document)
   - You return to old code and don't remember why (→ ADR or journal)
   - You onboard someone and notice something obvious is missing (→ ONBOARDING or CONFIG)
   - The agent generates something against an unwritten convention (→ CLAUDE.md or TESTING.md)

2. **A document without an owner is dead.** Every file you add must answer: *"who reviews this, and when?"*

3. **A repo with 30 stale markdown files is worse than a repo with 5 always-fresh ones.** Discipline > completeness.

4. **PRD is a starting artifact, not a living document.** Freeze after v1 ships.

5. **ADRs are immutable. Specs freeze after shipping. Only the stable layer evolves continuously.**

6. **Context per task, not context per project.** Don't dump everything on the agent. Pick what's relevant.

7. **A short ADR written today beats a perfect one written never.**

8. **The plan is alive, but decision history must be visible.** Move open questions to decisions; never delete them.

9. **Code shows what. Documentation shows why.** If the why isn't written down, it doesn't exist.

10. **Spec-driven development is a discipline, not a methodology.** Apply it daily, not as a one-time event.

---

*This guide reflects practical patterns from real-world .NET projects using Claude Code, Copilot, and similar AI coding agents. Adapt to your stack and team size.*
