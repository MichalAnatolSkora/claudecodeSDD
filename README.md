# Spec-Driven Development with AI Coding Agents

> **Built for the 10-person, billion-dollar company.** A small team wielding AI coding agents can now ship what used to take hundreds of engineers — but raw speed isn't the moat. The moat is keeping the agents aligned with your intent as the codebase grows. Spec-driven development is that discipline; these guides are the playbook.

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
│   ├── adr-guide.md                   # full ADR how-to: format, lifecycle, examples, anti-patterns
│   ├── prd-guide.md                   # PRD how-to: formats, era pattern, two worked examples, AI prompts
│   ├── research-guide.md              # research in the repo: PII, folder structure, synthesis, agent context
│   ├── quality-gates-guide.md         # enforcement + evaluation: hooks, subagents, slash commands, CI
│   ├── spec-plan-tasks-guide.md       # full worked-example trio + AI-authoring prompts
│   ├── sdd-in-teams-guide.md          # roles, ownership, handoffs, OSS, regulated industries
│   ├── legacy-to-sdd-migration-guide.md   # retrofitting SDD onto an existing repo
│   └── sdd-in-the-wild.md             # who actually practices SDD principles
└── templates/                         # copy-pasteable starting points
    ├── CLAUDE.md                      # agent instruction hub (behavioral + project layer)
    ├── PRD.md
    ├── spec.md
    ├── plan.md
    ├── tasks.md                       # execution checklist with verification per step
    ├── ADR.md
    ├── runbook.md
    ├── postmortem.md                   # blameless incident analysis (timeline, root cause, lessons, action items)
    └── research-synthesis.md           # interview synthesis (anonymized; raw stays outside repo)
```

## Guides

- **[Spec-Driven Development Guide](guides/spec-driven-development-guide.md)** — the core methodology: PRD, PLAN.md, the three-layer documentation model, ADRs, and the workflow that ties them together.
- **[Runbook / Operations Documentation Guide](guides/runbook-operations-guide.md)** — companion guide for the operational layer: what to do when things break, how to write entries that pass the 3 a.m. test, and how the agent reads runbooks when generating diagnostic scripts.
- **[Working with AI Coding Agents](guides/working-with-agents-guide.md)** — how the agent actually reads your repo, when it loads a file (and why it sometimes doesn't), how many docs is too many before things break down, plus the practical prompting patterns for specs, ADRs, refactors, and runbook work.
- **[Writing a Good CLAUDE.md](guides/claude-md-guide.md)** — the deep-dive on the single most important file in an SDD repo: what goes in (and what doesn't), good-vs-bad example lines, how to size it by project scale, and what changes when your repo has 30+ docs competing for the agent's attention.
- **[Writing Good ADRs](guides/adr-guide.md)** — the full how-to for Architecture Decision Records: Nygard format section-by-section, alternative formats (MADR, Y-statements), four worked examples (Accepted, Superseded pair, Proposed, Deprecated), the Supersedes pattern in depth, numbering, cross-referencing, anti-patterns, tooling, maintenance.
- **[Writing a Good PRD (Per Era, For Humans)](guides/prd-guide.md)** — the practice companion to main SDD's "The PRD Layer": format alternatives (PR-FAQ, lean PRD, one-pager, full), anatomy section-by-section, two complete worked PRDs (original launch + era-2 expansion), era-boundary heuristics, five AI-authoring prompts, success-metrics deep-dive, stakeholder review process, PRD-specific anti-patterns.
- **[Research in the Repo](guides/research-guide.md)** — the upstream-of-PRD layer: what synthesized research belongs in the repo (and what stays out for PII reasons), folder structure for `docs/research/`, the five artifact types (interview synthesis, competitive, sizing, validation, opportunity briefs), the synthesis discipline, AI-assisted synthesis prompts, the PRD↔research interface, and research-specific anti-patterns. Research is for humans + agent context; the agent never generates code from research.
- **[Quality Gates: Enforcing and Evaluating SDD](guides/quality-gates-guide.md)** — the "how do I make SDD stick?" guide: three categories of checks (mechanical / LLM evaluator / human), five implementation patterns (pre-commit hooks, Claude Code hooks, configured subagents, slash commands, CI/CD), what to mechanize vs leave human, a complete worked-example setup, and the anti-patterns of over-automation (alert fatigue, no escape hatch, mechanical checks of subjective things).
- **[Writing the Feature Trio: spec → plan → tasks](guides/spec-plan-tasks-guide.md)** — the practice companion to the main SDD's three principle-level sections: two full worked examples (a feature + a bugfix) showing what `spec.md` + `plan.md` + `tasks.md` look like filled in for the same change, six AI-authoring prompts (draft/review/validate per artifact), iteration loops, cross-artifact consistency checks, and the slash commands worth turning into shortcuts.
- **[SDD in Teams](guides/sdd-in-teams-guide.md)** — running SDD with more than one engineer: per-role profiles (Engineer / Tech Lead / EM / PM / Founder / QA / SRE / Security / OSS contributors), artifact ownership in depth, lifecycle handoffs, cadence patterns, monorepo specifics, the OSS case, regulated-industry overlays, onboarding flow, team failure modes, a RACI matrix.
- **[Migrating a Legacy Repo to SDD](guides/legacy-to-sdd-migration-guide.md)** — the one-time process of retrofitting SDD onto an existing codebase: 30-minute audit, week-1 foundation, forward-only specs, reactive ADRs, reusable agent prompts, anti-patterns (the big-bang sprint, fabricated-history ADRs), and worked examples (Python web app, .NET monorepo, C# legacy, OSS project).
- **[SDD in the Wild](guides/sdd-in-the-wild.md)** — notes on which teams and companies publicly practice spec-driven (or spec-adjacent) engineering: GitHub spec-kit, Anthropic Claude Code, Amazon's PR-FAQ, Google design docs, Stripe RFCs, Basecamp Shape Up, and the methodologies that pre-date the SDD label.

## Templates

Minimal starting points. Copy into your project and fill in the brackets:

- **[CLAUDE.md](templates/CLAUDE.md)** — agent instruction hub with two layers: literal-copied behavioral guidelines (from [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/CLAUDE.md), kept verbatim with attribution) + bracketed project-specific sections you fill in
- **[PRD.md](templates/PRD.md)** — product requirements document (problem, users, success criteria, constraints) — the *starting* artifact, freezes after v1
- **[spec.md](templates/spec.md)** — feature specification (goal, scope, acceptance criteria, open questions)
- **[plan.md](templates/plan.md)** — implementation plan (decisions, file structure, tasks, constraints)
- **[tasks.md](templates/tasks.md)** — execution checklist with one verification per step
- **[ADR.md](templates/ADR.md)** — architecture decision record (context, decision, consequences, alternatives)
- **[runbook.md](templates/runbook.md)** — operational runbook entry (symptoms, diagnosis, recovery, verification)
- **[postmortem.md](templates/postmortem.md)** — blameless incident analysis (summary, timeline, root cause, lessons, action items)
- **[research-synthesis.md](templates/research-synthesis.md)** — interview synthesis with themed evidence + role-attributed quotes (anonymized; raw transcripts stay outside the repo)

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
