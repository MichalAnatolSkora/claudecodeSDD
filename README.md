# Spec-Driven Development with AI Coding Agents

> **Built for teams of 1–10.** A small team wielding AI coding agents can now ship what used to take hundreds of engineers — but raw speed isn't the moat. The moat is keeping the agents aligned with your intent as the codebase grows. This repo gives you that discipline **and the commands that run it.**

Most SDD writeups hand you a *method*. This repo's strong point is the method **plus a runnable pipeline of 13 slash commands** — and, more importantly, **the discipline for operating them** so they fit real work instead of hardening into a waterfall.

---

## The strong point: commands that *run* SDD — and how to operate them

**The pipeline — 13 `/sdd-*` slash commands, idea to merge.** Each does one narrow thing and **stops for your judgment**: drafts are shown before saving, audits are read-only, open questions are surfaced rather than invented. The agent types; you stay the decision-maker.

```
    idea
     │
     ▼
    /prd-new · /prd-review      1 · the PRD
     │
     ▼
 ┌─ /features-from-prd          2 · vertical slices
 │   │
 │   ▼
 │  /spec-new … /trio-check     3–6 · the trio
 │   │
 │   ▼
 │  /implement                  7 · red→green
 │   │
 │   ▼
 │  merge & freeze
 │   │
 └───┘ loop: a shipped slice re-ranks the next slice
```

The skill isn't running all 13 in a row — it's **operating** them. Three rules:

