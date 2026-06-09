# Spec-Driven Development with AI Coding Agents

> **Built for the 10-person, billion-dollar company.** A small team wielding AI coding agents can now ship what used to take hundreds of engineers — but raw speed isn't the moat. The moat is keeping the agents aligned with your intent as the codebase grows. Spec-driven development is that discipline; these guides are the playbook.

A collection of practical guides for structuring your repository so AI coding agents (Claude Code, Copilot, Cursor) produce consistent, maintainable code instead of drifting away from your intent.

**New here? Start with [SDD in 5 files](guides/sdd-in-5-files.md) — the entire method on one page.** Most readers never need more than that. When you want depth, the one guide to read is [The Feature Trio: spec → plan → tasks](guides/spec-plan-tasks-guide.md) — the core loop, three short markdown files written in order, that everything else here exists to support.

## Repository layout

```
.
├── README.md                          # you are here
├── guides/                            # long-form methodology guides
│   ├── sdd-in-5-files.md              # the whole method on one page — the front door
│   ├── spec-driven-development-guide.md
│   ├── runbook-operations-guide.md
│   ├── working-with-agents-guide.md
│   ├── testing-guide.md              # testing in SDD: agent-written tests, TESTING.md, what to test
│   ├── claude-md-guide.md             # how to write a good CLAUDE.md (incl. many-docs repos)
│   ├── adr-guide.md                   # full ADR how-to: format, lifecycle, examples, anti-patterns
│   ├── prd-guide.md                   # PRD how-to: formats, era pattern, two worked examples, AI prompts
│   ├── research-guide.md              # research in the repo: PII, folder structure, synthesis, agent context
│   ├── quality-gates-guide.md         # enforcement + evaluation: hooks, subagents, slash commands, CI
│   ├── spec-plan-tasks-guide.md       # ★ the core loop — worked trio + AI prompts (read first)
│   ├── flow-guide.md                  # the whole flow end to end: every command, step by step
│   ├── sdd-in-teams-guide.md          # SDD for 1–10 people: ownership, review, onboarding (light)
│   ├── legacy-to-sdd-migration-guide.md   # retrofitting SDD onto an existing repo
│   └── who-uses-sdd.md             # who uses SDD and what they call it (plain who-uses-what)
├── templates/                         # copy-pasteable starting points
    ├── .claude/                       # trio slash commands (copy into your project)
    ├── CLAUDE.md                      # agent instruction hub (behavioral + project layer)
    ├── PRD.md
    ├── spec.md
    ├── plan.md
    ├── tasks.md                       # execution checklist with verification per step
    ├── TESTING.md                     # test conventions the agent reads (framework, mocking, "done")
    ├── ADR.md
    ├── runbook.md
    ├── postmortem.md                   # blameless incident analysis (timeline, root cause, lessons, action items)
    └── research-synthesis.md           # interview synthesis (anonymized; raw stays outside repo)
└── examples/                          # worked end-to-end SDD doc sets (illustrative, docs-only)
    └── order-export/                  # one fictional app: PRD → trio → ADR, all cross-linked
```

## Guides

