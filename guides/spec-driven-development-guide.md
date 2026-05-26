# Spec-Driven Development with AI Coding Agents

> A practical guide to structuring your repository so AI coding agents (Claude Code, Copilot, Cursor) produce consistent, maintainable code instead of drifting away from your intent.

---

## Table of Contents

1. [Core Philosophy](#core-philosophy)
2. [Who Practices SDD](#who-practices-sdd)
3. [When SDD Pays Off (and When It's Overhead)](#when-sdd-pays-off-and-when-its-overhead)
4. [The Absolute Minimum](#the-absolute-minimum)
5. [The PRD Layer](#the-prd-layer)
6. [Writing a Good spec.md](#writing-a-good-specmd)
7. [Writing a Good PLAN.md](#writing-a-good-planmd)
8. [The Three-Layer Documentation Model](#the-three-layer-documentation-model)
9. [Workflow for Changes](#workflow-for-changes)
10. [SDD in Teams: Roles and Responsibilities](#sdd-in-teams-roles-and-responsibilities)
11. [Architecture Decision Records (ADR)](#architecture-decision-records-adr)
12. [Migrating from a Legacy Repo to SDD](#migrating-from-a-legacy-repo-to-sdd)
13. [Additional Files Worth Adding](#additional-files-worth-adding)
14. [Repository Organization](#repository-organization)
15. [Working with the Agent](#working-with-the-agent)
16. [Golden Rules](#golden-rules)

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

## When SDD Pays Off (and When It's Overhead)

SDD has costs. Writing specs takes time; maintaining docs takes time; coordinating around docs adds friction. *"Just prompt and let the agent figure it out"* is faster — for a while.

The question is when the overhead pays off. The answer turns on one thing: how much does **cross-prompt consistency** matter to you?

### Two components of agent output quality

The agent's output has two components:

1. **Single-prompt quality** — how well it writes code for *this one request*, in isolation.
2. **Cross-prompt consistency** — whether its output matches the rest of the codebase, your conventions, and your prior decisions.

Normal prompting handles (1) well even with no docs. Modern agents are good at writing code that works for a single, contained request.

Normal prompting fails at (2) the moment the codebase outgrows a single context window, the work crosses more than one session, or a second human (or agent) starts touching the same code. **SDD pays its cost specifically to maintain (2).**

If (2) doesn't matter to you, SDD is overhead. If it does, SDD is one of the highest-leverage investments you can make.

### SDD wins when any 2+ of these are true

- The project will be **touched again** after today (next week, next month, next year)
- **More than one person** (human or agent) will edit this code
- The code will **run in production** — anything that matters when nobody is watching
- The domain has **non-obvious logic** — finance, healthcare, regulated industries, complex B2B integrations
- Some decisions are **hard to reverse** (ORM, database schema, auth model, message broker)
- The agent will be invoked **across many sessions** against the same codebase

The more of these are true, the more leverage SDD provides. A long-lived production codebase touched by a team using multiple AI tools is the maximum-value case — and the one where the cost of *not* doing SDD compounds fastest.

### Normal prompting is fine when

- The code will be **deleted within a week**, or never touched again
- This is a **one-shot transformation** — convert this CSV, reformat this text, generate this SQL query
- A **single-session investigation** — Jupyter notebook for one analysis, then archived
- **Throwaway** code — hackathon, spike, learning exercise, prototype to validate one assumption
- The domain is **generic enough** that the agent's priors are reasonable defaults (CRUD on simple entities with no special rules)

In these cases, writing a `CLAUDE.md` is overhead. Just prompt.

### The middle ground: SDD-lite

For solo projects that *might* grow, or projects in the 1–3 month window where it's not yet clear if they'll survive:

- Write a `CLAUDE.md` (one file, ~100 lines)
- Use `specs/` for non-trivial changes; skip it for trivial ones
- Skip everything else until a specific trigger fires (see [The Absolute Minimum](#the-absolute-minimum))

That's the minimum-viable SDD investment. It costs ~2 hours upfront and captures most of the drift-prevention benefit without committing you to full SDD discipline. If the project survives 3 months, you'll already have the foundation in place.

### The hidden cost of NOT doing SDD

People often weigh *"the cost of writing docs"* against *"the speed of just prompting."* This misses the cost of NOT having docs:

- Time spent **re-explaining the same convention** every session
- Time spent **reviewing or refactoring** agent output that drifted from the rest of the code
- Decisions **re-litigated in PR review** (*"didn't we already discuss this?"*)
- **New contributors** (human or agent) starting from zero context
- **Old code that nobody understands** because the rationale lives in someone's head
- **Production incidents** where nobody knows the original constraints

The trade is: pay X% upfront in docs, save Y% over time in drift, rework, and onboarding friction. The X:Y ratio improves the longer the project lives and the more people touch it.

### A practical decision rule

If you can answer **yes** to two or more of these, the SDD overhead has already paid for itself:

- *"Will I (or anyone) work on this code three months from now?"*
- *"Will an AI agent touch this code in more than one session?"*
- *"Does this code have to behave consistently with other code in the same repo?"*
- *"Are there decisions here that would be expensive to reverse?"*

If the answer to all four is *no* — prompt away. Don't perform documentation theatre on code that's about to be deleted.

---

## The Absolute Minimum

If you're starting today and don't want to overinvest in docs upfront, what's the smallest set of markdown files that still buys you SDD's benefits?

### The single most important file

**`CLAUDE.md`** at the repo root (or `.cursorrules` for Cursor, equivalent files for other agent tools). If you do nothing else from this guide, do this. It's the only file most agents load automatically; without it your conventions don't exist as far as the agent is concerned. One file the agent always reads beats ten files it might never reach.

What separates a useful `CLAUDE.md` from a generic one — what to put in, what to leave out, how big it should get, how it changes when the repo has 30+ other markdown files competing for attention — is the subject of its own detail guide: [`claude-md-guide.md`](claude-md-guide.md). Read that one before writing your project's `CLAUDE.md` from scratch; the difference between a "wall of text" and a hub that routes the agent's attention well is mostly about the rules in that guide.

### Floor: 3 files (do not go lower)

1. **`CLAUDE.md`** — conventions, what NOT to do, pointers to other docs. The agent's instruction hub.
2. **`README.md`** — what this project is, how to run it. Entry point for humans (and a fallback for agents).
3. **One spec file** — `specs/<date-slug>/spec.md` for the current change you're working on (see [Writing a Good spec.md](#writing-a-good-specmd) below for what should be in it). Even if it's 20 lines, even if it's for a single PR.

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

## The PRD Layer

### Why PRDs exist

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

### PRD vs spec — two layers, same question

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

**Concrete example:**

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

## Writing a Good spec.md

A `spec.md` answers: *what are we building, why, and how do we know it works?* It's the document the agent reads before generating code; the document the human PR reviewer reads before approving; the document future-you reads when wondering *"why is this code shaped this way?"*

A spec is **per-feature** and **frozen after merge** — different from a PRD (whole-product, frozen after v1) and from a plan (the *how*, written alongside the spec — see [the next section](#writing-a-good-planmd)).

### Principles that work

- **Goal in three sentences.** What problem, for whom, in which system. Anything longer means the feature isn't clear in your head yet.
- **In-scope and out-of-scope, both explicit.** *"What we're NOT doing"* prevents scope creep more reliably than *"what we are doing"* does. The agent reads it; PR reviewers reference it.
- **Acceptance criteria as checkboxes.** Testable, specific, ideally automatable. *"`POST /api/x` with payload Y returns 201 and a row in `event_log`"* — not *"the endpoint works."*
- **Impact on existing code.** Which modules touched, which contracts changed. Catches *"ah, this breaks `IBaseHandler<TSelf>`"* before code is written.
- **Open questions in a dedicated section.** Forces decisions before generation. The agent will fabricate plausible answers if they're missing.
- **References to relevant ADRs and prior specs.** Show the lineage; help the agent understand the constraints.

### Template

```markdown
# [Feature name]

> Copy this file to `specs/YYYY-MM-feature-slug/spec.md` and fill in.

## Goal

[1-3 sentences. What problem, for whom, in which system.]

## In scope

- [Concrete, observable outcomes]
- ...

## Out of scope (deliberately, not now)

- [Things you'd consider but are explicitly excluding from this change]
- ...

## Acceptance criteria

- [ ] [Concrete, testable. e.g. "POST /api/x with payload Y returns 201 and a row in event_log"]
- [ ] ...

## Impact on existing code

- [Which files / modules will change]
- [Which contracts or conventions this touches]
- [Anything that could break — call it out by name]

## Open questions

- [ ] [Question that must be answered before implementation starts]
- [ ] ...

## References

- [Link to related ADRs, prior specs, or docs]
```

A copy-pasteable version lives at [`templates/spec.md`](../templates/spec.md).

### Spec status lifecycle

A spec moves through four states:

1. **Draft** — being written; open questions still listed.
2. **In implementation** — open questions answered (or moved to ADRs); `plan.md` and `tasks.md` written alongside; engineer is coding.
3. **Shipped** — PR merged. Append `STATUS: shipped (PR #N, YYYY-MM-DD)` at the top of `spec.md`.
4. **Frozen** — same as shipped, but emphasized: from this point the spec is *history*. Never edit retroactively. If the feature changes, write a new spec.

The most common rot pattern: specs sitting in *Draft* indefinitely. A spec older than ~2 weeks with unanswered Open Questions should either move forward or be deleted — stale drafts pollute the catalog and erode trust in the discipline.

### Practical tips

**Keep specs short.** A spec longer than ~150 lines is usually two specs in a trenchcoat. Split.

**Spec ships before code.** If you find yourself writing the spec *after* the PR to explain code that already exists, that's documentation, not a spec. File it under `docs/`, not `specs/`. (See [`legacy-to-sdd-migration-guide.md`](legacy-to-sdd-migration-guide.md) § "Phase 2 — Forward-only specs" — this is the migration discipline applied to every change.)

**Open Questions are valuable.** Don't be embarrassed by them. A spec with three Open Questions answered honestly beats a spec with zero questions and three silent assumptions.

**The PR description references the spec.** *"Implements: `specs/2026-05-x/`"*. Bidirectional link: PR → spec, spec → PR (via the shipped status). After a year, traceability is intact.

**Anti-pattern: spec as wishlist.** A spec is what *this PR* will do. If you find yourself listing 12 ideas, that's a roadmap entry — not a spec. Pick one and ship.

### Where to go deeper

The template above plus [`templates/spec.md`](../templates/spec.md) cover ~90% of features. For the per-role ownership of spec lifecycle (who drafts, who reviews, who approves, who updates status), see [`sdd-in-teams-guide.md`](sdd-in-teams-guide.md) § "Spec lifecycle." For how the agent reads specs and implements them task-by-task, see [`working-with-agents-guide.md`](working-with-agents-guide.md) § "After Creating a Spec — Starting Implementation."

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

## SDD in Teams: Roles and Responsibilities

The workflow above describes what happens for one engineer working on one feature. In a team — even a team of two — the same workflow becomes a multi-actor protocol. Different artifacts have different owners; different decisions need different sign-offs.

### Default ownership matrix

| Artifact | Primary owner | Reviewers | Updated by |
|----------|---------------|-----------|------------|
| `PRD.md` | Product / founder | Tech lead, founding team | (frozen after v1) |
| `CLAUDE.md` | Tech lead (or designated maintainer) | All engineers | Reactive — anyone proposes, owner approves |
| `ARCHITECTURE.md` | Tech lead / staff engineer | All engineers, security | When boundaries change |
| `DOMAIN.md` | Domain expert (often PM or senior eng) | Engineers writing in the domain | When new terms enter the codebase |
| `docs/adr/*.md` | The engineer proposing the decision | Tech lead + relevant team | (immutable from `Accepted`) |
| `specs/*/spec.md` | The engineer driving the change | PM (for scope), tech lead (for design) | Status updated after merge |
| `docs/runbooks/*` | On-call team | On-call rotation | Within 24h of any incident |
| `RUNBOOK.md` / `OPERATIONS.md` | SRE / DevOps | On-call engineers | Continuously |
| `TESTING.md` | Tech lead or QA lead | All engineers | When conventions change |

The matrix is a *default*, not a law. Small teams collapse rows (founder owns PRD + ARCHITECTURE + CLAUDE.md until the next hire). Large teams split rows (per-service `ARCHITECTURE.md`, per-team `CLAUDE.md` in monorepos).

### Three most common team failure modes

1. **Diffused ownership.** *"Everyone is supposed to update `CLAUDE.md`; nobody does."* Cure: one named maintainer per artifact, not *"the team"*. A name in the header.
2. **The documentation hero.** One person writes everything; the rest never internalize the discipline; when the hero leaves, the docs die within a quarter. Cure: rotate ownership of `CLAUDE.md` and the quarterly audit; require every PR to update the relevant artifact (spec status, ADR list, CLAUDE.md if a convention changed).
3. **Stale by Tuesday.** Docs updated Monday; code changes Tuesday; docs lie by Wednesday. Cure: updates ride *with* the PR that motivated the change, not as a follow-up. PR template includes a docs checkbox; review blocks if it's not ticked.

### The solo-developer case

A team of one still benefits — but the SDD discipline serves a different purpose. With no teammate to forget context, the artifacts protect *future you*. Six months from now you won't remember why you picked Dapper; the ADR is for your future self, not for an absent colleague.

Solo SDD collapses the matrix: you own everything; you also "review" your own work (the discipline replaces the second pair of eyes). Specs can be shorter; ADRs are usually still worth the 10 minutes.

### Going deeper

The full treatment — per-role profiles (Engineer / Tech Lead / EM / PM / Founder / QA / SRE / Security / External OSS contributors), artifact lifecycle handoffs, cadence patterns (PR-based vs meeting-based ADR review), multi-team and monorepo specifics, the onboarding flow for new hires, governance for OSS projects, regulated-industry overlays, and a longer list of team failure modes with corrections — lives in [`sdd-in-teams-guide.md`](sdd-in-teams-guide.md).

Read it when your team crosses ~3 engineers; before that, the simpler version above is usually enough.

---

## Architecture Decision Records (ADR)

ADRs were introduced by Michael Nygard in 2011. The premise: **code shows *what* you did, but not *why*. ADRs fill that gap.** Each ADR is a short, dated, immutable document capturing a single decision, its context, and the alternatives rejected.

### When to write an ADR

Write an ADR when a decision:

- **Is hard to reverse** (ORM choice, message broker, authorization approach)
- **Has consequences beyond a single module** (logging convention across the system)
- **Was made against the obvious choice** (*"why Dapper when the .NET standard is EF?"*)
- **Keeps resurfacing in discussions** (an ADR ends the re-litigation permanently)

### When NOT to write an ADR

- **Variable naming, formatting, code style** → `CLAUDE.md` or `.editorconfig`
- **Decisions inside a single feature** → `spec.md`
- **Things obvious from the stack** (*"we use async/await"*)

**Practical test:** if a year from now someone asks *"why the hell did we do it this way?"* — that's an ADR.

### Minimal structure (Nygard format)

```markdown
# ADR-007: Dapper as the Data Access Layer

## Status
Accepted — 2026-01-15

## Context
[The forces that made this decision necessary — 3 to 8 sentences.]

## Decision
[The choice itself — 3 to 5 sentences. Not a tutorial.]

## Consequences
**Positive:** [...]
**Negative:** [...]
**Neutral:** [...]

## Alternatives Rejected
- [Option A] — [one-sentence reason]
- [Option B] — [one-sentence reason]
```

For smaller decisions, **MADR** (Markdown ADR) trims to just Status / Context / Decision / Consequences.

### The Supersedes pattern (the rule that makes ADRs work)

You **never edit the body of an Accepted ADR.** When the decision changes, you write a new ADR with `Supersedes: ADR-NNN` in its Status, and you update *only the status header* of the original to `Superseded by ADR-NNN — YYYY-MM-DD`. The body of the old ADR stays intact as history.

This is what makes ADRs valuable a year later: someone reading the repo can trace the decision lineage by following Supersedes links. Editing old ADRs to "keep them current" destroys the artifact.

### Going deeper

The full how-to — Nygard format section-by-section, alternative formats (MADR, Y-statements), four worked examples (Accepted, Superseded pair before/after, Proposed under discussion, Deprecated), the lifecycle in depth, numbering conventions, cross-referencing patterns, review process, agent-assisted drafting prompts, anti-patterns with corrections, edge cases (sub-ADRs, retroactive, monorepos), tooling, maintenance discipline, golden rules — lives in [`adr-guide.md`](adr-guide.md).

Read it before writing your first few ADRs; the difference between an ADR that survives a year and one that rots in three months is mostly about the rules in that guide.

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
├── .claude/                           # Claude Code config — shipped with the repo, used by every contributor
│   ├── settings.json                  # hooks, permissions, env vars (see Claude Code Building Blocks)
│   ├── commands/                      # slash commands — recurring prompts as files
│   │   ├── spec-new.md                # /spec-new <slug> — drafts a new spec from the template
│   │   ├── audit-docs.md              # /audit-docs — reports staleness across guides + runbooks
│   │   └── end-session.md             # /end-session — writes a session-notes file
│   └── skills/                        # multi-step procedures Claude can auto-invoke
│       └── adr-draft/                 # one folder per skill
│           └── SKILL.md               # frontmatter + procedure
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
- **Token economy** — where tokens are wasted and how prompt caching changes session shape
- **Claude Code building blocks** — skills, slash commands, subagents, hooks: which to reach for and how each fits an SDD repo
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
