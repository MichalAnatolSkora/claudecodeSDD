# Spec-Driven Development with AI Coding Agents

> **Built for teams of 1–10.** A practical guide to structuring your repository so AI coding agents (Claude Code, Copilot, Cursor) produce consistent, maintainable code instead of drifting away from your intent. Raw speed isn't the moat — keeping the agents aligned with your intent as the codebase grows is.

**This guide is the map: each section says what a piece is for and points to the guide that covers it.** New? Start with the one-page [SDD in 5 files](sdd-in-5-files.md). If you read only one deep guide, make it [The Feature Trio: spec → plan → tasks](spec-plan-tasks-guide.md) — the core loop everything below supports.

---

## Table of Contents

1. [Core Philosophy](#core-philosophy)
2. [When SDD Pays Off (and When It's Overhead)](#when-sdd-pays-off-and-when-its-overhead)
3. [The whole flow](#the-whole-flow)
4. [The Absolute Minimum](#the-absolute-minimum)
5. [The PRD Layer](#the-prd-layer)
6. [The trio: SPEC.md, PLAN.md, TASKS.md](#the-trio-specmd-planmd-tasksmd)
7. [The Three-Layer Documentation Model](#the-three-layer-documentation-model)
8. [Workflow for Changes](#workflow-for-changes)
9. [SDD in Teams](#sdd-in-teams)
10. [Architecture Decision Records (ADR)](#architecture-decision-records-adr)
11. [Migrating from a Legacy Repo to SDD](#migrating-from-a-legacy-repo-to-sdd)
12. [Runbook vs Postmortem](#runbook-vs-postmortem)
13. [Repository Organization](#repository-organization)
14. [Working with the Agent](#working-with-the-agent)
15. [Enforcing and Evaluating SDD](#enforcing-and-evaluating-sdd)
16. [Golden Rules](#golden-rules)

---

## Core Philosophy

**Spec-Driven Development (SDD)** means writing requirements, decisions, and constraints down *before* asking an AI agent to generate code. Every minute spent on specification saves an hour of refactoring agent-generated drift.

The core insight: **AI agents fill gaps with assumptions.** The more they assume, the further the code drifts from what you actually want. Good specs eliminate the gaps.

Related terms you'll encounter:
- **Context engineering** — preparing complete context before generation
- **PRD-first approach** — starting from a Product Requirements Document
- **Spec-kit workflow** — GitHub's formalization: `SPEC.md` → `PLAN.md` → `TASKS.md`

These are facets of the same idea. The name matters less than the discipline — and none of it is new: GitHub ships it as `spec-kit`, Amazon has written PR-FAQs before code for decades, Google runs on design docs. Who uses what, and what they call it: [`who-uses-sdd.md`](who-uses-sdd.md).

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

In these cases, writing a `CLAUDE.md` is overhead. Just prompt — don't perform documentation theatre on code that's about to be deleted.

### Not sure yet?

For a solo project that *might* grow, or one in the 1–3 month window where it's not clear it will survive, start with the five files in [`sdd-in-5-files.md`](sdd-in-5-files.md) — about two hours upfront — and skip everything else until a specific trigger fires ([The Absolute Minimum](#the-absolute-minimum)). That captures most of the drift-prevention benefit without committing you to the full discipline; if the project survives three months, the foundation is already in place.

### The hidden cost of NOT doing SDD

People often weigh *"the cost of writing docs"* against *"the speed of just prompting."* This misses the cost of NOT having docs:

- Time spent **re-explaining the same convention** every session
- Time spent **reviewing or refactoring** agent output that drifted from the rest of the code
- Decisions **re-litigated in PR review** (*"didn't we already discuss this?"*)
- **New contributors** (human or agent) starting from zero context
- **Old code that nobody understands** because the rationale lives in someone's head
- **Production incidents** where nobody knows the original constraints

The trade is: pay X% upfront in docs, save Y% over time in drift, rework, and onboarding friction. The X:Y ratio improves the longer the project lives and the more people touch it.

---

## The whole flow

Everything in this guide fits one pipeline — and for a team of **1–10**, you almost never run all of it: **you enter where your change starts and skip the rest.**

```
PRD  →  slice into  →  ┌ spec → plan → tasks ┐ → implement + test → ADR? → merge
(per    features         └──── the trio ──────┘   (agent, red→green)  (record  (freeze
 era)   (/features-       one per feature                             keepers)  the spec)
                       from-prd)
```

**Where you enter** — most small-team work enters late; you rarely run the whole pipeline. The entry table (*what you have* → *where to start* → *what to skip*) lives once, in the spine [`flow-guide.md`](flow-guide.md), so the two copies can't drift.

At 2–3 people you'll usually skip formal **research** and a heavy **PRD** (a one-pager, or a paragraph in an issue, is plenty) and enter at *slice* or straight at the *trio*. Add an upstream stage only when **more than one person has to agree**, or the bet is big enough that building the wrong thing is expensive.

**The stages, with their detail guides:**

*(Upstream and optional, not really an SDD phase: **research** — informal user/market discovery that grounds the PRD. Most 1–10 teams skip it; see [§ Before the PRD: Research and Discovery](#before-the-prd-research-and-discovery). It stays human + agent context, never a code source.)*

1. **PRD** — [`prd-guide.md`](prd-guide.md): the product-level *what & why*, per era.
2. **Slice → features** — [`prd-guide.md`](prd-guide.md) § "Slicing the PRD into features": vertical slices, walking skeleton first.
3. **The trio** — [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md): the core loop; one `spec → plan → tasks` per feature.
4. **Implement + test** — [`working-with-agents-guide.md`](working-with-agents-guide.md) (the implementation loop) and [`testing-guide.md`](testing-guide.md) (how the agent writes the tests): the agent works `TASKS.md` red→green; the tests come straight from the spec's acceptance criteria.
5. **Decisions** — [`adr-guide.md`](adr-guide.md): record the ones worth not relitigating.

Under all of it sits the **stable layer** — `CLAUDE.md`, `ARCHITECTURE.md`, `DOMAIN.md` — the docs the agent reads every session. That's where you actually start on day one ([The Absolute Minimum](#the-absolute-minimum), next), adding the rest reactively. When `ARCHITECTURE.md` earns its place, `/sdd-2-architecture-from-prd` (greenfield) or `/sdd-2-architecture-from-code` (existing codebase) bootstraps it — structure plus a stub ADR per hard-to-reverse decision.

**For the full step-by-step** — every command and what it produces, with a cheat sheet — see [`flow-guide.md`](flow-guide.md).

---

## The Absolute Minimum

If you're starting today and don't want to overinvest in docs upfront, what's the smallest set of markdown files that still buys you SDD's benefits?

### The single most important file

**`CLAUDE.md`** at the repo root (or `.cursor/rules/` for Cursor, equivalent files for other agent tools). If you do nothing else from this guide, do this. It's the only file most agents load automatically; without it your conventions don't exist as far as the agent is concerned. One file the agent always reads beats ten files it might never reach.

What separates a useful `CLAUDE.md` from a generic one — what to put in, what to leave out, how big it should get, how it changes when the repo has 30+ other markdown files competing for attention — is the subject of its own detail guide: [`claude-md-guide.md`](claude-md-guide.md). Read that one before writing your project's `CLAUDE.md` from scratch; the difference between a "wall of text" and a hub that routes the agent's attention well is mostly about the rules in that guide.

### Five files and one loop

The starting set — `CLAUDE.md`, `README.md`, `ARCHITECTURE.md`, `DOMAIN.md` (skip it if your domain is generic CRUD), and one `SPEC.md` per change — and the five-step loop every change runs through (write the spec → red→green → break the code → merge & freeze → one `CLAUDE.md` line per correction) live on one page: [`sdd-in-5-files.md`](sdd-in-5-files.md). Three files — `CLAUDE.md`, `README.md`, one spec — is the floor; below that you're hoping, not specifying.

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
| `openapi.yaml` / `API.md` | First time the agent invents an endpoint convention your API doesn't use |
| `docs/schemas/` (current DDL) | First time the agent invents a column name |
| `docs/postmortems/<date>.md` | First incident worth not repeating |

A `RUNBOOK.md` written before you have a system in production is fiction; a `CHANGELOG.md` written before you have users is busywork. Wait for the trigger; the act of writing the doc *while the problem is fresh* is what makes it useful.

### A rule of thumb for any candidate file

If you can't say *who reads this file, and when*, you don't need it yet.

For each candidate doc, answer:

- **Who reads it?** You? New hires? The agent? Auditors?
- **When?** Daily? On incident? Before a change? Once at onboarding?
- **What goes stale if you don't update it?**

A file that fails all three is decoration — skip it.

For the related discipline, see the first entry in [Golden Rules](#golden-rules) (*"Don't create documents on spec"*).

---

## The PRD Layer

A **PRD (Product Requirements Document)** captures *what* you're building and *why* — the problem, who has it, what success looks like, what's out of scope, the constraints — before any code exists. It's the product-level layer above the specs; at 2–3 people it's often a one-pager or a paragraph in an issue, and solo you can usually skip it.

### Before the PRD: Research and Discovery

Before a PRD, someone figures out *what to build, for whom, why* — customer conversations, a look at competitors, a sanity check on the size of the problem. Keep it lean: **most 1–10 teams never create `docs/research/` at all** — a PRD that says *"based on ~6 customer conversations"* is honest and sufficient. If you do keep research artifacts: **synthesis, not raw data; PII out** (anonymize before commit — raw sources stay in your research tool); and **findings flow through a PRD, never straight into code.**

### PRD vs spec — two layers, same question

They look alike — both have scope sections — but they live at different layers, with different lifetimes:

| | PRD | Spec |
|---|---|---|
| **Scope** | The whole product / an era of the system | A single feature or change |
| **Lifetime** | Written per era, frozen after that era ships | Written before the PR, frozen after merge |
| **Granularity** | *"Build an order delivery platform"* | *"Add ZIP-per-batch compression to `BatchXmlMerger`"* |
| **Audience** | Humans — stakeholders, founders, product strategists | Developers **and the AI agent** about to write code |
| **Agent implements from it?** | **No** — it reads the PRD only to draft, review, or slice it | **Yes** — first artifact the agent reads to implement |
| **Frequency** | Once per era (so: rarely; 1–4 over a product's life) | Dozens or hundreds per year |
| **Owner** | Product / founder | Engineer driving the change |

The PRD answers *why this system exists*; specs answer *what changed this week*.

**The agent doesn't implement from the PRD.** It reads the PRD to draft, review, or slice it — never as a code-generation source; it implements from `SPEC.md → PLAN.md → TASKS.md`. So don't write implementation detail into a PRD *"for the agent"*: *"use Dapper"* or *"implement Y as middleware"* belongs in `PLAN.md`, an ADR, or `CLAUDE.md`.

**One PRD per era, frozen.** Once an era ships, its PRD freezes as history — feature changes live in specs, not PRD edits (an ever-edited PRD becomes a Frankenstein that reads like a press release for a product that doesn't exist). A real direction shift — a new market, a major capability set, a pivot — gets a *new* dated PRD in `docs/prd/`, and the directory listing becomes the strategic timeline. Most projects have one PRD that lasts years.

Formats, two worked PRDs, the era-boundary heuristics, the commands that draft and review a PRD, and slicing it into features: [`prd-guide.md`](prd-guide.md).

---

## The trio: SPEC.md, PLAN.md, TASKS.md

Every feature runs through three short files, written in order: `SPEC.md` (*what & why*, with acceptance criteria as the test contract) → `PLAN.md` (*how*: decisions, file structure, constraints) → `TASKS.md` (*in what order*, each step ending in `→ verify:`). The spec is the first document the agent reads when implementing, the one the PR reviewer reads before approving, and the one future-you reads when wondering why the code is shaped this way.

> **The full method lives in [The Feature Trio: spec → plan → tasks](spec-plan-tasks-guide.md) — the centerpiece of this repo:** what sections each file needs, three worked examples, the command for each authoring step, the iteration loops, and the consistency checks that keep the three documents honest. Starting points: [`templates/spec.md`](../templates/spec.md), [`templates/plan.md`](../templates/plan.md), [`templates/tasks.md`](../templates/tasks.md).

### Rules of thumb

- **A spec over ~150 lines is usually two specs in a trenchcoat.** Split.
- **The spec ships before the code.** Written after the PR to explain code that exists, it's documentation — file it under `docs/`, not `specs/`.
- **Open questions are valuable.** Three answered honestly beat zero questions and three silent assumptions — the agent fabricates plausible answers for whatever you leave unsaid.
- **A plan over ~200 lines has outgrown one change.** Keep this change's decisions in `PLAN.md`; move durable, cross-feature structure into `ARCHITECTURE.md`.
- **One task should be reviewable in 5–15 minutes.** A task that takes more than ~1–2 hours of focused work won't be — split it.
- **Skip `TASKS.md` for a one- or two-commit change.** A one-task list is just the spec with extra ceremony.
- **Tasks freeze messy.** When the plan changes mid-flight, update `PLAN.md` and append tasks — don't retcon the old ones. The Notes are the most underrated part of the file.

### Spec status lifecycle

A spec moves through four states — the same vocabulary [`templates/spec.md`](../templates/spec.md) uses:

1. **Draft** — being written and reviewed; open questions still listed.
2. **Active** — open questions answered (or moved to ADRs); `PLAN.md` and `TASKS.md` written alongside; engineer is coding.
3. **Shipped** — PR merged. Append `STATUS: shipped (PR #N, YYYY-MM-DD)` at the top of `SPEC.md` (and if you use the optional `Status:` header, flip it too — the appended line is the lightweight default, the header is for teams that want a visible state field). From this point the spec is *history*: never edit retroactively. It's frozen as the **intent you had at merge, not as a description of current behavior** — if the spec and the code later disagree, the code wins. If the feature changes, write a new spec. (Need to flag that a shipped spec has gone stale without rewriting it? Append a dated, one-line erratum at the bottom — `Correction (YYYY-MM-DD): <what's now wrong> → see <newer spec / ADR / the code>` — append-only: a pointer, never an edit to the body, and never a substitute for a new spec when the feature actually changes.)
4. **Superseded** — a later spec replaces this one; link the replacement.

**Changing a spec mid-implementation is allowed — the freeze starts at merge.** If an acceptance criterion turns out wrong while the status is still *Active*, edit the spec, mark the edit visibly (e.g. `CHANGED during implementation: <what and why>` next to the AC), update `PLAN.md`/`TASKS.md` to match, and re-run the trio consistency check. What's forbidden is silently rewriting history *after* the PR merges — that's when a change means a new spec.

The most common rot pattern: specs sitting in *Draft* indefinitely. A spec older than ~2 weeks with unanswered Open Questions should either move forward — resolve the open questions and start implementation — or be deleted; stale drafts pollute the catalog and erode trust in the discipline.

---

## The Three-Layer Documentation Model

Once you ship v1, the docs live on three different clocks. A repo that confuses them — editing old ADRs, treating the PRD as living, leaving specs perpetually unfinished — rots fastest.

### Layer 1: Stable — evolves with the code

- `CLAUDE.md` — conventions, stack, what NOT to do
- `ARCHITECTURE.md` — high-level structure, layers, boundaries
- `DOMAIN.md` — domain model, terminology glossary

The "physics" of your codebase. The agent reads them nearly every session — `CLAUDE.md` automatically, the rest via its pointers. Change them only when fundamentals shift, and in the same PR as the change that moved them.

### Layer 2: Frozen — written once, then history

- `specs/2026-01-order-retry/` — one folder (or one file) per change; append `STATUS: shipped (PR #123, 2026-01-15)` at merge, never edit after
- `docs/prd/` — one PRD per era, frozen once that era ships

Frozen as the intent you had at the time, not as a description of today's behavior: when a frozen doc and the code disagree, the code wins.

### Layer 3: Append-only — decisions and incidents

- `docs/adr/ADR-007-dapper-instead-of-ef.md`, `docs/adr/ADR-003-quartz-for-scheduler.md`
- `docs/postmortems/`, `CHANGELOG.md`

New entries are added; old ones never change. If you change your mind, write a new ADR with `Supersedes: ADR-007` — only the old one's status header moves. The history is the value.

---

## Workflow for Changes

Each feature (sliced out of the PRD — see [The whole flow](#the-whole-flow)) goes through the **trio**, compressed to fit the change — you compress small changes, you don't skip the discipline. A one-line tweak is a PR description, a bugfix a short `SPEC.md`, a small feature a one-file trio, a real feature the three-file trio; the full table is [`spec-plan-tasks-guide.md` § When to skip parts of the trio](spec-plan-tasks-guide.md#when-to-skip-parts-of-the-trio). Two rules sit on top of it:

- **If the change moves the stable layer, the same PR updates it.** A new component, boundary, data flow, or external dependency → the PR author updates `ARCHITECTURE.md`; a new domain concept → `DOMAIN.md`.
- **A large change (refactor, stack change, new pattern) starts with an ADR** — why, alternatives rejected, consequences — and only then an implementation spec that references it.

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

**Write ADRs after the fact when you must.** Ideally an ADR is written before the decision. In practice you often discover a pattern after 3 similar implementations — then write the ADR retroactively and link future specs to it.

**"Impact on existing code" section in every SPEC.md.** Forces thinking about side effects before the agent starts generating. This is where you catch "ah, this breaks the `IBaseHandler<TSelf>` contract" *before* four hours of debugging.

**Link spec to PR.** In the PR description: `Implements: specs/2026-01-order-retry/`. After merge, append the PR number to SPEC.md. After a year you have full traceability: feature → spec → code → decision.

---

## SDD in Teams

Solo, the docs protect future-you; add a second person and they become a shared source of truth. What changes is small: **one named owner per artifact, never "the team"**; review in the PR, author plus one reviewer; docs ride in the same PR as the code. The light version for 2–10 people — driver / reviewer / keeper, the spec lifecycle, ADRs as a shared decision log, onboarding, and the failure modes that actually hit small teams — is [`sdd-in-teams-guide.md`](sdd-in-teams-guide.md).

---

## Architecture Decision Records (ADR)

ADRs (Michael Nygard, 2011) fill the gap code leaves: **code shows *what* you did, not *why*.** Each ADR is a short, dated, immutable record of one decision — its context, the choice, the consequences, and the alternatives rejected.

**Write one when a decision** is hard to reverse (ORM, message broker, auth approach), has consequences beyond a single module, was made against the obvious choice (*"why Dapper when the .NET standard is EF?"*), or keeps resurfacing in discussions. **Don't** for naming and style (`CLAUDE.md`, `.editorconfig`), decisions inside one feature (`SPEC.md`), or things obvious from the stack. The test: if a year from now someone asks *"why the hell did we do it this way?"* — that's an ADR.

**Never edit the body of an Accepted ADR.** When the decision changes, write a new ADR with `Supersedes: ADR-NNN` and update only the old one's status header to `Superseded by ADR-NNN — YYYY-MM-DD`. Following the Supersedes links is what makes the decision lineage readable a year later.

Template: [`templates/ADR.md`](../templates/ADR.md); a compact real one: [`ADR-015`](../examples/order-export/docs/adr/ADR-015-delivery-retry-backoff.md) in the order-export example. The full how-to — the Nygard format section by section, MADR and the compact variant, four worked examples, the lifecycle, numbering, anti-patterns — is [`adr-guide.md`](adr-guide.md).

---

## Migrating from a Legacy Repo to SDD

Already have a five-year-old codebase with no `CLAUDE.md` and decisions buried in old Slack threads? Skip the "documentation sprint" — two weeks, fifty files, stale by week three. Instead: **foundation first, in days** (`CLAUDE.md`, `ARCHITECTURE.md`, maybe `DOMAIN.md`, drafted by the agent from your code and cut hard by you); **specs go forward, never backward** (a spec for code that already shipped is fan fiction); **ADRs are reactive** (write one when the agent fights an unwritten decision). When a doc and the code disagree, the code wins. The week-by-week shape, the prompts, and worked examples: [`legacy-to-sdd-migration-guide.md`](legacy-to-sdd-migration-guide.md).

---

## Runbook vs Postmortem

Both cover incidents, on different clocks. A **runbook** entry is what the on-call does *right now* — symptoms → diagnosis → recovery → verification — and it's living: updated within 24 h of every incident. A **postmortem** is what happened, why, and what changes — written once, blameless, then frozen. They're a sequence, not alternatives: a postmortem usually produces a runbook update. Write the first runbook entry when the first incident happens, not before. The full comparison, the post-incident sequence, and how to write entries that pass the 3 a.m. test: [`runbook-operations-guide.md`](runbook-operations-guide.md); a postmortem template: [`templates/postmortem.md`](../templates/postmortem.md).

---

## Repository Organization

Almost none of the files above exist on day 1. The earliest version of any SDD repo is:

```
/
├── README.md
├── CLAUDE.md
└── specs/
    └── <first-feature>/spec.md
```

Three files. Everything else is added when a specific trigger fires ([the trigger table](#add-the-rest-reactively-not-proactively)). A new repo that ships a full documentation tree on day 1 has docs that are 80% wrong, because they were written without active context. A year in, a healthy repo has the stable layer at the root, `docs/` for ADRs, PRDs, runbooks and integration notes, and `specs/` with one folder or file per change.

### Common variations

- **Monorepo, shared code** (services of one system) — one root `CLAUDE.md` for cross-service conventions plus a `CLAUDE.md` per service for service-specific stack/patterns; same split for `ARCHITECTURE.md` (root: service boundaries; per-service: internals). Reusable code lives in `packages/`, deployables in `apps/` (or `services/`). ADRs: global `docs/adr/` for cross-cutting decisions, per-service for the rest — see [adr-guide.md](adr-guide.md) and the sub-`CLAUDE.md` pattern in [claude-md-guide.md](claude-md-guide.md).
- **Monorepo, no shared code** (independent apps under one roof) — drop `packages/`; each app under `apps/<name>/` is an island with its own deps, lockfile, tests, `docs/` (spec→plan→tasks + local ADRs), and `CLAUDE.md`. The root holds only the genuinely global: README-as-map, shared conventions, cross-cutting ADRs. The one real trap is CI — add path filters so a change under `apps/<name>/**` builds and releases only that app; without them every commit rebuilds everything and you lose independent deploys. If the apps share *nothing*, confirm one repo still earns its keep (atomic commits, one clone, shared conventions) over separate repos before committing to the layout.
- **Small project / solo dev** — collapse `docs/` content into top-level files until it gets unwieldy. A single `ARCHITECTURE.md` covering both structure and runbook is fine until it crosses ~600 lines.
- **OSS project** — heavier emphasis on `CONTRIBUTING.md` and `LICENSE`; lighter on `ROADMAP.md` and `RUNBOOK.md` (operational stuff is private). `CLAUDE.md` written with external contributors' agents in mind.

### The principle behind the layout

Every folder and every file should be there because **something pulled it into being** — a real reader, a real trigger, a real recurring need. If you find yourself adding files because *"the template says to,"* stop. A mature layout is a description of the destination, not a prescription for the path.

---

## Working with the Agent

This guide covers *what* to write down; [`working-with-agents-guide.md`](working-with-agents-guide.md) covers *how to put it in front of the agent* — how the agent finds and loads files, how many docs is too many, the token economy and prompt caching, the prompts for starting and ending sessions, and what to do when the agent drifts or can't build from a spec. Read it once before you use this workflow on a real project.

---

## Enforcing and Evaluating SDD

Can you make the discipline stick without everyone remembering every rule? For most 1–10 teams: good docs the agent reads, one human on the PR, and the break-the-code check already in `/sdd-7-implement` — plus one mechanical gate (a PII pre-commit hook, an Accepted-ADR lock) added only when a real pain shows up. The rule for any gate: block only when a false positive is cheap, warn when it's expensive, and leave judgment to a human. The details, ready scripts, and the Claude Code hooks: [`quality-gates-guide.md`](quality-gates-guide.md).

---

## Golden Rules

1. **Don't create documents on spec.** Add the next one when:
   - You explain the same thing to the agent for the third time (→ this should be a document)
   - You return to old code and don't remember why (→ ADR or journal)
   - You onboard someone and notice something obvious is missing (→ ONBOARDING or CONFIG)
   - The agent generates something against an unwritten convention (→ CLAUDE.md or TESTING.md)

2. **A document without an owner is dead.** Every file you add must answer: *"who reviews this, and when?"*

3. **A repo with 30 stale markdown files is worse than a repo with 5 always-fresh ones.** Discipline > completeness.

4. **The PRD (and any research behind it) is for humans + agent context, never a code source.** The agent reads `SPEC.md → PLAN.md → TASKS.md`. PRDs live in `docs/prd/` (frozen per era; multiple over time = fine, editing old ones = Frankenstein). Research, if you keep any, is synthesis-only and anonymized — and most 1–10 teams just cite a few customer conversations in the PRD.

5. **ADRs are immutable; specs freeze as authored intent, not current truth.** A frozen ADR is a decision and loses nothing. A frozen spec is a snapshot of intent on the day it merged — when it and the code later disagree, the *code* wins; read it as history, not as a description of today's behavior. Only the stable layer evolves continuously.

6. **Context per task, not context per project.** Don't dump everything on the agent. Pick what's relevant.

7. **A short ADR written today beats a perfect one written never.**

8. **The plan is alive, but decision history must be visible.** Move open questions to decisions; never delete them.

9. **Code shows what. Documentation shows why.** If the why isn't written down, it doesn't exist.

10. **Spec-driven development is a discipline, not a methodology.** Apply it daily, not as a one-time event.

---

*This guide reflects practical patterns from real-world .NET projects using Claude Code, Copilot, and similar AI coding agents. Adapt to your stack and team size.*
