# Spec-Driven Development with AI Coding Agents

A collection of practical guides for structuring your repository so AI coding agents (Claude Code, Copilot, Cursor) produce consistent, maintainable code instead of drifting away from your intent.

## Repository layout

```
.
├── README.md                          # you are here
├── guides/                            # long-form methodology guides
│   ├── spec-driven-development-guide.md
│   ├── runbook-operations-guide.md
│   ├── working-with-agents-guide.md
│   ├── claude-md-guide.md             # how to write a good CLAUDE.md (incl. many-docs repos)
│   ├── legacy-to-sdd-migration-guide.md   # retrofitting SDD onto an existing repo
│   └── sdd-in-the-wild.md             # who actually practices SDD principles
└── templates/                         # copy-pasteable starting points
    ├── PRD.md
    ├── spec.md
    ├── plan.md
    ├── ADR.md
    └── runbook.md
```

## Guides

- **[Spec-Driven Development Guide](guides/spec-driven-development-guide.md)** — the core methodology: PRD, PLAN.md, the three-layer documentation model, ADRs, and the workflow that ties them together.
- **[Runbook / Operations Documentation Guide](guides/runbook-operations-guide.md)** — companion guide for the operational layer: what to do when things break, how to write entries that pass the 3 a.m. test, and how the agent reads runbooks when generating diagnostic scripts.
- **[Working with AI Coding Agents](guides/working-with-agents-guide.md)** — how the agent actually reads your repo, when it loads a file (and why it sometimes doesn't), how many docs is too many before things break down, plus the practical prompting patterns for specs, ADRs, refactors, and runbook work.
- **[Writing a Good CLAUDE.md](guides/claude-md-guide.md)** — the deep-dive on the single most important file in an SDD repo: what goes in (and what doesn't), good-vs-bad example lines, how to size it by project scale, and what changes when your repo has 30+ docs competing for the agent's attention.
- **[Migrating a Legacy Repo to SDD](guides/legacy-to-sdd-migration-guide.md)** — the one-time process of retrofitting SDD onto an existing codebase: 30-minute audit, week-1 foundation, forward-only specs, reactive ADRs, reusable agent prompts, anti-patterns (the big-bang sprint, fabricated-history ADRs), and worked examples (Python web app, .NET monorepo, C# legacy, OSS project).
- **[SDD in the Wild](guides/sdd-in-the-wild.md)** — notes on which teams and companies publicly practice spec-driven (or spec-adjacent) engineering: GitHub spec-kit, Anthropic Claude Code, Amazon's PR-FAQ, Google design docs, Stripe RFCs, Basecamp Shape Up, and the methodologies that pre-date the SDD label.

## Templates

Minimal starting points. Copy into your project and fill in the brackets:

- **[PRD.md](templates/PRD.md)** — product requirements document (problem, users, success criteria, constraints) — the *starting* artifact, freezes after v1
- **[spec.md](templates/spec.md)** — feature specification (goal, scope, acceptance criteria, open questions)
- **[plan.md](templates/plan.md)** — implementation plan (decisions, file structure, tasks, constraints)
- **[ADR.md](templates/ADR.md)** — architecture decision record (context, decision, consequences, alternatives)
- **[runbook.md](templates/runbook.md)** — operational runbook entry (symptoms, diagnosis, recovery, verification)

## Who this is for

Engineers and teams using AI coding agents on real projects (not toy examples) who want:

- Generated code that matches their conventions instead of drifting
- A repository structure that scales beyond the first few features
- Documentation that the agent actually uses, not documentation that exists for its own sake
- Operational discipline so the system stays running, not just builds green

## How to read

Start with the [Spec-Driven Development Guide](guides/spec-driven-development-guide.md). It covers the foundational layers — `CLAUDE.md`, `ARCHITECTURE.md`, `DOMAIN.md`, ADRs, and per-feature specs — and the workflow that ties them together.

Then read [Working with AI Coding Agents](guides/working-with-agents-guide.md). It's the thinnest layer between the documentation conventions in the first guide and the prompts that actually put them in front of the agent.

Read the [Runbook / Operations Guide](guides/runbook-operations-guide.md) once you have a system in production (or close to it) and need to capture *how to keep it running* alongside *how it was built*.

All three guides assume a mid-sized project context (.NET, Quartz, SFTP integrations, Dapper, MS SQL Server) but the patterns transfer to any stack.

## Status

These guides are opinionated distillations of patterns from real-world projects. Adapt to your stack, team size, and risk tolerance — the discipline matters more than the exact filenames.
