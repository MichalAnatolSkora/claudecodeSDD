# CLAUDE.md

> Conventions and meta-rules for working on this repository (the SDD/Claude Code guides themselves). Read this before editing any guide.

## Repository purpose

Long-form guides + copy-pasteable templates for spec-driven development with AI coding agents. Structure:

```
.
├── README.md                          # entry point / map
├── CLAUDE.md                          # this file — conventions for editing
├── AGENTS.md                          # thin pointer to this file (cross-tool convention name)
├── .claude/commands/                  # repo-local slash commands (e.g. /pdf build helper)
├── guides/                            # long-form methodology guides (full list under "Editorial structure")
├── templates/                         # copy-pasteable starting points
│   └── .claude/commands/              # the sdd-1 … sdd-7 slash-command pipeline for user repos
├── examples/                          # worked end-to-end SDD doc sets (illustrative, docs-only)
│   └── order-export/                  # one fictional app: PRD → trio → ADR, cross-linked
├── pdf-style.html                     # full PDF style (not used by default)
├── pdf-style-compact.html             # compact PDF style (narrower 1cm-margin variant)
└── pdf-style-wide.html                # full-width PDF style — DEFAULT
```

Built PDFs live in `pdf/` (a per-guide PDF for each `guides/*.md`, plus `claudecodeSDD-bundle.pdf`) and are **gitignored** — generated artifacts, never committed. The whole `pdf/` folder is ignored.

`examples/` holds worked, **docs-only** SDD sets (no runnable code) — one folder per fictional app, kept consistent with the guides and the neutral-naming rules below. Currently: `order-export/`.

## Editorial structure — important

`guides/spec-driven-development-guide.md` is the **overview / entry-point** guide. Treat it as a high-level introduction that links out to detail files. Keep general principles and workflow framing here; push deep mechanics into separate detail guides.

