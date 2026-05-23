# Spec-Driven Development with AI Coding Agents

> A practical guide to structuring your repository so AI coding agents (Claude Code, Copilot, Cursor) produce consistent, maintainable code instead of drifting away from your intent.

---

## Table of Contents

1. [Core Philosophy](#core-philosophy)
2. [Who Practices SDD](#who-practices-sdd)
3. [The PRD-First Approach](#the-prd-first-approach)
4. [PRD vs Spec — Two Layers, Same Question](#prd-vs-spec--two-layers-same-question)
5. [Writing a Good PLAN.md](#writing-a-good-planmd)
6. [The Three-Layer Documentation Model](#the-three-layer-documentation-model)
7. [Workflow for Changes](#workflow-for-changes)
8. [Architecture Decision Records (ADR) in Depth](#architecture-decision-records-adr-in-depth)
9. [The Absolute Minimum](#the-absolute-minimum)
10. [Migrating from a Legacy Repo to SDD](#migrating-from-a-legacy-repo-to-sdd)
11. [Additional Files Worth Adding](#additional-files-worth-adding)
12. [Repository Organization](#repository-organization)
13. [Working with the Agent](#working-with-the-agent)
14. [Golden Rules](#golden-rules)

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

## Who Practices SDD

*"Spec-driven development"* as a *named* methodology is recent — it crystallized in 2024–2025 around the rise of AI coding agents. The underlying disciplines, though, have decades of track record across the industry.

**Explicit AI-era SDD adopters:**

- **GitHub** ships [`spec-kit`](https://github.com/github/spec-kit) — the most direct, named SDD reference implementation, with 100k+ stars. The `spec.md → plan.md → tasks.md` flow in this guide mirrors theirs.
- **Anthropic** treats `CLAUDE.md` as a first-class concept in Claude Code, with public engineering guidance that maps onto the same workflow.
- **Cursor** (`.cursorrules`), **Aider** (conventions files), **Continue.dev** (`.continuerules`), and other agent tools have independently converged on the same *"one file the agent always reads"* pattern.

**Pre-AI organizations practicing spec-adjacent discipline:**

- **Amazon** — *Working Backwards* / PR-FAQ memos: new products start with a fake press release plus an FAQ. Structurally a PRD before any code exists.
- **Google** — design-doc culture by default; Malte Ubl's [*Design docs at Google*](https://www.industrialempathy.com/posts/design-docs-at-google/) is the canonical reference.
- **Stripe** — RFC-driven engineering; written prose treated as a first-class deliverable.
- **Basecamp / 37signals** — [Shape Up](https://basecamp.com/shapeup) pitches map almost 1:1 onto modern SDD specs.

**Methodologies that predate SDD and contributed its patterns:**

IETF RFCs (1969+), Architecture Decision Records (Michael Nygard, 2011), the C4 model (Simon Brown), the Diátaxis documentation framework (Daniele Procida), and pre-mortems (Gary Klein) — all share DNA with what SDD does today.

For the fuller list, the verified references, and a discussion of adoption depth (vs. depth-of-marketing), see [`sdd-in-the-wild.md`](sdd-in-the-wild.md).

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

## PRD vs Spec — Two Layers, Same Question

PRDs and feature specs look similar at a glance — both have "scope" sections, both describe "what we're building." New teams ask: *do we really need both?*

Yes. They live at different layers and have different lifetimes.

| | PRD | Spec |
|---|---|---|
| **Scope** | The whole product/system | A single feature or change |
| **Lifetime** | Written once, frozen after v1 ships | Written before the PR, frozen after merge |
| **Granularity** | *"Build an order delivery platform"* | *"Add ZIP-per-batch compression to `BatchXmlMerger`"* |
| **Audience** | Stakeholders, founding team | Developers, the AI agent about to write code |
| **Frequency** | Maybe once in the project's life | Dozens or hundreds per year |
| **Owner** | Product / founder | Engineer driving the change |

### Concrete example

```
PRD (2025-01): "Build an order delivery platform with HDR/DTL
                files, SFTP delivery, partner mailing."
   ↓
spec (2026-01-order-retry): "Add retry-with-backoff to SFTP
                                delivery on transient failures."
spec (2026-02-zip-per-batch):  "Switch from per-order ZIPs
                                to per-batch ZIPs."
spec (2026-03-...): ...
```

The PRD answers *why this system exists*. Specs answer *what changed this week*.

### The trap each avoids

**Specs without a PRD:** six months in, nobody remembers why retries even matter — the partner SLA requires < 1% failure rate, but that requirement lives only in someone's head. New engineers (and the AI agent) propose "simpler" solutions that miss the point.

**PRD without specs:** the PRD becomes a Frankenstein document where every change request gets appended. Original v1 intent dilutes, decision history disappears, and after two years the PRD reads like a press release for a product that doesn't exist.

Two artifacts, two lifetimes, two purposes. The overlap in fields is real but cosmetic — the level of detail is completely different.

---

## Writing a Good PLAN.md

A good `PLAN.md` answers questions the agent would otherwise guess at. Every guess is a potential drift from intent.

### Principles That Work

- **Concrete decisions, not options.** Don't write "consider MediatR or services" — pick one. A plan is not a brainstorm.
- **Separate what (requirements) from how (architecture).** Mixing them produces code that mixes layers.
- **Explicit exclusions.** An "out of scope" section prevents the agent from adding features you don't want now.
- **Testable acceptance criteria.** "Works correctly" → bad. "POST /api/x with payload Y returns 201 and a row in event_log" → good.
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
- Naming conventions: lowercase snake_case for schemas/tables (e.g. `app.orders`), primary keys named `id`

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
1. DDL migration script for `app.x`
2. `XRepository` + unit tests for queries
3. `XHandler` (pure logic, no I/O in tests)
4. `POST /api/x` endpoint + FluentValidation
5. Integration tests with Testcontainers
6. Quartz job registration (if applicable)

## Acceptance Criteria
- [ ] `POST /api/x` with valid payload → 201 + row in `x`
- [ ] Missing required field → 400 with specific message
- [ ] Integration test covers happy path + 2 error cases
- [ ] Each test runs against an isolated database

## Constraints
- Do NOT use EF Core
- Do NOT add CQRS/MediatR — handlers called directly
- Maintain consistency with `BatchXmlMerger` pattern

## Open Questions
- [ ] Does `x` use soft-delete or hard?
- [ ] Retry policy for SFTP — Polly or custom?

## References
- `src/.../OrderImportRepository.cs` — repo pattern to follow
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

- `specs/2026-01-order-retry/spec.md` + `plan.md` + `tasks.md`
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

Say you have a deployed `BatchXmlMerger` and want to add ZIP compression per batch:

```
specs/2026-02-order-zip-per-batch/
├── spec.md
│   # Goal: ZIP per batch instead of per order
│   # Affects: BatchXmlMerger, ReadyFileWriter
│   # Does NOT affect: SFTP delivery, order_log schema
├── plan.md
│   # Decisions: SharpZipLib (already in solution), streaming, not in-memory
│   # Files to modify: ...
│   # Files to add: ZipBatchWriter.cs
└── tasks.md
    # 1. Integration test for new behavior (red)
    # 2. ZipBatchWriter + unit tests
    # 3. Integration in MergeFilesIntoBatchXml
    # 4. Quartz config migration (if interval changes)
```

In `CLAUDE.md` you add one line in the "Conventions" section: *"Batch compression: ZipBatchWriter, never manually ZipArchive."* That's it.

### Practical Workflow Tips

**Context per task.** Don't load all specs into the agent. For a new feature provide: `CLAUDE.md` + `ARCHITECTURE.md` + relevant `DOMAIN.md` fragment + 1-2 old specs of similar features as templates. The rest is noise.

**Snapshot before large refactors.** Before starting a major rewrite, generate `docs/snapshots/2026-02-pre-refactor.md` describing the current state. The agent (and you, three months later) will know where you started.

**Write ADRs after the fact when you must.** Ideally an ADR is written before the decision. In practice you often discover a pattern after 3 similar implementations — then write the ADR retroactively and link future specs to it.

**"Impact on existing code" section in every spec.md.** Forces thinking about side effects before the agent starts generating. This is where you catch "ah, this breaks the `IBaseHandler<TSelf>` contract" *before* four hours of debugging.

**Link spec to PR.** In the PR description: `Implements: specs/2026-01-order-retry/`. After merge, append the PR number to spec.md. After a year you have full traceability: feature → spec → code → decision.

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
The system processes orders with HDR/DTL files, where critical needs include:
- batch query performance (10k+ records per order)
- full control over SQL (`legacy_import` schema with many joins)
- debugging simplicity — team knows T-SQL better than LINQ

EF Core would be the natural choice for a new .NET 8 project, but:
- EF migrations don't align with our existing DDL-diff process
- LINQ-to-SQL produces unpredictable plans on complex queries
- our repositories would write raw SQL via FromSqlRaw anyway

## Decision
We use Dapper as the sole data access layer.
Repositories are hand-written, one class per aggregate (`OrderImportRepository`,
`OrderLogRepository`). SQL lives in constants in the repo class, not in .sql files.

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
- Spike: PR #87 (EF vs Dapper comparison on order_log query)
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

## The Absolute Minimum

If you're starting today and don't want to overinvest in docs upfront, what's the smallest set of markdown files that still buys you SDD's benefits?

### The single most important file

**`CLAUDE.md`** at the repo root (or `.cursorrules` for Cursor, equivalent files for other agent tools). If you do nothing else from this guide, do this. It's the only file most agents load automatically; without it your conventions don't exist as far as the agent is concerned. One file the agent always reads beats ten files it might never reach.

### Floor: 3 files (do not go lower)

1. **`CLAUDE.md`** — conventions, what NOT to do, pointers to other docs. The agent's instruction hub.
2. **`README.md`** — what this project is, how to run it. Entry point for humans (and a fallback for agents).
3. **One spec file** — `specs/<date-slug>/spec.md` for the current change you're working on. Even if it's 20 lines, even if it's for a single PR.

Below three, you're hoping, not specifying.

### Practical minimum: 5 files (recommended starting set)

For any project beyond a weekend hack, add two more:

4. **`ARCHITECTURE.md`** — high-level structure: layers, modules, key boundaries. A diagram plus ~200 words of prose is enough on day one. Grow it as the system grows.
5. **`DOMAIN.md`** (or `GLOSSARY.md`) — business terminology, abbreviations, the words specific to your problem space. Skip if your domain is generic (CRUD on things); essential if it has jargon (finance, biotech, logistics, regulated industries).

Five files cover most of the SDD benefit for a project in the 0–3 month range.

### Add the rest reactively, not proactively

Beyond the starter set, each new doc should be triggered by something concrete — not by *"the methodology mentions it."*

| File | Trigger to create it |
|------|----------------------|
| `docs/adr/ADR-001-*.md` | First non-trivial, hard-to-reverse decision (ORM choice, message broker, auth approach) |
| `RUNBOOK.md` / `OPERATIONS.md` | First production incident, or first deploy to a real environment |
| `TESTING.md` | Third time you explain the test conventions to the agent |
| `.env.example` + `CONFIG.md` | First time onboarding takes > 1 hour |
| `CONTRIBUTING.md` | First external contributor, or first time you forget your own commit convention |
| `CHANGELOG.md` | First time a user asks *"what changed in this version?"* |
| `docs/integrations/<vendor>.md` | First time the agent generates wrong code against an external API |
| `docs/postmortems/<date>.md` | First incident worth not repeating |

A `RUNBOOK.md` written before you have a system in production is fiction; a `CHANGELOG.md` written before you have users is busywork. Wait for the trigger; the act of writing the doc *while the problem is fresh* is what makes it useful.

### A rule of thumb for any candidate file

If you can't say *who reads this file, and when*, you don't need it yet.

For each candidate doc, answer:

- **Who reads it?** You? New hires? The agent? Auditors?
- **When?** Daily? On incident? Before a change? Once at onboarding?
- **What goes stale if you don't update it?**

A file that fails all three is decoration — skip it.

See [Additional Files Worth Adding](#additional-files-worth-adding) for the fuller catalog organized by priority tier, and the first entry in [Golden Rules](#golden-rules) for the related discipline (*"Don't create documents on spec"*).

---

## Migrating from a Legacy Repo to SDD

The previous section described the starter set for a *new* project. What if you already have a 5-year-old codebase with thousands of files, no `CLAUDE.md`, scattered docs, and decisions buried in two-year-old Slack threads?

**The trap:** the "documentation sprint week" — block out two weeks, write 50 files, generate a wall of agent-confusing text. By week 3 the docs are stale and nobody trusts them. Worst of both worlds: lost feature velocity *and* docs nobody uses.

**The right shape:** incremental, agent-assisted, forward-leaning. Three rules:

1. **Foundation first, in 5–7 days, not 5–7 weeks.** Write `CLAUDE.md`, `ARCHITECTURE.md`, and (if relevant) `DOMAIN.md` *with* the agent reading your existing code. Heavy review, but not from scratch.
2. **Specs go forward, never backward.** Don't write a "spec" for code that already shipped — that's fan fiction. Every *new* change from migration day forward gets a spec; old features are documented only when something pulls you back into them.
3. **ADRs are reactive, not retrospective.** Write an ADR the first time the agent makes a bad suggestion that contradicts an unwritten decision. Don't try to backfill every decision ever made.

**Typical 90-day shape:**

- **Week 1**: Foundation (CLAUDE.md → ARCHITECTURE.md → DOMAIN.md), first ADR for the most-violated unwritten rule.
- **Week 2 onward**: every new change uses spec-driven workflow.
- **Months 1–3**: ADRs accumulate reactively as triggers fire; first runbook entry after first incident.
- **End of Q1**: roughly 10–15 markdown files, all earned by use, none decorative.

For the concrete migration scripts, agent prompts that draft each foundation file from existing code, anti-patterns (the big-bang sprint, fabricated-history ADRs, fan-fiction specs, CLAUDE.md as wall-of-text), and worked examples (Python web app, .NET monorepo, C# legacy, OSS project), see [`legacy-to-sdd-migration-guide.md`](legacy-to-sdd-migration-guide.md).

---

## Additional Files Worth Adding

Beyond what we've covered (`CLAUDE.md`, `ARCHITECTURE.md`, `DOMAIN.md`, ADRs, specs, PRD), consider these categories — from most important to "nice to have."

### Critical (should exist early)

**`RUNBOOK.md` / `OPERATIONS.md`** — what to do when things break. How to restart Quartz jobs, how to check SFTP state, where the logs live, how to diagnose a stuck order. This is the document that saves you at 3 a.m. The agent also uses it when you ask for diagnostic scripts. → See the companion [Runbook / Operations Documentation Guide](runbook-operations-guide.md) for templates, anti-patterns, and agent prompts.

**`SECURITY.md`** — how to report vulnerabilities (for public repos or projects with clients), but also internally: where secrets live, how we rotate them, what we *never* commit. Essential for projects with GDPR/RODO obligations.

**`.env.example` + `CONFIG.md`** — complete list of environment variables with descriptions and allowed values. Without this, onboarding a new developer (or agent) takes hours instead of minutes.

**`CONTRIBUTING.md`** — branch naming, commit format (Conventional Commits?), squash vs merge, PR checklist. These are the rules an AI agent can lean on when generating commit messages.

### Very Useful (second tier)

**`GLOSSARY.md`** — separately or as a section of `DOMAIN.md`. Dictionary of business terms: HDR, DTL, RDY, batch, order, partner. Without this, the agent mixes terminology and you spend time on corrections. This is the document that *most improves* generated code quality, because variable and class names start being consistent.

**`API.md` or OpenAPI/Swagger spec in the repo** — contract with the outside world. If you have a public API, generate `openapi.yaml` and keep it in the repo. The agent reads it before adding an endpoint and doesn't invent new conventions.

**`TESTING.md`** — testing strategy. What's tested in unit, what in integration, where Testcontainers, naming conventions, coverage expectations. Stops the agent from writing tests in its "own" style.

**`CHANGELOG.md`** — in Keep a Changelog format. Client, developer, agent — everyone sees what changed between versions. Written manually or generated from Conventional Commits.

**`ROADMAP.md`** — what you plan in the coming quarters. Not details, just direction. Gives the agent (and new joiners) context on *where* we're going, so proposed solutions don't conflict with future plans.

### Domain-Specific

**`docs/integrations/`** — one file per external integration. `sftp-acme-bank.md`, `carrier-shipping-api.md`, `imagegen-vendor.md`. Each contains: endpoints, auth, limits, quirks, sample payloads, partner-side contacts. Invaluable when the agent generates integration code — without it, it improvises based on general API knowledge.

**`docs/data-flows/`** — diagrams (Mermaid in markdown) of main flows: how an order flows from HDR file to delivery, how partner mailing works. Mermaid renders natively in GitHub/GitLab; the agent can also read it.

**`docs/schemas/`** — DDL of current schema + ERD. `legacy_import.sql`, `app.sql`. Plus diff migrations in a separate folder. The agent reading the current schema won't invent column names.

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

This is what a *mature* SDD repo looks like — a mid-sized project, ~6–12 months into adoption, with most of the optional files earned by use. It is **not** what you start with on day 1. For the day-1 minimum, see [The Absolute Minimum](#the-absolute-minimum); for the migration shape if you have an existing repo, see [`legacy-to-sdd-migration-guide.md`](legacy-to-sdd-migration-guide.md).

```
/
├── README.md                          # entry point for humans — what the project is, how to run
├── CLAUDE.md                          # entry point for the agent — conventions, what NOT to do, doc map
├── ARCHITECTURE.md                    # high-level structure — layers, modules, key boundaries
├── DOMAIN.md                          # business terminology, abbreviations, glossary (skip if generic)
├── CONTRIBUTING.md                    # branch/commit conventions, PR checklist, how to contribute
├── CHANGELOG.md                       # user-visible changes per version (Keep a Changelog format)
├── SECURITY.md                        # vulnerability reporting + internal secrets/rotation policy
├── ROADMAP.md                         # planned direction over coming quarters (not detailed plans)
├── LICENSE                            # legal — required for OSS, useful internally too
├── .env.example                       # template of required env vars (no real values committed)
├── docs/                              # everything that isn't a top-level signpost
│   ├── adr/                           # architecture decisions — immutable, numbered, append-only
│   │   ├── ADR-001-dapper.md          # one decision per file, status header on top
│   │   └── ADR-002-quartz.md
│   ├── prd/                           # original PRDs — archive, reference only (frozen after v1)
│   │   └── 2025-12-original-prd.md
│   ├── runbooks/                      # per-incident recovery procedures (the 3 a.m. file)
│   ├── integrations/                  # one file per external integration — endpoints, quirks, contacts
│   ├── data-flows/                    # Mermaid diagrams of main flows through the system
│   ├── schemas/                       # DDL of current schema + ERDs
│   ├── postmortems/                   # incident analyses — root cause, lessons, follow-ups
│   ├── research/                      # spike notes, library comparisons, experiments
│   ├── templates/                     # PR/issue/spec/ADR templates to copy from
│   ├── snapshots/                     # point-in-time captures before major refactors (rollback context)
│   ├── GLOSSARY.md                    # alternative home for DOMAIN.md content if you prefer it under docs/
│   ├── TESTING.md                     # test strategy, conventions, coverage expectations
│   ├── ONBOARDING.md                  # git clone → working environment, step by step
│   ├── CONFIG.md                      # env vars and config flags explained
│   └── decisions-log.md               # in-flight decisions not yet worth a full ADR (review quarterly)
└── specs/                             # per-feature specs, one folder per change
    ├── _template/                     # blank spec/plan/tasks to copy from
    │   └── spec.md
    ├── 2026-01-order-retry/           # date-slug naming — folder freezes after PR merges
    │   ├── spec.md                    # what + acceptance criteria
    │   ├── plan.md                    # how — decisions, file structure, task order
    │   └── tasks.md                   # execution checklist with checkboxes
    └── 2026-02-partner-csv-validation/
        └── ...
```

### How to read this layout

Three lifetimes are mixed together in the same tree:

- **Stable layer** — `CLAUDE.md`, `ARCHITECTURE.md`, `DOMAIN.md`. Updated as the codebase changes, but the *file itself* exists for the life of the project. The agent loads these (or is pointed at them by `CLAUDE.md`) nearly every session.
- **Append-only layer** — `docs/adr/`, `docs/postmortems/`, `CHANGELOG.md`. Each new entry is added; old entries are never edited (except for ADR status changes via `Supersedes`). The history is the value.
- **Frozen layer** — `docs/prd/`, `specs/<date-slug>/`. Written once, frozen after their moment passes (PRD: after v1 ships; spec: after the PR merges). Treat as historical artifacts.

A repo that confuses these layers — editing old ADRs, treating PRD as living, leaving specs perpetually unfinished — rots fastest.

### What you actually start with

Almost none of this on day 1. The earliest version of any SDD repo is:

```
/
├── README.md
├── CLAUDE.md
└── specs/
    └── <first-feature>/spec.md
```

Three files. Everything else is added when a specific trigger fires (see the trigger table in [The Absolute Minimum](#the-absolute-minimum)). A new repo that ships the full tree above on day 1 has docs that are 80% wrong because they were written without active context.

### Common variations

- **Monorepo with multiple services** — one root `CLAUDE.md` for cross-service conventions, plus a `CLAUDE.md` per service for service-specific stack/patterns. Same with `ARCHITECTURE.md` (root: service-level boundaries; per-service: internal structure).
- **Small project / solo dev** — collapse `docs/` content into top-level files until it gets unwieldy. A single `ARCHITECTURE.md` covering both structure and runbook is fine until it crosses ~600 lines.
- **OSS project** — heavier emphasis on `CONTRIBUTING.md` and `LICENSE`; lighter on `ROADMAP.md` and `RUNBOOK.md` (operational stuff is private). `CLAUDE.md` written with external contributors' agents in mind.
- **Regulated industry** — add `docs/compliance/` (audit trails, control mappings) alongside the standard tree. ADRs gain extra weight; throwing decisions in Slack is no longer an option.
- **Heavy data/ML work** — add `docs/datasets/` (provenance, schemas, sampling) and `docs/experiments/` (separate from `docs/research/` — experiments are reproducible artifacts, research is exploratory notes).

### The principle behind the layout

Every folder and every file in this tree should be there because **something pulled it into being** — a real reader, a real trigger, a real recurring need. The layout is a snapshot of what an organization that practiced SDD discipline for a year ended up with. It is not a checklist to copy onto day 1. If you find yourself adding files because *"the template says to,"* stop. The template is a description of the destination, not a prescription for the path.

---

## Working with the Agent

This guide describes *what* to write down: PRDs, specs, plans, ADRs, the three-layer documentation model. The companion guide [`working-with-agents-guide.md`](working-with-agents-guide.md) describes *how to put it in front of the agent* — concrete prompts for starting and ending sessions, drafting and superseding ADRs, implementing specs task-by-task, recovering from drift, plus the underlying mechanics:

- **When the agent loads a file** (and why it sometimes feels random)
- **How many files is too many** before the agent gets lost in your repo
- Universal prompting patterns (`Plan before code`, `Cite your source`, `Diff, don't replace`, …)
- Anti-patterns in agent interaction

Read it once before you start using the workflow described in this guide on a real project. It's the thinnest layer between the documentation conventions here and the prompts that put them into effect.

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