- **[SDD in 5 files](guides/sdd-in-5-files.md)** — the whole method on one page: the five files you start with, the per-change loop, and when to add more. The front door — start here, and descend into the rest only when a real need fires.
- **[Spec-Driven Development Guide](guides/spec-driven-development-guide.md)** — the entry point and map: PRD, PLAN.md, the three-layer documentation model, ADRs, and the workflow that ties them together.
- **★ [The Feature Trio: spec → plan → tasks](guides/spec-plan-tasks-guide.md)** — **the core loop of the whole method, and the one guide to read first.** Three short markdown files written in order (`spec.md` → `plan.md` → `tasks.md`) drive every change. Inside: two complete worked examples (a feature as the full three-file trio, and a small feature as a one-file trio), what sections each file needs, six AI-authoring prompts (draft/review/validate per artifact), iteration loops, cross-artifact consistency checks, and the slash commands worth turning into shortcuts. Deliberately simple — no toolkit required.
- **[The whole flow, end to end](guides/flow-guide.md)** — the complete loop as a runnable sequence: every command and what it produces, from idea → PRD → slice → trio → implement+test → ADR → merge, plus a one-screen cheat sheet. Enter where your change starts; skip the rest.
- **[Runbook / Operations Documentation Guide](guides/runbook-operations-guide.md)** — companion guide for the operational layer: what to do when things break, how to write entries that pass the 3 a.m. test, and how the agent reads runbooks when generating diagnostic scripts.
- **[Working with AI Coding Agents](guides/working-with-agents-guide.md)** — how the agent actually reads your repo, when it loads a file (and why it sometimes doesn't), how many docs is too many before things break down, plus the practical prompting patterns for specs, ADRs, refactors, and runbook work.
- **[Testing in SDD](guides/testing-guide.md)** — how testing fits the trio and how the AI agent writes tests you can trust: acceptance criteria as the test contract, the `TESTING.md` conventions file, TDD red→green with an agent, getting good tests out of it (and the break-the-code check so you don't trust green), and pragmatic coverage at 1–10.
- **[Writing a Good CLAUDE.md](guides/claude-md-guide.md)** — the deep-dive on the single most important file in an SDD repo: what goes in (and what doesn't), good-vs-bad example lines, how to size it by project scale, and what changes when your repo has 30+ docs competing for the agent's attention.
- **[Writing Good ADRs](guides/adr-guide.md)** — the full how-to for Architecture Decision Records: Nygard format section-by-section, alternative formats (MADR, Y-statements), four worked examples (Accepted, Superseded pair, Proposed, Deprecated), the Supersedes pattern in depth, numbering, cross-referencing, anti-patterns, tooling, maintenance.
- **[Writing a Good PRD (Per Era, For Humans)](guides/prd-guide.md)** — the practice companion to main SDD's "The PRD Layer": format alternatives (PR-FAQ, lean PRD, one-pager, full), anatomy section-by-section, two complete worked PRDs (original launch + era-2 expansion), era-boundary heuristics, five AI-authoring prompts, success-metrics deep-dive, stakeholder review process, PRD-specific anti-patterns.
- **[Research in the Repo](guides/research-guide.md)** — the upstream-of-PRD layer: what synthesized research belongs in the repo (and what stays out for PII reasons), folder structure for `docs/research/`, the five artifact types (interview synthesis, competitive, sizing, validation, opportunity briefs), the synthesis discipline, AI-assisted synthesis prompts, the PRD↔research interface, and research-specific anti-patterns. Research is for humans + agent context; the agent never generates code from research.
- **[Quality Gates: Enforcing and Evaluating SDD](guides/quality-gates-guide.md)** — the "how do I make SDD stick?" guide: three categories of checks (mechanical / LLM evaluator / human), five implementation patterns (pre-commit hooks, Claude Code hooks, configured subagents, slash commands, CI/CD), what to mechanize vs leave human, a complete worked-example setup, and the anti-patterns of over-automation (alert fatigue, no escape hatch, mechanical checks of subjective things).
- **[SDD in Teams (1–10 People)](guides/sdd-in-teams-guide.md)** — what changes when SDD goes from solo to a small team, kept deliberately light: who owns what (one name per artifact), lightweight PR-based review, the spec lifecycle, ADRs as a shared decision log, the solo case, onboarding, the failure modes that actually hit small teams, and what to add only once you outgrow ~10.
- **[Migrating a Legacy Repo to SDD](guides/legacy-to-sdd-migration-guide.md)** — the one-time process of retrofitting SDD onto an existing codebase: 30-minute audit, week-1 foundation, forward-only specs, reactive ADRs, reusable agent prompts, anti-patterns (the big-bang sprint, fabricated-history ADRs), and worked examples (Python web app, .NET monorepo, C# legacy, OSS project).
- **[Who uses SDD (and what they call it)](guides/who-uses-sdd.md)** — plain "who uses what" tables: AI-coding tools (GitHub spec-kit, Claude Code + `CLAUDE.md`, Cursor `.cursorrules`, Aider/Continue/Cline) and the pre-AI crowd (Amazon PR-FAQ, Google design docs, Stripe RFCs, Basecamp Shape Up), plus the older ideas SDD borrows from. Useful when someone calls SDD a fad.

## Templates

Minimal starting points. Copy into your project and fill in the brackets:

**Bootstrap into a new repo** — run *from inside your new repo*. Pick based on whether you also want a starter `CLAUDE.md`:

**Just the commands** (fastest, needs `npx`):

```bash
npx degit MichalAnatolSkora/claudecodeSDD/templates/.claude .claude
```

That drops all ten commands (`/prd-new`, `/features-from-prd`, `/spec-new`, `/spec-review`, `/plan-from-spec`, `/plan-validate`, `/tasks-from-plan`, `/tasks-add`, `/trio-check`, `/implement`) into `.claude/commands/`, ready to use.

**Commands + a starter `CLAUDE.md`** (plain git — degit can't grab the single `CLAUDE.md` file):

```bash
git clone --depth 1 https://github.com/MichalAnatolSkora/claudecodeSDD /tmp/sdd && cp -r /tmp/sdd/templates/.claude /tmp/sdd/templates/CLAUDE.md . && rm -rf /tmp/sdd
```

That drops `.claude/` (the commands) **and** a bracketed `CLAUDE.md` at your repo root — then fill in the brackets (see [Writing a Good CLAUDE.md](guides/claude-md-guide.md)).

- **[CLAUDE.md](templates/CLAUDE.md)** — agent instruction hub with two layers: literal-copied behavioral guidelines (from [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/CLAUDE.md), kept verbatim with attribution) + bracketed project-specific sections you fill in
- **[PRD.md](templates/PRD.md)** — product requirements document (problem, users, success criteria, constraints) — the *starting* artifact, freezes after v1
- **[spec.md](templates/spec.md)** — feature specification (goal, scope, acceptance criteria, open questions)
- **[plan.md](templates/plan.md)** — implementation plan (decisions, file structure, constraints)
- **[tasks.md](templates/tasks.md)** — execution checklist with one verification per step
- **[TESTING.md](templates/TESTING.md)** — the test conventions the agent reads before writing tests (framework, where tests live, mocking rules, what "done" means)
- **[.claude/ trio commands](templates/.claude/)** — copy-pasteable Claude Code slash commands (`/prd-new`, `/features-from-prd`, `/spec-new`, `/spec-review`, `/plan-from-spec`, `/plan-validate`, `/tasks-from-plan`, `/tasks-add`, `/trio-check`, `/implement`) that take an idea from PRD through the trio and into implementation
- **[ADR.md](templates/ADR.md)** — architecture decision record (context, decision, consequences, alternatives)
- **[runbook.md](templates/runbook.md)** — operational runbook entry (symptoms, diagnosis, recovery, verification)
- **[postmortem.md](templates/postmortem.md)** — blameless incident analysis (summary, timeline, root cause, lessons, action items)
- **[research-synthesis.md](templates/research-synthesis.md)** — interview synthesis with themed evidence + role-attributed quotes (anonymized; raw transcripts stay outside the repo)

## Examples

- **[examples/order-export/](examples/order-export/)** — a complete, illustrative SDD paper trail for one fictional app (a B2B order-export platform): a PRD, the stable layer (`CLAUDE.md`, `ARCHITECTURE.md`, `DOMAIN.md`, `TESTING.md`), one feature taken through the full `spec → plan → tasks` trio, and the ADR it produced — all cross-linked so you can follow the "golden thread" end to end. Docs only; no runnable code. Start at its [README](examples/order-export/README.md).

## Who this is for

**Built for teams of 1–10** — from a solo builder to a ten-person company. Engineers and teams using AI coding agents on real projects (not toy examples) who want:

- Generated code that matches their conventions instead of drifting
- A repository structure that scales beyond the first few features
- Documentation that the agent actually uses, not documentation that exists for its own sake
- Operational discipline so the system stays running, not just builds green

## How to read

**New here? The one-page [SDD in 5 files](guides/sdd-in-5-files.md) is the front door — most people start there and don't need more.** For the fuller picture, start with the [Spec-Driven Development Guide](guides/spec-driven-development-guide.md) for the map. It covers the foundational layers — `CLAUDE.md`, `ARCHITECTURE.md`, `DOMAIN.md`, ADRs, and per-feature specs — and the workflow that ties them together.

Then read **[The Feature Trio: spec → plan → tasks](guides/spec-plan-tasks-guide.md)** — the core loop and the most important guide here. It's where the method actually lives: three short markdown files, written in order, that drive every change. If you have time for only one guide, make it this one.

Then read [Working with AI Coding Agents](guides/working-with-agents-guide.md). It's the thinnest layer between the documentation conventions in the first guide and the prompts that actually put them in front of the agent.

Read the [Runbook / Operations Guide](guides/runbook-operations-guide.md) once you have a system in production (or close to it) and need to capture *how to keep it running* alongside *how it was built*.

The guides assume a mid-sized project context (.NET, Quartz, SFTP integrations, Dapper, MS SQL Server) but the patterns transfer to any stack.

## Status

These guides are opinionated distillations of patterns from real-world projects. Adapt to your stack, team size, and risk tolerance — the discipline matters more than the exact filenames.
