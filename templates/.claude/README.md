# Trio commands & skills (copy into your project's `.claude/`)

Copy-pasteable Claude Code slash commands and skills for the spec → plan → tasks loop, as described in [`guides/spec-plan-tasks-guide.md`](../../guides/spec-plan-tasks-guide.md) § "Slash commands and skills worth having".

## How to use

Copy `commands/` and `skills/` into your project's `.claude/` directory:

```
your-project/.claude/
├── commands/
│   ├── spec-new.md
│   ├── spec-review.md
│   ├── plan-from-spec.md
│   ├── plan-validate.md
│   ├── tasks-from-plan.md
│   ├── tasks-add.md
│   └── trio-check.md
└── skills/
    ├── trio-author/SKILL.md
    └── trio-consistency/SKILL.md
```

## What each does

**Commands** — you invoke with `/name`:

| Command | Does |
|---------|------|
| `/spec-new <idea>` | Draft a `spec.md` from a one-paragraph idea |
| `/spec-review [path]` | Audit a spec for gaps (read-only) |
| `/plan-from-spec [path]` | Draft `plan.md` from a reviewed spec |
| `/plan-validate [path]` | Check the plan against ADRs + `ARCHITECTURE.md` (read-only) |
| `/tasks-from-plan [path]` | Draft an ordered, verifiable `tasks.md` from scratch |
| `/tasks-add <what> [path]` | Append/insert task(s) into an existing `tasks.md` (or a one-file trio's Tasks section) |
| `/trio-check [path]` | Final consistency audit across all three (read-only) |

**Skills** — the agent invokes them when relevant:

| Skill | Does |
|-------|------|
| `trio-author` | Drives spec → review → plan → review → tasks with human gates |
| `trio-consistency` | Auto-checks the trio agrees before implementation |

These assume the standard SDD layout (`specs/`, `templates/`, `docs/adr/`, `ARCHITECTURE.md`, `CLAUDE.md`). Adjust the paths inside each file to match your project.
