# Who uses spec-driven development (and what they call it)

> Real teams and tools that do the same thing: **write down what you're building, before you build it.** Some call it spec-driven development; most call it something else. Handy when someone asks *"is this just a Claude Code fad?"* — it isn't.

---

## The short version

"Spec-driven development" is a new name (2024–2025, out of the AI-coding-agent world) for an old habit: **decide what you're building and why, in writing, before you write the code.** AI agents made the habit pay off visibly — give the agent structured context up front and it stops guessing — but the habit itself is decades old.

So two groups show up below: the new AI-coding tools that bake it in, and the engineering orgs that have done it for years under other names.

This is a notes page, not a survey — adoption moves fast, so treat it as a starting point.

---

## AI-coding tools — who uses what

Every serious AI-coding tool landed on the same idea: **one file the agent always reads before it does anything.** That they all reinvented it separately is the strongest sign this isn't a fad.

| Tool / team | What they use | What it is |
|-------------|---------------|------------|
| **GitHub** (Microsoft) | [`spec-kit`](https://github.com/github/spec-kit) | A literal "Spec-Driven Development" toolkit: `spec.md → plan.md → tasks.md`, paired with Copilot. Shipped Aug 2025, tens of thousands of stars. The headline if you need to convince someone: *GitHub ships this themselves.* |
| **Anthropic** | Claude Code + `CLAUDE.md` | A repo-level instruction file the agent reads every session, plus "plan before code" prompting. Maps almost 1:1 to the workflow in this repo. |
| **Cursor** | `.cursorrules` | Same idea as `CLAUDE.md` — a conventions file the agent always reads. The community `awesome-cursorrules` catalog is a de-facto pattern library. |
| **Aider** | [conventions file](https://aider.chat/docs/usage/conventions.html) | Loaded into every session. |
| **Continue.dev** | `.continuerules` | Per-repo rules. |
| **Cline** | `.clinerules` | Per-repo rules. |

Different filenames, same move: **tell the agent your conventions instead of letting it guess.**

---

## Engineering orgs — who did it before AI

These teams wrote structured docs before code long before AI agents existed. SDD is partly the AI-era repackaging of the same instinct — useful references when you're pitching it to a team that hasn't adopted agents yet.

| Org | What they call it | What it is |
|-----|-------------------|------------|
| **Amazon** | Working Backwards / **PR-FAQ** | A new product starts as a fake press release + an FAQ, reviewed *before* any code. Plus the "6-page memo, read in silence, then discuss" meeting culture. (Book: *Working Backwards*, 2021.) |
| **Google** | **Design docs** + RFCs | Context, goals, non-goals, design, alternatives, rollout — a PRD + ADR + plan in one. (See Malte Ubl, [*Design docs at Google*](https://www.industrialempathy.com/posts/design-docs-at-google/).) |
| **Stripe** | **RFCs** | Long-form written proposals, treated with the same rigor as code. Decisions land in writing before code does, and the doc survives as memory. |
| **Basecamp / 37signals** | **Shape Up** | Work starts from a *pitch* — problem, appetite, solution sketch, rabbit holes, no-gos — which is basically a modern spec. ([Free book](https://basecamp.com/shapeup).) |
| **Microsoft** (broadly) | Design docs, post-mortems, ADRs | A long history across teams; GitHub (Microsoft-owned) productized it for the AI era with spec-kit. |

---

## Older ideas SDD borrows from

The pieces have a 10–50 year track record — handy when someone calls SDD a new fad:

- **IETF RFCs** (1969–) — the original "write the spec before the code." Every internet standard starts as one.
- **ADRs** — Architecture Decision Records, formalized by Michael Nygard (2011). Now standard tooling, and the basis for this repo's ADR layer.
- **C4 model** (Simon Brown) — structured architecture diagrams: Context → Container → Component → Code.
- **Diátaxis** (Daniele Procida) — splits docs into Tutorial / How-To / Reference / Explanation.
- **Pre-mortems** (Gary Klein) — write the failure story *before* launch, to surface risks early. Maps to a spec's Risks / Open questions.

---

## What this list doesn't tell you

- **Most companies do this privately.** The list leans toward orgs with public engineering blogs. Regulated industries (finance, health, defense) do SDD-like work out of necessity but rarely publish.
- **Depth varies.** "We have an ADR folder" is not "every change ships with a spec." The list mixes both ends.
- **The AI-era name is new.** Amazon, Google, Stripe, and Basecamp have done this for years without calling it SDD.
- **A `CLAUDE.md` in a repo proves little** on its own — its presence is a weak signal that real spec-driven work is happening.

Found a public example that belongs here? Add it — the landscape moves fast, so this page is best kept up collaboratively.

---

*Companion to [`spec-driven-development-guide.md`](spec-driven-development-guide.md) (the method) and [`working-with-agents-guide.md`](working-with-agents-guide.md) (the practice). It's here so you can check SDD's lineage and footprint without leaving the repo.*
