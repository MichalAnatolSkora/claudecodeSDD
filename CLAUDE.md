# CLAUDE.md

> Conventions and meta-rules for working on this repository (the SDD/Claude Code guides themselves). Read this before editing any guide.

## Repository purpose

Long-form guides + copy-pasteable templates for spec-driven development with AI coding agents. Structure:

```
.
├── README.md                          # entry point / map
├── CLAUDE.md                          # this file — conventions for editing
├── guides/                            # long-form methodology guides
│   ├── spec-driven-development-guide.md
│   ├── working-with-agents-guide.md
│   └── runbook-operations-guide.md
├── templates/                         # copy-pasteable starting points
├── pdf-style.html                     # full PDF style (not used by default)
├── pdf-style-compact.html             # compact PDF style — DEFAULT
└── output.pdf                         # built artifact
```

## Editorial structure — important

`guides/spec-driven-development-guide.md` is the **overview / entry-point** guide. Treat it as a high-level introduction that links out to detail files. Keep general principles and workflow framing here; push deep mechanics into separate detail guides.

When new content needs to land in the docs:

- **High-level principle, workflow framing, or repo-wide convention** → `spec-driven-development-guide.md`
- **Mechanics, prompts, or anti-patterns specific to one workflow** → its own detail guide, linked from the overview
- The overview's job is to point at the right detail file, not to absorb the details

Existing detail guides:

- `guides/working-with-agents-guide.md` — how the agent reads files, prompting patterns, anti-patterns, file-count thresholds, token economy, Claude Code building blocks (skills/commands/subagents/hooks)
- `guides/runbook-operations-guide.md` — operational / runbook layer
- `guides/claude-md-guide.md` — how to write a good `CLAUDE.md` itself (what goes in, sizing, the many-docs case, anti-patterns, template)
- `guides/legacy-to-sdd-migration-guide.md` — the one-time process of retrofitting SDD onto an existing codebase (audit, foundation, forward-only specs, reactive ADRs, agent prompts, worked examples)
- `guides/sdd-in-the-wild.md` — companies/teams publicly practicing SDD principles (verified adopters + adjacent methodologies)

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
       -o output.pdf \
       --pdf-engine=prince \
       -H pdf-style-compact.html \
       --metadata title="claudecodeSDD"
```

- Use `pdf-style-compact.html` (small font), not the full `pdf-style.html` template — the owner prefers compact.
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