**The flagship is `guides/spec-plan-tasks-guide.md`** — the spec → plan → tasks core loop, and the most polished guide in the repo. The overview above stays the entry point / map; the trio guide is the heart of the method (both are true). Keep the trio guide deliberately simple: three markdown files and the order discipline, no toolkit / scaffolder / generated-file machinery. (We evaluated GitHub's spec-kit and intentionally stayed lighter — do not import its constitution / clarify / analyze ceremony without explicit instruction.)

**Design target: teams of 1–10.** Default every convention to the lightest form that works for a solo dev; make ceremony opt-in (e.g. optional `Status:` / `Owner:` headers, not required ones). Multi-person depth — ownership, review flow — lives in `guides/sdd-in-teams-guide.md`, which itself stays scoped to 1–10 (RACI, multi-team/monorepo, and regulated-industry gates are named there as "when you outgrow this", not documented). When an edit would add *required* structure, ask whether a one-person team would still want it; if not, make it optional.

When new content needs to land in the docs:

- **High-level principle, workflow framing, or repo-wide convention** → `spec-driven-development-guide.md`
- **Mechanics, prompts, or anti-patterns specific to one workflow** → its own detail guide, linked from the overview
- The overview's job is to point at the right detail file, not to absorb the details

Existing detail guides:

- `guides/sdd-in-5-files.md` — the one-page front door: the five-file minimum + the per-change loop, with pointers out to the depth. The lightweight entry most readers should hit first. Keep it to one screen and consistent with the overview's "Absolute Minimum" — if you change one, change the other (the two must not drift).
- `guides/working-with-agents-guide.md` — how the agent reads files, prompting patterns, anti-patterns, file-count thresholds, token economy, Claude Code building blocks (skills/commands/subagents/hooks)
- `guides/testing-guide.md` — testing in SDD, built for agent-written tests: acceptance criteria as the test contract, the `TESTING.md` conventions file, TDD red→green with an agent, getting good tests out of it (and the break-the-code check so you don't trust green), pragmatic 1–10 coverage, anti-patterns
- `guides/runbook-operations-guide.md` — operational / runbook layer
- `guides/claude-md-guide.md` — how to write a good `CLAUDE.md` itself (what goes in, sizing, the many-docs case, anti-patterns, template)
- `guides/adr-guide.md` — full ADR how-to: Nygard format, MADR/Y-statements, four worked examples, Supersedes pattern, numbering, cross-referencing, anti-patterns, tooling, maintenance
- `guides/prd-guide.md` — PRD how-to: formats (PR-FAQ / lean / one-pager / full), anatomy, two worked-example PRDs (era 1 + era 2), era-boundary heuristics, AI-authoring prompts, review process, anti-patterns- `guides/quality-gates-guide.md` — enforcement + evaluation of SDD: three categories of checks (mechanical / LLM evaluator / human), five implementation patterns (pre-commit / Claude Code hooks / configured subagent / slash command / CI), what to mechanize vs leave human, worked example with full setup, anti-patterns
- `guides/spec-plan-tasks-guide.md` — **the flagship guide and centerpiece of the repo: the core spec → plan → tasks loop.** Three worked-example trios (a non-trivial feature as the three-file trio, a small feature as a one-file trio, and a discovery feature whose first spec was wrong), six AI-authoring prompts per artifact, iteration loops (incl. delivery vs discovery modes and the code→spec correction loop), cross-artifact consistency checks, slash-command sketches. Deliberately kept simple (no toolkit/scaffolder). Keep this the most polished guide.
- `guides/flow-guide.md` — the whole SDD flow as a runnable sequence: each step's command and output (idea → PRD → slice → trio → implement+test → ADR → merge), an entry-points table, and a cheat sheet. The spine; depth lives in the linked guides. Complements the "The whole flow" map section in the overview.
- `guides/sdd-in-teams-guide.md` — running SDD with 2–10 people, kept deliberately light: who owns what (one name per artifact), lightweight PR review, spec lifecycle, ADRs as a shared decision log, the solo case, onboarding, small-team failure modes, and what to add only when you outgrow ~10. No RACI / enterprise ceremony.
- `guides/legacy-to-sdd-migration-guide.md` — the one-time process of retrofitting SDD onto an existing codebase (audit, foundation, forward-only specs, reactive ADRs, agent prompts, worked examples)
- `guides/who-uses-sdd.md` — who uses SDD and what they call it: plain "who uses what" tables for AI-coding tools + pre-AI orgs, and the older ideas it borrows from

**Do NOT restructure `spec-driven-development-guide.md` without explicit instruction.** The owner will say when something should move out into a detail file. Until then, leave structural shape alone — content edits within sections are fine; pulling sections out into new files is not.

## Examples and naming conventions in the guides

When writing illustrative examples or sample content **in the guides themselves**:

- **Do not reuse names from the owner's real work.** Forbidden examples include (but are not limited to): `campaign`, `velobank`, `CHD`, `SKD`, `FLG`, `OneXmlOneFileStrategyMerge`, `CampaignLogRepository`, `FileImportTablesRepository`, `FlagFileWriter`, `inpost-shipx`, `gemini-image-gen`, `brokerage` (as a domain term), `T_CAMPAGINES_LOG`, `SCH_CAMPAGINES`. Use neutral placeholders: orders/HDR/DTL/RDY, `acme-bank`, `BatchXmlMerger`, `OrderLogRepository`, etc.
- **Database naming convention** (when illustrating schema/table examples):
  - Schemas and tables: lowercase snake_case (e.g. `app.order_log`, `legacy_import.records`)
  - No `T_` prefix on tables, no `SCH_` prefix on schemas, no `PK_T_` prefix on primary keys
  - Primary keys named `id`
  - Column names lowercase snake_case (`status`, `created_at`, `order_id`)
  - SQL keywords stay UPPERCASE (`SELECT`, `FROM`, `WHERE`, `ORDER BY`)
- **Tech stack examples** are fine as-is (.NET, Dapper, Quartz, Polly, Serilog, etc. are public ecosystem names, not owner-specific).

If the owner explicitly asks for a guide change that needs an example with one of the forbidden names — they'll say so. Default is generic.

## PDF generation

Build with pandoc + Prince. Default invocation:

```bash
pandoc guides/spec-driven-development-guide.md \
       guides/working-with-agents-guide.md \
       guides/runbook-operations-guide.md \
       -o pdf/claudecodeSDD-bundle.pdf \
       --pdf-engine=prince \
       -H pdf-style-wide.html \
       --metadata title="claudecodeSDD"
```

- **Output goes to the gitignored `pdf/` folder.** To regenerate a per-guide PDF for every guide, loop:
  ```bash
  mkdir -p pdf
  for f in guides/*.md; do
    pandoc "$f" -o "pdf/$(basename "$f" .md).pdf" --pdf-engine=prince \
      -H pdf-style-wide.html --metadata title="$(grep -m1 '^# ' "$f" | sed 's/^# *//')"
  done
  ```
  The bundle above is the 3-guide reading-flow PDF; the loop is the full per-guide set.

- Use `pdf-style-wide.html` — the **single unified style** for every PDF (per-guide builds and the bundle above). It keeps the small compact font but drops the side margins to `0.3cm` and overrides pandoc's centered-column cap (`body { max-width: none }`), so text runs full-width on A4 while page-number footers stay. Do **not** use `pdf-style.html` (full template) or `pdf-style-compact.html` (narrower 1cm-margin variant) unless explicitly asked.
- Do **NOT** pass `--toc`. Each guide already has its own Table of Contents section in markdown; an auto-TOC duplicates them.
- Guide order in the PDF: spec-driven-development → working-with-agents → runbook-operations (matches the README's "How to read" flow).
- The `unsupported properties: overflow-x` warning from Prince is harmless — Prince ignores web-only CSS.

## Working style

- When the owner says *"nie zmieniaj X"* / *"don't change X"*, treat that as a hard rule until they explicitly lift it.
- The owner writes detailed change requests when they want them. Don't expand scope on your own initiative beyond what's asked.
- For multi-step work (3+ distinct steps), use TaskCreate / TaskUpdate to track progress.
- The owner often writes in Polish but the guides are in English. Keep guides in English; replies can match the owner's language.

## When in doubt

- Editorial structure questions → ask before restructuring
- Example/name choice → default to generic; ask if a specific name is needed
- Convention conflict between this file and a guide → this file wins; flag the conflict so it can be resolved at the source