1. **Enter where your change starts; skip the rest.** You rarely run the whole pipeline. A one-line fix is a short spec and `/implement`. A known feature starts at `/spec-new`. A new product starts at `/prd-new`. → [entry table](guides/flow-guide.md#at-a-glance)

2. **Compress to fit the change.** A one-liner → just a PR. A bug → a short `SPEC.md`. A small feature → a one-file trio (spec/plan/tasks in one file). A real feature → the full three-file trio. Match the ceremony to the size; when in doubt, write the shorter one.

3. **Loop, don't waterfall.** The order — spec → plan → tasks → code — is the discipline *within* a slice; *between* slices you loop. Three loops reuse the **same** commands, no new ones:
   - **delivery vs discovery** — when you can't write the acceptance criteria yet, mark the spec `discovery`, spike first (throwaway), let the AC *emerge*, then run the trio;
   - **code→spec** — when implementation proves an AC wrong, edit the still-`Active` spec in place (`CHANGED during implementation:`) and re-run `/trio-check`;
   - **re-slice** — when a shipped slice reshapes the backlog, re-run `/features-from-prd` (it merges, never resets).

   → [Iterative loops — the same commands, run in cycles](guides/flow-guide.md#iterative-loops--the-same-commands-run-in-cycles)

That third rule is the part most SDD tooling misses: a command pipeline that reads as a one-way waterfall and breaks the moment real work loops back. Here the loop is first-class — but the order discipline inside each slice stays intact.

---

## Get the commands

Run *from inside your repo*. Pick based on whether you also want a starter `CLAUDE.md`.

**Just the commands** (fastest, needs `npx`):

```bash
npx degit MichalAnatolSkora/claudecodeSDD/templates/.claude .claude
```

That drops all thirteen commands (`/prd-new`, `/prd-review`, `/features-from-prd`, `/architecture-from-prd`, `/architecture-from-code`, `/spec-new`, `/spec-review`, `/plan-from-spec`, `/plan-validate`, `/tasks-from-plan`, `/tasks-add`, `/trio-check`, `/implement`) into `.claude/commands/`, ready to use. *(The shipped files are phase-numbered — `sdd-3-spec-new.md` → `/sdd-3-spec-new` — so they sort in pipeline order; the short forms above are the same commands. Keep or drop the `sdd-N-` prefix as you like.)*

**Commands + a starter `CLAUDE.md`** (plain git — degit can't grab the single `CLAUDE.md` file):

```bash
git clone --depth 1 https://github.com/MichalAnatolSkora/claudecodeSDD /tmp/sdd && cp -r /tmp/sdd/templates/.claude /tmp/sdd/templates/CLAUDE.md . && rm -rf /tmp/sdd
```

That drops `.claude/` (the commands) **and** a bracketed `CLAUDE.md` at your repo root — then fill in the brackets (see [Writing a Good CLAUDE.md](guides/claude-md-guide.md)).

**Update an existing repo** — already have the commands and want the latest versions? Run *from your repo root*. This clears out the old SDD commands and pulls fresh copies, leaving your own custom commands (and `CLAUDE.md`) untouched:

```bash
rm -f .claude/commands/sdd-*.md && npx degit MichalAnatolSkora/claudecodeSDD/templates/.claude/commands .claude/commands --force
```

The `rm` drops the stale `sdd-*` files (so anything renamed or removed upstream doesn't linger); `--force` lets degit write into the non-empty `.claude/commands/`. Only the `sdd-*` commands are touched — refresh your `CLAUDE.md` by hand if you want upstream wording there.

No `npx`? The plain-git equivalent:

```bash
git clone --depth 1 https://github.com/MichalAnatolSkora/claudecodeSDD /tmp/sdd && rm -f .claude/commands/sdd-*.md && cp -r /tmp/sdd/templates/.claude/commands/. .claude/commands/ && rm -rf /tmp/sdd
```

---

## Start here

**New? The one-page [SDD in 5 files](guides/sdd-in-5-files.md) is the front door** — the whole method, the five files you start with, and the per-change loop. Most readers never need more.

Then, in order:

1. **[The whole flow, end to end](guides/flow-guide.md)** — the command pipeline as a runnable sequence: every `/sdd-*` command, what it produces, the entry table, the iterative loops, and a one-screen cheat sheet. **This is the "how to operate the commands" guide.**
2. **★ [The Feature Trio: spec → plan → tasks](guides/spec-plan-tasks-guide.md)** — the core loop and the one guide to read deeply. Three short markdown files written in order drive every change; the commands above are just this loop, automated. The order holds within a slice; across slices you iterate.
3. **[Spec-Driven Development Guide](guides/spec-driven-development-guide.md)** — the full map: the stable layer (`CLAUDE.md`, `ARCHITECTURE.md`, `DOMAIN.md`), PRDs, ADRs, the three-layer documentation model, and the workflow that ties them together.

The guides assume a mid-sized project context (.NET, Quartz, SFTP integrations, Dapper, MS SQL Server) but the patterns transfer to any stack.

## Guides

- **[SDD in 5 files](guides/sdd-in-5-files.md)** — the whole method on one page: the five files you start with, the per-change loop, and when to add more. The front door — start here, and descend into the rest only when a real need fires.
- **[The whole flow, end to end](guides/flow-guide.md)** — the complete loop as a runnable command sequence: every command and what it produces, from idea → PRD → slice → trio → implement+test → ADR → merge, the entry table, the iterative loops, and a one-screen cheat sheet. Enter where your change starts; skip the rest.
- **★ [The Feature Trio: spec → plan → tasks](guides/spec-plan-tasks-guide.md)** — **the core loop of the whole method, and the one guide to read first.** Three short markdown files written in order (`SPEC.md` → `PLAN.md` → `TASKS.md`) drive every change. Inside: three worked examples (a feature as the full three-file trio, a small feature as a one-file trio, and a discovery feature whose first spec was wrong), what sections each file needs, six AI-authoring prompts (draft/review/validate per artifact), iteration loops (delivery vs discovery, the code→spec correction loop), cross-artifact consistency checks, and the slash commands worth turning into shortcuts. Deliberately simple — no toolkit required.
- **[Spec-Driven Development Guide](guides/spec-driven-development-guide.md)** — the entry point and map: PRD, PLAN.md, the three-layer documentation model, ADRs, and the workflow that ties them together.
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

## Repository layout

```
.
├── README.md                          # you are here
├── guides/                            # long-form methodology guides (the "why" behind the commands)
│   ├── sdd-in-5-files.md              # the whole method on one page — the front door
│   ├── flow-guide.md                  # the command pipeline + how to operate it (incl. iterative loops)
│   ├── spec-plan-tasks-guide.md       # ★ the core loop — worked trio + AI prompts (read first)
│   ├── spec-driven-development-guide.md   # the full map
│   ├── runbook-operations-guide.md
│   ├── working-with-agents-guide.md
│   ├── testing-guide.md              # testing in SDD: agent-written tests, TESTING.md, what to test
│   ├── claude-md-guide.md             # how to write a good CLAUDE.md (incl. many-docs repos)
│   ├── adr-guide.md                   # full ADR how-to: format, lifecycle, examples, anti-patterns
│   ├── prd-guide.md                   # PRD how-to: formats, era pattern, worked examples, AI prompts
│   ├── research-guide.md              # research in the repo: PII, folder structure, synthesis, agent context
│   ├── quality-gates-guide.md         # enforcement + evaluation: hooks, subagents, slash commands, CI
│   ├── sdd-in-teams-guide.md          # SDD for 1–10 people: ownership, review, onboarding (light)
│   ├── legacy-to-sdd-migration-guide.md   # retrofitting SDD onto an existing repo
│   └── who-uses-sdd.md                # who uses SDD and what they call it (plain who-uses-what)
├── templates/                         # copy-pasteable starting points
│   ├── .claude/commands/             # the /sdd-1…7 slash-command pipeline (copy into your project)
│   ├── CLAUDE.md                      # agent instruction hub (behavioral + project layer)
│   ├── ARCHITECTURE.md                # stable architecture layer: components, boundaries, data flow
│   ├── PRD.md
│   ├── FEATURES.md                    # prioritized backlog of vertical slices from the PRD(s)
│   ├── spec.md
│   ├── plan.md
│   ├── tasks.md                       # execution checklist with verification per step
│   ├── TESTING.md                     # test conventions the agent reads (framework, mocking, "done")
│   ├── ADR.md
│   ├── runbook.md
│   ├── postmortem.md                  # blameless incident analysis (timeline, root cause, lessons, actions)
│   └── research-synthesis.md          # interview synthesis (anonymized; raw stays outside repo)
└── examples/                          # worked end-to-end SDD doc sets (illustrative, docs-only)
    └── order-export/                  # one fictional app: PRD → trio → ADR, all cross-linked
```

## Templates

The artifacts the commands produce and the agent reads. Copy into your project and fill in the brackets:

- **[.claude/ command pipeline](templates/.claude/)** — the 13 Claude Code slash commands (`/prd-new` … `/implement`) that take a change from idea through the trio into implementation. This is the operating layer; see [Get the commands](#get-the-commands) above to install.
- **[CLAUDE.md](templates/CLAUDE.md)** — agent instruction hub with two layers: literal-copied behavioral guidelines (from [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/CLAUDE.md), kept verbatim with attribution) + bracketed project-specific sections you fill in
- **[ARCHITECTURE.md](templates/ARCHITECTURE.md)** — the stable architecture layer: components, boundaries, and the main data flow the agent reads before touching a feature (the *why* lives in ADRs)
- **[PRD.md](templates/PRD.md)** — product requirements document (problem, users, success criteria, constraints) — the *starting* artifact, freezes after v1
- **[FEATURES.md](templates/FEATURES.md)** — the feature index: a prioritized backlog of vertical slices derived from the PRD(s), with status per slice (optional — `specs/*/` + git can be the map instead)
- **[spec.md](templates/spec.md)** — feature specification (goal, scope, acceptance criteria, open questions)
- **[plan.md](templates/plan.md)** — implementation plan (decisions, file structure, constraints)
- **[tasks.md](templates/tasks.md)** — execution checklist with one verification per step
- **[TESTING.md](templates/TESTING.md)** — the test conventions the agent reads before writing tests (framework, where tests live, mocking rules, what "done" means)
- **[ADR.md](templates/ADR.md)** — architecture decision record (context, decision, consequences, alternatives)
- **[runbook.md](templates/runbook.md)** — operational runbook entry (symptoms, diagnosis, recovery, verification)
- **[postmortem.md](templates/postmortem.md)** — blameless incident analysis (summary, timeline, root cause, lessons, action items)
- **[research-synthesis.md](templates/research-synthesis.md)** — interview synthesis with themed evidence + role-attributed quotes (anonymized; raw transcripts stay outside the repo)

## Examples

- **[examples/order-export/](examples/order-export/)** — a complete, illustrative SDD paper trail for one fictional app (a B2B order-export platform): a PRD, the stable layer (`CLAUDE.md`, `ARCHITECTURE.md`, `DOMAIN.md`, `TESTING.md`), features taken through the trio in both shapes (one-file and three-file), a **discovery** feature whose first spec was wrong (fixed mid-flight), and the ADR the work produced — all cross-linked so you can follow the "golden thread" end to end. Docs only; no runnable code. Start at its [README](examples/order-export/README.md).

## Who this is for

**Built for teams of 1–10** — from a solo builder to a ten-person company. Engineers and teams using AI coding agents on real projects (not toy examples) who want:

- Generated code that matches their conventions instead of drifting
- A command pipeline they drive — enter where the change starts, compress to fit, loop instead of waterfall
- A repository structure that scales beyond the first few features
- Documentation that the agent actually uses, not documentation that exists for its own sake
- Operational discipline so the system stays running, not just builds green

## Status

These guides and commands are opinionated distillations of patterns from real-world projects. Adapt to your stack, team size, and risk tolerance — the discipline matters more than the exact filenames.
