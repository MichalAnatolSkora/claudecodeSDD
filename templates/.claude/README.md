# Trio commands (copy into your project's `.claude/`)

Copy-pasteable Claude Code slash commands for the spec → plan → tasks loop, plus `/prd-new` (idea → PRD draft) and `/features-from-prd` (slice a PRD into features) at the front, and `/implement` (work the tasks red→green) at the back. See [`guides/spec-plan-tasks-guide.md`](../../guides/spec-plan-tasks-guide.md) § "Slash commands worth having".

## How to use

Copy `commands/` into your project's `.claude/` directory:

```
your-project/.claude/
└── commands/
    ├── prd-new.md
    ├── features-from-prd.md
    ├── spec-new.md
    ├── spec-review.md
    ├── plan-from-spec.md
    ├── plan-validate.md
    ├── tasks-from-plan.md
    ├── tasks-add.md
    ├── trio-check.md
    └── implement.md
```

## What each does

You invoke each with `/name`:

| Command | Does |
|---------|------|
| `/prd-new <1–3 sentences>` | Sketch a lean PRD from a seed idea, then fill the gaps by Q&A |
| `/features-from-prd [path]` | Slice an accepted PRD into a prioritized, vertically-sliced feature list (read-only — no specs written) |
| `/spec-new <idea>` | Draft a `spec.md` from a one-paragraph idea |
| `/spec-review [path]` | Audit a spec for gaps (read-only) |
| `/plan-from-spec [path]` | Draft `plan.md` from a reviewed spec |
| `/plan-validate [path]` | Check the plan against ADRs + `ARCHITECTURE.md` (read-only) |
| `/tasks-from-plan [path]` | Draft an ordered, verifiable `tasks.md` from scratch |
| `/tasks-add <what> [path]` | Append/insert task(s) into an existing `tasks.md` (or a one-file trio's Tasks section) |
| `/trio-check [path]` | Final consistency audit across all three (read-only) |
| `/implement [path]` | Work `tasks.md` red→green, commit per task, then the break-the-code check |

These assume the standard SDD layout (`specs/`, `templates/`, `docs/adr/`, `ARCHITECTURE.md`, `CLAUDE.md`). Adjust the paths inside each file to match your project.
