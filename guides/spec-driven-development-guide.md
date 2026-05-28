# Spec-Driven Development with AI Coding Agents

> **Built for the 10-person, billion-dollar company.** A small team wielding AI coding agents can now ship what used to take hundreds of engineers — but raw speed isn't the moat. The moat is keeping the agents aligned with your intent as the codebase grows. Spec-driven development is that discipline; these guides are the playbook.

> A practical guide to structuring your repository so AI coding agents (Claude Code, Copilot, Cursor) produce consistent, maintainable code instead of drifting away from your intent.

---

## Table of Contents

1. [Core Philosophy](#core-philosophy)
2. [Who Practices SDD](#who-practices-sdd)
3. [When SDD Pays Off (and When It's Overhead)](#when-sdd-pays-off-and-when-its-overhead)
4. [The Absolute Minimum](#the-absolute-minimum)
5. [Before the PRD: Research and Discovery](#before-the-prd-research-and-discovery)
6. [The PRD Layer](#the-prd-layer)
7. [Writing a Good spec.md](#writing-a-good-specmd)
8. [Writing a Good PLAN.md](#writing-a-good-planmd)
9. [Writing a Good tasks.md](#writing-a-good-tasksmd)
10. [The Three-Layer Documentation Model](#the-three-layer-documentation-model)
11. [Workflow for Changes](#workflow-for-changes)
12. [SDD in Teams: Roles and Responsibilities](#sdd-in-teams-roles-and-responsibilities)
13. [Architecture Decision Records (ADR)](#architecture-decision-records-adr)
14. [Migrating from a Legacy Repo to SDD](#migrating-from-a-legacy-repo-to-sdd)
15. [Additional Files Worth Adding](#additional-files-worth-adding)
16. [Runbook vs Postmortem](#runbook-vs-postmortem)
17. [Repository Organization](#repository-organization)
18. [Working with the Agent](#working-with-the-agent)
19. [Enforcing and Evaluating SDD](#enforcing-and-evaluating-sdd)
20. [Golden Rules](#golden-rules)

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

## Before the PRD: Research and Discovery

Before you can write a PRD, someone has to figure out *what to build, for whom, why*. That's the **research** phase (also called *discovery* in product/UX vocabulary — same thing). It includes customer interviews and synthesis, competitive analysis, market sizing, problem validation, and opportunity briefs.

The PRD is the *output* of research. Research is what feeds it.

### The principle: humans + agent context, never code source

Research artifacts follow the same rule as PRDs, one layer up: **research is for humans plus the agent's context — the agent never generates code from research**.

Three ways the agent legitimately uses research:

1. **As context when drafting PRDs.** *"Per `docs/research/2025-Q3-interviews.md`, controllers spend 8–12 hours/month on reconciliation."* The PRD's problem statement gets grounded in cited findings.
2. **As context when interpreting *"for whom"* during implementation.** Helps name variables, write UI copy, match user vocabulary, sanity-check assumptions.
3. **As a synthesis assistant.** Given anonymized notes already in the repo, the agent can extract themes, identify patterns, or flag contradictions across interviews.

The agent **never** turns research into a feature without a PRD + specs in between. Research is upstream of strategy, not a substitute for it.

### What goes in `docs/research/`

**Yes:**
- Synthesized interview themes (anonymized; quotes attributed to roles — *"Controller at mid-market SaaS company"* — not names)
- Competitive analyses with named real competitors and sourced claims
- Market sizing with assumptions documented
- Problem validation studies and the conclusions drawn
- Opportunity briefs (PRD candidates that haven't been ratified)
- Postmortems from failed pilots or canceled directions

**No:**
- Raw interview transcripts with PII
- Customer names, contracts, NDA-protected materials
- Salary or revenue data identifiable to specific companies
- Anything you'd be uncomfortable seeing in a leak

### The PII gate

**Anonymization happens before commit, not after.** The discipline: synthesized artifact ≠ raw source. Raw sources stay in your research-ops tool (Dovetail, Grain, Notion, etc.); only synthesized artifacts enter the repo. *"When in doubt, leave it out."*

For the practical mechanics — full folder structure, the artifact-type breakdown, the synthesis discipline (raw → patterns → themes), AI-assisted synthesis prompts, the PRD↔research interface, and research-specific anti-patterns (raw transcripts in repo, opinion-as-research, advocacy disguised as synthesis, research that quietly becomes a PRD by accident) — see [`research-guide.md`](research-guide.md).

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

**Critical insight:** The PRD is a *starting artifact*, not a living document. Once an era's initial version ships, that era's PRD freezes as history. Subsequent feature-level changes live in specs, not in PRD edits. Major direction shifts get a *new* PRD (see [Multiple PRDs over time](#multiple-prds-over-time-the-era-pattern) below), never edits to the old one.

Treating PRD as living causes two problems:
- It becomes a dumping ground for every change request
- The original intent gets diluted over time

Keep each PRD pristine. Archive in `docs/prd/` with a date. New direction → new PRD; *never* edits to an old one.

For the practical side — format alternatives (PR-FAQ, lean PRD, one-pager, full template), two complete worked examples, era boundary heuristics, AI-authoring prompts, stakeholder review process, and PRD-specific anti-patterns — see [`prd-guide.md`](prd-guide.md).

### PRD vs spec — two layers, same question

PRDs and feature specs look similar at a glance — both have "scope" sections, both describe "what we're building." New teams ask: *do we really need both?*

Yes. They live at different layers and have different lifetimes.

| | PRD | Spec |
|---|---|---|
| **Scope** | The whole product / an era of the system | A single feature or change |
| **Lifetime** | Written per era, frozen after that era ships | Written before the PR, frozen after merge |
| **Granularity** | *"Build an order delivery platform"* | *"Add ZIP-per-batch compression to `BatchXmlMerger`"* |
| **Audience** | **Humans only** — stakeholders, founders, product strategists | Developers **and the AI agent** about to write code |
| **Read by the agent?** | **No** — never used as a code-generation source | **Yes** — first artifact the agent reads to implement |
| **Frequency** | Once per era (so: rarely; 1–4 over a product's life) | Dozens or hundreds per year |
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

### The agent doesn't read PRD

A consequence worth stating explicitly: **the AI agent never generates code from a PRD.** The agent reads `spec.md → plan.md → tasks.md`. The PRD lives one layer up — it answers product-level questions (*what are we building, for whom, why*) that the implementation layer takes as given.

Practical implications:

- **Don't feed PRD to the agent and expect a working feature.** It hasn't got the granularity. The agent will fabricate plausible implementation details that may or may not match what you actually want.
- **Don't write PRD with implementation detail *"for the agent."*** If you find yourself adding *"use Dapper for data access"* or *"implement Y as middleware"* to a PRD, that's a sign the content belongs in `plan.md`, an ADR, or `CLAUDE.md` — not the PRD.
- **PRD shapes the spec — it doesn't replace it.** When an engineer writes a new spec, they may reference the PRD for context (*"per the original PRD, partner SLA requires < 1% failure rate"*), but the spec itself is what the agent then consumes.

The PRD audience is **humans**: stakeholders, founders, future product strategists. The agent's audience starts at `spec.md`.

### The trap each avoids

**Specs without a PRD:** six months in, nobody remembers why retries even matter — the partner SLA requires < 1% failure rate, but that requirement lives only in someone's head. New engineers (and the AI agent) propose "simpler" solutions that miss the point.

**PRD without specs:** the PRD becomes a Frankenstein document where every change request gets appended. Original v1 intent dilutes, decision history disappears, and after two years the PRD reads like a press release for a product that doesn't exist.

Two artifacts, two lifetimes, two purposes. The overlap in fields is real but cosmetic — the level of detail is completely different.

### Multiple PRDs over time (the era pattern)

A 5-year product won't have the same PRD it had on day 1 — and that's fine. The rule *"PRD freezes after v1"* doesn't mean *"one PRD per project lifetime"*; it means *"one PRD per **era**."*

An **era** is a major strategic chapter — a moment where the product's *what* and *why* change at the top level. Examples:

- Original launch (your first PRD)
- A pivot in market or audience (B2C → B2B, or vice versa)
- A major capability set added (dashboard → full platform, single-tenant → multi-tenant)
- Geographic expansion that changes core assumptions
- Post-acquisition direction shift

Each era gets its own PRD. Each PRD freezes after that era's initial release. The collection in `docs/prd/` becomes the company's strategic timeline at a glance:

```
docs/prd/
├── 2025-12-original.md              # original launch
├── 2027-Q2-mobile-app.md            # adding mobile as a first-class product
├── 2029-Q1-platform-shift.md        # B2B → B2C2B pivot
└── 2030-Q3-european-expansion.md    # geographic expansion
```

**This is versioning at the directory level**, not at the document level:

- **Each PRD is its own complete document** — not a delta from the previous one.
- **Each PRD freezes after its era ships** — body never edited again.
- **The history lives in the directory listing**, not inside any single file.

The pattern preserves the *"freeze after v1"* discipline (no Frankenstein document) while honestly reflecting that long-lived products have multiple strategic chapters.

**When to write a new PRD** (era boundary):

- New market or audience (e.g., adding enterprise customers to a B2C product)
- Major capability set (e.g., dashboard → full platform; data tool → multi-tenant SaaS)
- Strategic pivot (e.g., self-service → high-touch sales motion)
- After a major reorganization that changes how the product is built

**When NOT to write a new PRD** (these are specs or roadmap entries):

- A new feature within the current era's vision → spec
- A refactor of existing functionality → spec (+ ADR if architectural)
- A bugfix or polish → spec, or just a PR
- A "what we're planning next quarter" update → `ROADMAP.md`, not a new PRD

**Sanity check for smaller projects:** the era pattern says *eras exist when strategic chapters change*, not *eras exist on a calendar*. Most projects have **one PRD that lasts years** — that's the normal case. The era pattern matters when a major shift actually happens, which for many products is never.

---

## Writing a Good spec.md

A `spec.md` answers: *what are we building, why, and how do we know it works?* It's the **first document the agent reads when implementing** (unlike PRD, which is for humans only — see [The agent doesn't read PRD](#the-agent-doesnt-read-prd)); the document the human PR reviewer reads before approving; the document future-you reads when wondering *"why is this code shaped this way?"*

A spec is **per-feature** and **frozen after merge** — different from a PRD (whole-product per era, frozen after that era's release) and from a plan (the *how*, written alongside the spec — see [the next section](#writing-a-good-planmd)).

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

The template above plus [`templates/spec.md`](../templates/spec.md) cover ~90% of features. For a full worked example of `spec.md` + `plan.md` + `tasks.md` filled in for one feature, plus AI-authoring prompts (*drafting a spec from an idea*, *reviewing for gaps*, *the trio consistency check*), see [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md). For the per-role ownership of spec lifecycle (who drafts, who reviews, who approves, who updates status), see [`sdd-in-teams-guide.md`](sdd-in-teams-guide.md) § "Spec lifecycle." For how the agent reads specs and implements them task-by-task, see [`working-with-agents-guide.md`](working-with-agents-guide.md) § "After Creating a Spec — Starting Implementation."

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

For a full worked example of plan.md alongside its sibling spec.md and tasks.md (with cross-document references, AI-authoring prompts for drafting `plan.md` from `spec.md`, and the validation prompt that checks the plan against existing ADRs and `ARCHITECTURE.md`), see [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md).

---

## Writing a Good tasks.md

A `tasks.md` answers: *what's the execution order, and how do we know each step worked?* It's the checklist the engineer (and the agent) walks through to turn `spec.md` + `plan.md` into shipped code.

If `spec.md` is *what + why*, and `plan.md` is *how*, then `tasks.md` is *do, in this order, with verification at each step.*

### When you need a tasks.md

Most non-trivial features benefit from one. Skip it when:

- The change is one or two commits (`tasks.md` becomes overhead for trivial work)
- The spec is small enough that the order is obvious
- The work is genuinely exploratory and the order will change as you learn

For everything else — write it. The act of writing tasks down forces you to think about order, verification, and dependencies *before* you start typing code.

### Principles that work

- **One task = one verification.** Each task has a way to confirm it's done before moving on. *"Add validation"* → bad. *"Write tests for invalid inputs (red), then implement validation (green)"* → good.
- **Order matters.** Tasks are sequential by default. If two are truly parallel, mark them — but the default assumption is that task N depends on N-1.
- **Test-first when applicable.** Red-green-refactor is the most common discipline; `tasks.md` often opens with the failing test.
- **Reference acceptance criteria.** Each AC from `spec.md` should map to one or more tasks. If an AC has no matching task, you're missing work; if a task doesn't trace to an AC, it's scope creep.
- **Atomic enough to checkpoint.** A task should be small enough that you (or the agent) can finish it without losing context, then pause for review.
- **Updated as work progresses.** Checkboxes flip; notes appended. After merge, the file freezes alongside `spec.md` as historical record.

### Template

```markdown
# Tasks: [Feature name]

> Order matters. Each task has its own verification. Check off as you go.

## Setup

- [ ] [Any prerequisites — branch, DB migration, env setup]

## Implementation (in execution order)

- [ ] 1. [Task description] → verify: [how to confirm]
- [ ] 2. [Task description] → verify: [how to confirm]
- [ ] 3. ...

## Verification (against acceptance criteria)

- [ ] AC1: [from spec.md] → [test / manual check]
- [ ] AC2: [from spec.md] → [test / manual check]

## Post-merge

- [ ] Append `STATUS: shipped (PR #N, YYYY-MM-DD)` to spec.md
- [ ] Update relevant docs (CLAUDE.md, ARCHITECTURE.md, etc.) if needed
- [ ] Close related issues / tickets

## Notes (append as you work)

- [YYYY-MM-DD]: [observation, blocker, decision made on the fly]
```

A copy-pasteable version lives at [`templates/tasks.md`](../templates/tasks.md).

### Granularity — how big is a task?

Roughly: **one task should be reviewable in 5–15 minutes** of focused attention. Smaller is fine; bigger is a smell.

| Size | Example | Verdict |
|------|---------|---------|
| Too small | *"Add a return statement"* | Combine with surrounding work |
| Right | *"Implement `OrderValidator.Validate`, with tests for 3 invalid-input cases"* | Yes |
| Right | *"Wire `POST /api/orders` (controller → handler → repository) with 1 happy-path integration test"* | Yes |
| Too big | *"Implement the entire order processing pipeline"* | Split into 3–5 tasks |

A task that takes more than ~2 hours of focused work is usually two tasks.

### How the agent uses tasks.md

When the agent implements a spec, the standard prompt is *"work through tasks.md one task at a time, with checkpoints."* The agent then:

1. Reads the next unchecked task
2. Restates what it's about to do (a brief plan)
3. Implements the task
4. Runs the verification step
5. Waits for your confirmation
6. Moves to the next task

This is why **the verification step matters** — without it, the agent can't tell when it's done. With it, the agent is self-checkpointing.

See [`working-with-agents-guide.md`](working-with-agents-guide.md) § "After Creating a Spec — Starting Implementation" for the full prompt pattern.

### Practical tips

**Update as you work.** Check off completed tasks; append notes when something unexpected happens. The notes are the most underrated part of `tasks.md` — they tell future you (or the post-mortem) what really happened.

**Freezes alongside spec.md after merge.** All checkboxes should be ticked; notes should be readable. Don't tidy up retroactively — leave the messy reality.

**Pause for review at task boundaries, not mid-task.** If the agent is mid-task and you want to interject, finish the current task first. Mid-task interruptions produce broken artifacts.

**Skip for trivial changes.** A 1-task `tasks.md` is just `spec.md` with extra ceremony. If you genuinely have only one thing to do, mention it in `spec.md` and skip the file.

### Anti-patterns

1. **Tasks without verification.** *"- [ ] Implement validation"* gives the agent no way to know when it's done. Always add `→ verify:`.
2. **Tasks describing the result, not the action.** *"- [ ] Order processing works"* is a vibe, not a task. *"- [ ] Implement `OrderProcessor` with 3 cases (happy + 2 failures)"* is a task.
3. **`tasks.md` as TODO list for the whole feature lifecycle.** *"- [ ] Decide on logging library"* belongs in `plan.md` or an ADR. `tasks.md` is for *implementation* steps, not pending decisions.
4. **Rewriting `tasks.md` mid-flight.** If the plan changes, update `plan.md` and add new tasks at the end. Don't retcon the old ones — the history is valuable.
5. **No checkboxes.** Plain text without `- [ ]` defeats half the point. The checkboxes give a visible progress signal at any moment.

### Where to go deeper

For a full worked example of `tasks.md` alongside its `spec.md` + `plan.md` siblings, the AI prompt for drafting tasks from a plan, and the `/trio-check` consistency audit before implementation, see [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md). For per-role ownership (who writes `tasks.md`, who reviews, who flips checkboxes), see [`sdd-in-teams-guide.md`](sdd-in-teams-guide.md) § "Spec lifecycle". For the agent-side implementation flow with concrete prompts, see [`working-with-agents-guide.md`](working-with-agents-guide.md) § "After Creating a Spec — Starting Implementation".

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

**`docs/research/`** — synthesized research artifacts upstream of PRDs: interview themes, competitive analyses, market sizing, problem validation, opportunity briefs. Lives here as **agent context** (the agent uses it when drafting PRDs and interpreting user vocabulary), never as a code-generation source. **Raw data with PII stays out — synthesis only.** See [Before the PRD: Research and Discovery](#before-the-prd-research-and-discovery) above and [`research-guide.md`](research-guide.md) for the full discipline.

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

**`docs/external-resources.md`** — links to partner documentation, articles that inspired decisions, talks. Source attribution = easier verification in the future.

---

## Runbook vs Postmortem

Both `RUNBOOK.md` / `docs/runbooks/` and `docs/postmortems/` were listed above. They cover the same surface area — *incidents* — but operate on different timescales and serve different purposes. New teams routinely confuse them (or pick one assuming it covers what the other does). It's the same kind of confusion as PRD-vs-spec from [The PRD Layer](#the-prd-layer): same domain, different lifetimes.

|  | Runbook | Postmortem |
|---|---|---|
| **When read** | During an incident (3 a.m., half awake) | Days or weeks after the incident |
| **Time frame** | *"What do I do right now?"* | *"What happened, why, and what changes?"* |
| **Audience** | On-call engineer under stress | Engineering team + management, in a calm review |
| **Format** | Symptoms → diagnosis → recovery → verification | Summary → timeline → root cause → lessons → action items |
| **Updated when** | Within 24 h of every incident (plus quarterly audit) | Once, after the incident — then frozen |
| **Lifetime** | Living (continuously updated) | Frozen artifact, archived |
| **Lives in** | `docs/runbooks/<symptom-slug>.md` | `docs/postmortems/YYYY-MM-DD-<incident-slug>.md` |
| **Goal** | Mean-time-to-recovery ↓ | Mean-time-between-incidents ↓ |

### They aren't alternatives — they're a sequence

A postmortem usually *produces* a new runbook entry (or updates an existing one). The sequence per incident:

```
Incident → resolved
   ↓
24 h:    Runbook entry written/updated   (so next on-call doesn't relearn)
   ↓
1–7 d:   Postmortem written              (root cause + lessons + follow-ups)
   ↓
The postmortem may flag "runbook needs further updating" — feedback loop closes
```

### The trap each avoids

**Runbook without postmortems:** recovery procedures stay shallow because nobody analyzes *why* incidents keep recurring. The same root cause produces three more incidents over six months; each one gets a runbook update, but the underlying problem is never named.

**Postmortems without runbook:** analyses are excellent, lessons are real — but at 3 a.m. when the next incident hits, the on-call still has to figure out the recovery from scratch. A six-month-old postmortem doesn't help under stress.

### A minimal postmortem template

A copy-pasteable postmortem template lives at [`templates/postmortem.md`](../templates/postmortem.md). The headings:

```
# Postmortem: [one-line incident summary]

Date / Severity / Duration / Author / Status

## Summary
## Timeline
## Root cause
## What went well
## What went wrong
## Lessons learned
## Action items (each with owner + due date)
```

**Blameless framing matters.** A postmortem accuses systems and decisions, not individuals. Otherwise the next one isn't written honestly — and the whole discipline collapses within a quarter.

### Where to go deeper

For the runbook side — what makes an entry pass the 3 a.m. test, full templates, anti-patterns, agent prompts for drafting entries after incidents, and how runbook updates trigger off postmortems — see [`runbook-operations-guide.md`](runbook-operations-guide.md). For postmortem discipline at scale (blameless practices, action-item tracking, cultural framing), most teams adopt one of the public templates (Google SRE workbook, Atlassian's incident handbook, Increment magazine articles) and adapt to local practice.

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
│   ├── prd/                           # PRDs per era — humans only, agent never reads these
│   │   ├── 2025-12-original-prd.md    # the launch PRD (frozen after v1)
│   │   ├── 2027-Q2-mobile-app.md      # later era: adding mobile (frozen after that release)
│   │   └── 2029-Q1-platform-shift.md  # later era: B2B → B2C2B pivot
│   ├── runbooks/                      # per-incident recovery procedures (the 3 a.m. file)
│   ├── integrations/                  # one file per external integration — endpoints, quirks, contacts
│   ├── data-flows/                    # Mermaid diagrams of main flows through the system
│   ├── schemas/                       # DDL of current schema + ERDs
│   ├── postmortems/                   # incident analyses — root cause, lessons, follow-ups
│   ├── research/                      # synthesized research — agent context, never code source
│   │   ├── README.md                  # index — what's here, currency, used-for
│   │   ├── interviews/                # anonymized interview themes (raw stays in research-ops tool)
│   │   ├── competitive/               # competitive analyses with named real competitors
│   │   ├── sizing/                    # TAM / SAM / SOM with assumptions documented
│   │   ├── validation/                # problem validation studies, failed-pilot postmortems
│   │   └── opportunity-briefs/        # PRD candidates not yet ratified
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
- **Frozen layer** — `docs/prd/`, `specs/<date-slug>/`. Written once, frozen after their moment passes (each PRD: after its era ships; each spec: after the PR merges). Treat as historical artifacts.

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

## Enforcing and Evaluating SDD

A practical question that surfaces once a team is several months into SDD: *can we make the discipline stick without depending on every contributor remembering every rule?*

Partially yes. SDD enforcement operates on **three layers**, each with different reach:

| Layer | What it does | Example |
|-------|-------------|---------|
| **Mechanical** (regex, hooks, CI) | Hard gate: blocks specific actions | Block edits to bodies of ADRs with `Status: Accepted` |
| **LLM evaluator** (configured subagent, slash command) | Soft gate: scores quality, surfaces issues | *"Are these acceptance criteria testable?"* |
| **Human review** (code review, architecture meeting) | Judgment: ensures quality, completeness, intent | *"Is this Out-of-scope list complete enough to prevent drift?"* |

The mistake most teams make is trying to mechanize judgment items (like "is this AC testable") or leave mechanical items (like "is the ADR body unchanged") to vigilance. Neither works at scale.

**The asymmetric-cost rule of thumb:**

- *False positive is cheap, false negative is catastrophic* → mechanical hard gate (PII scan, ADR body lock)
- *False positive is expensive (blocks legit work), false negative is recoverable* → LLM evaluator that warns, not blocks
- *Both depend on context* → human review

A working SDD setup typically has: pre-commit hooks for PII and Accepted ADR protection, Claude Code hooks for in-session feedback (active ADR list, end-of-session reminders), a configured subagent (`trio-auditor`) for cross-artifact checks, slash commands (`/spec-check`, `/trio-check`) for user-invoked evaluation, and CI checks that block on mechanical issues but advise on LLM evaluations.

For the full treatment — five implementation patterns (pre-commit hooks, Claude Code hooks, configured subagents, slash commands, CI/CD), what belongs in each category for each SDD artifact, a complete worked example of a setup, and the anti-patterns (over-strict hooks, no escape hatch, mechanical checks of subjective things, alert fatigue) — see [`quality-gates-guide.md`](quality-gates-guide.md).

---

## Golden Rules

1. **Don't create documents on spec.** Add the next one when:
   - You explain the same thing to the agent for the third time (→ this should be a document)
   - You return to old code and don't remember why (→ ADR or journal)
   - You onboard someone and notice something obvious is missing (→ ONBOARDING or CONFIG)
   - The agent generates something against an unwritten convention (→ CLAUDE.md or TESTING.md)

2. **A document without an owner is dead.** Every file you add must answer: *"who reviews this, and when?"*

3. **A repo with 30 stale markdown files is worse than a repo with 5 always-fresh ones.** Discipline > completeness.

4. **Research and PRD are both for humans + agent context, never code source.** The agent reads `spec.md → plan.md → tasks.md`. Research lives in `docs/research/` (anonymized — synthesis only, no raw PII); PRDs live in `docs/prd/` (frozen per era; multiple over time = fine, editing old ones = Frankenstein).

5. **ADRs are immutable. Specs freeze after shipping. Only the stable layer evolves continuously.**

6. **Context per task, not context per project.** Don't dump everything on the agent. Pick what's relevant.

7. **A short ADR written today beats a perfect one written never.**

8. **The plan is alive, but decision history must be visible.** Move open questions to decisions; never delete them.

9. **Code shows what. Documentation shows why.** If the why isn't written down, it doesn't exist.

10. **Spec-driven development is a discipline, not a methodology.** Apply it daily, not as a one-time event.

---

*This guide reflects practical patterns from real-world .NET projects using Claude Code, Copilot, and similar AI coding agents. Adapt to your stack and team size.*
