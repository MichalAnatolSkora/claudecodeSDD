# Working with AI Coding Agents

> How the agent actually reads your repo, why it sometimes misses things, how far you can push the document count before it breaks down — and the prompting patterns that keep it on track.

---

## Table of Contents

1. [Core Philosophy](#core-philosophy)
2. [How the Agent Reads Your Repo](#how-the-agent-reads-your-repo)
3. [The Agent's Toolset](#the-agents-toolset)
4. [When the Agent Loads a File — and Why It Sometimes Doesn't](#when-the-agent-loads-a-file--and-why-it-sometimes-doesnt)
5. [How Many Files Is Too Many](#how-many-files-is-too-many)
6. [Token Economy](#token-economy)
7. [Starting and Ending Sessions](#starting-and-ending-sessions)
8. [Working on Specs, ADRs, and Refactors](#working-on-specs-adrs-and-refactors)
9. [Working on Runbooks and Operational Tasks](#working-on-runbooks-and-operational-tasks)
10. [Universal Prompting Patterns](#universal-prompting-patterns)
11. [Claude Code Building Blocks](#claude-code-building-blocks)
12. [When the Agent Drifts](#when-the-agent-drifts)
13. [Maintaining Documentation Proactively](#maintaining-documentation-proactively)
14. [Anti-Patterns](#anti-patterns)
15. [Golden Rules](#golden-rules)

---

## Core Philosophy

### The mental model

The agent is not a colleague who has been on the team for two years. It is a *very fast* colleague who joined this morning, who reads only what you put in front of them, and who confidently fills any blank with a plausible-sounding guess. Your job is to leave fewer blanks.

That framing matters because the most common mistake — by experienced engineers, no less — is treating the agent like it shares your context. It doesn't. It hasn't read last month's Slack discussion. It doesn't know which library you abandoned and why. It hasn't met your domain expert. Every session starts at zero plus whatever you put in front of it in the prompt.

The flip side is the part most people undervalue: within whatever context you *do* give it, the agent is unusually good. It pattern-matches well, follows clear instructions reliably, writes idiomatic code in the style you point it at, and doesn't get tired. Working with an agent effectively is less about *prompting* and more about *briefing* — feeding that capability the right material instead of fighting it.

### Three insights everything else flows from

**1. Agents don't "discover" your codebase. They read what you point them at, or what they find via explicit search.**

A new human teammate would naturally browse — open a few files, click around the tree, read the README, ask someone. The agent doesn't have curiosity. If your prompt says *"fix the bug in the order export,"* the agent runs `grep` for *"order export,"* reads the matches, and tries to fix the bug. It won't read `ARCHITECTURE.md` unless you tell it to, the relevant ADR unless someone references it, or the GLOSSARY entry that would have explained why this particular workaround exists.

This is both a feature (focus, fewer wasted reads) and a bug (blindness to context you *assumed* it had). Pointers in `CLAUDE.md` and explicit references in prompts beat hoping it will figure things out on its own.

**2. More docs is not more context. Past a threshold, more docs cause *more* drift, not less.**

The intuitive model — *more documentation = more for the agent to read = better output* — is wrong past a surprisingly low threshold. Three things break down at once:

- **Attention dilutes.** Even when everything fits in the context window, an important convention on page 30 gets the same per-token attention as filler on page 3. The agent reads it but doesn't weigh it equally.
- **Search noise rises.** With four similarly-named files, the agent picks one — sometimes wrong, sometimes inconsistently across the same session.
- **Stale content drags.** A two-year-old ADR marked `Status: Superseded` gets read as authoritative because the supersede note is small and easy to miss.

The implication: curate, don't accumulate. A 5-file repo with sharp, fresh docs outperforms a 50-file repo where most docs are partially stale. (See [How Many Files Is Too Many](#how-many-files-is-too-many) for the thresholds and mitigations.)

**3. The agent's memory ends when the session ends.**

This is the insight new users miss most often, and the one that compounds the most damage. The agent has no persistent memory across sessions. If you spent an hour today teaching it that *"in this codebase, repositories handle SQL and services handle business logic, and never the other way around"* — that knowledge is gone tomorrow. The next session starts blank.

The implication: anything that must persist lives in **committed files**. `CLAUDE.md` is where conventions live. `docs/adr/` is where decisions live. `specs/` is where intent lives. *Anything in chat history is ephemeral and will be forgotten.* The corollary: don't waste a session teaching the agent something verbally without writing it down — you're paying tokens for knowledge that's scheduled to evaporate.

### The mindset shift

Everything else in this guide is downstream of those three insights and one underlying mindset shift: from *"the agent is my colleague"* to *"the agent is my colleague with amnesia, who needs a written briefing before each session."*

If you internalize only the briefing-document framing, you'll get most of the value of the rest of this guide for free.

---

## How the Agent Reads Your Repo

The agent has exactly four ways to find a file:

1. **You tell it.** *"Read CLAUDE.md and ARCHITECTURE.md before doing X."* Highest-signal path. The agent uses its `Read` tool with the path you supplied.
2. **It runs a search.** You say *"fix the bug in the order export flow."* The agent runs `grep` or a glob to find candidates and reads matches. Depends on file/symbol names being findable.
3. **It follows a reference.** It reads `spec.md`, sees *"see ADR-007"*, and reads ADR-007. Cross-references in your docs become a discovery graph the agent can walk.
4. **It follows a meta-rule.** Your `CLAUDE.md` says *"before generating SQL, read DOMAIN.md."* The agent obeys when the task triggers that rule.

There is no fifth path. The agent does **not** "scan the repo to understand the project" the way you would on day one. If you don't point at it, name it, link to it, or rule-mention it — the agent may never read it.

This is why `CLAUDE.md` matters disproportionately. It's the only file the agent reliably loads without being asked, and it's the only place where you can register meta-rules that fire automatically.

---

## The Agent's Toolset

The previous section described *what* the agent reads. This one describes *how* — the concrete tools it uses to read, search, run, write, and delegate. The examples are Claude Code's default toolset; Cursor, Aider, Continue.dev, and others ship analogous tools under different names, with the same shapes.

### How the agent learns its tools

The agent doesn't memorize a fixed toolset. At the start of each session, the harness sends it a list of available tools, each with a **name**, a **description**, a **parameter schema**, and (sometimes) **examples** of correct and incorrect use. The model reads those descriptions and decides — for each step of the task — which tool to reach for.

This has a consequence most users don't see: **a tool's description shapes its use as much as its capabilities do.** If the description says *"prefer `Read` over `Bash cat`"*, the agent prefers `Read`. If the description omits that hint, the agent defaults to whatever pattern its training reinforced (often `Bash`). Customizing tool descriptions — via MCP servers or harness settings — is one of the highest-leverage ways to steer agent behavior.

### The core toolset (Claude Code default)

Roughly grouped by purpose. Tool names are exact (case matters).

**File operations**

- **`Read`** — read a file at a known path. Returns numbered lines. Used when the agent already knows what file to look at; offset/limit supported for partial reads.
- **`Edit`** — exact string replacement in a file. Cheap (only the diff is sent) and the right default for any modification of an existing file. Requires the file to have been `Read` first in the session.
- **`Write`** — create a new file, or overwrite an existing one completely. Higher token cost than `Edit` (sends the whole file). Reserve for new files or full rewrites.
- **`NotebookEdit`** — edit Jupyter `.ipynb` cells specifically; the JSON structure of notebooks makes raw `Edit` brittle.
- **`Glob`** — find files by path pattern (`src/**/*.cs`, `docs/**/README.md`). Returns the list of matching paths.

**Search**

- **`Grep`** — search file contents for a pattern (regex or literal). Returns matching lines with paths. The right default for *"find where X is defined / referenced."*
- **`Glob`** — already mentioned; also a search tool when you want files by name pattern.
- **`WebSearch`** — search the public web. Returns titles, URLs, snippets. Used when the agent doesn't know which URL to fetch.

**Execution**

- **`Bash`** — run shell commands. Used for git operations, build/test commands, anything not covered by a dedicated tool. Supports `timeout` and `run_in_background`. Most powerful and most dangerous tool — restrict aggressively in `settings.json` for shared repos.

**Web**

- **`WebFetch`** — fetch a specific URL and return content (typically converted to markdown). Used when the URL is known. Authenticated URLs (Confluence, private GitHub, Google Docs) require an MCP equivalent — `WebFetch` falls back gracefully but won't see protected content.

**Delegation**

- **`Agent`** (a.k.a. **`Task`**) — spawn a subagent for a contained task. The subagent has its own context window; the main session sees only the final report. Two modes: ad-hoc (one-off prompt) or configured (named agent file in `.claude/agents/`). See [Claude Code Building Blocks § Subagents](#subagents).

**Task tracking**

- **`TaskCreate`** — start a new tracked task with a subject and description.
- **`TaskUpdate`** — change status (`pending` → `in_progress` → `completed`), update subject, set owner, add dependencies.
- **`TaskList`** — list all current tasks; useful to find what's pending or claim the next available.
- **`TaskGet`** — view full details of a single task.

Used for multi-step work (3+ distinct steps); skip for trivial single-step tasks.

**User interaction**

- **`AskUserQuestion`** — show 1–4 multi-choice questions with options to the user. Use when the agent genuinely needs a decision before continuing; don't use for trivial questions ("which file?") or to defer thinking.

**Scheduling**

- **`ScheduleWakeup`** — schedule the agent to wake up after a delay (used in self-paced loops).
- **`CronCreate` / `CronList` / `CronDelete`** — manage scheduled recurring agents.

**Skills**

- **`Skill`** — invoke a named skill from `.claude/skills/` (project) or `~/.claude/skills/` (user). The skill's instructions are loaded as the current prompt context. See [Claude Code Building Blocks § Skills](#skills).

**Plan mode**

- **`EnterPlanMode` / `ExitPlanMode`** — explicit modes for "plan-only" sessions where the agent drafts a plan without touching files, then exits the mode (committed or canceled) before doing anything destructive.

**MCP extensions (variable per setup)**

MCP (Model Context Protocol) servers add tools beyond the default set. The most common categories:

- **Browser tools** — `Claude in Chrome`, `Claude Preview`: navigate, click, screenshot, read DOM, fill forms. Used when the task involves a running web app.
- **Design / project-management tools** — Figma, Linear, Asana, Jira, Notion, Slack: read/write artifacts in those systems. The agent uses them when the task explicitly references one (`"add a Linear ticket for…"`).
- **Database tools** — Supabase, Snowflake, BigQuery MCPs: query the DB directly without going through code.
- **Repository / directory access** — additional codebases the agent can read alongside the active one.

MCP tools appear in the same toolset list as built-ins; the agent treats them identically. The user enables them per project in `.claude/settings.json` or `~/.claude/settings.json`.

### How the agent chooses between similar tools

Multiple tools often *could* satisfy a request. The agent's choice depends on what its tool descriptions tell it to prefer. The most common decisions:

**Reading file content: `Read` vs `Grep` vs `Glob` vs `Bash cat`**

- **Known path, full file** → `Read`
- **Known path, partial** → `Read` with `offset` + `limit`
- **Don't know path; know a symbol/string** → `Grep` first, then `Read` on matches
- **Don't know path; know name pattern** → `Glob` first, then `Read` on matches
- **`Bash cat`** → almost never (more expensive, no numbered lines, worse fallback). Most tool descriptions explicitly forbid it.

**Modifying a file: `Edit` vs `Write` vs `Bash sed`**

- **Small change to existing file** → `Edit` (cheap, surgical)
- **Multiple edits at once** → multiple `Edit` calls (or a tool like `MultiEdit` if available)
- **New file, or replacing the whole file** → `Write`
- **`Bash sed/awk`** → almost never. Same reasoning as `cat`.

**Running shell vs dedicated tool**

- **`ls`, `find`, `cat`, `head`, `tail`** → use `Read` / `Glob` / `Grep` instead
- **`grep`** → use `Grep` (faster, returns structured results)
- **`git`, `gh`, `npm`, `pytest`, deploy scripts, anything else** → `Bash`
- **`echo`** for communication → never. Output text directly in the response.

**Inline vs delegated: when to spawn a subagent**

- **Under ~5 tool calls expected** → inline
- **Long-running audit, cross-repo research, parallel investigation** → `Agent`
- **Repeated workflow with stable shape** → configured subagent in `.claude/agents/`
- **One-off exploration** → ad-hoc `Agent` call with the prompt embedded

**Searching vs fetching a URL**

- **Know the URL** → `WebFetch`
- **Don't know the URL** → `WebSearch` first, then `WebFetch` on the result
- **Authenticated URL** → check for an MCP-provided fetch tool before falling back to `WebFetch` (which can't see private content)

**Asking the user vs inferring**

- **Genuinely ambiguous** (multiple valid interpretations) → `AskUserQuestion`
- **Stalled on a single missing fact** → ask in plain prose, not via `AskUserQuestion`
- **The user *just* told you and you forgot** → re-read the prompt, don't re-ask

### The tool-use loop

Inside one of the agent's turns, the loop is:

1. **Read context.** System prompt + `CLAUDE.md` + the user's prompt + recent tool results.
2. **Plan.** Decide what tools to call and in what order. Multiple independent calls are batched; dependent ones are sequenced.
3. **Call.** Send tool calls. The harness runs them and returns results.
4. **Read results.** Each tool returns text (file contents, search hits, command output). Errors come back with messages.
5. **Decide.** Either (a) reply to the user with prose, (b) call more tools, or (c) both.
6. **Loop.** Steps 2–5 repeat until the agent decides the task is done or needs the user.

The agent doesn't see *every* tool result — long outputs are sometimes truncated or summarized before reaching the model. This is one reason a 2,000-line `find` output is wasteful: most of it never influences the agent's reasoning.

### Parallel vs sequential calls

Modern Claude Code (and similar tools) supports multiple tool calls in a single response. The agent batches calls that are *independent of each other*:

- **Parallel** — three `Read` calls for three different files; the order doesn't matter.
- **Sequential** — `Grep` for `processOrder` → read the file with the most hits → propose an edit. Each step needs the previous result.

Batching speeds up sessions significantly. A 10-call task done sequentially takes ~10× the round-trip time of the same task done with calls batched in groups of 3–4. The agent decides based on whether outputs depend on each other; you can nudge it with prompts like *"read X, Y, Z in parallel."*

### Permissions and gating

Each tool can be allowed, denied, or gated via `.claude/settings.json` (project) or `~/.claude/settings.json` (user). Common patterns:

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Grep",
      "Glob",
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(npm test:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(git push --force:*)",
      "Write(/etc/**)"
    ],
    "ask": [
      "Bash(deploy.sh:*)",
      "WebFetch"
    ]
  }
}
```

- **`allow`** — auto-execute, no prompt
- **`deny`** — hard-blocked; agent gets an error
- **`ask`** — prompt the user before executing

The patterns support glob-like wildcards on tool inputs. Granular gating (allow `Bash(git status:*)` but not `Bash(rm:*)`) is the right default for shared repos; blanket-allow of `Bash` is the most common mistake.

The agent sees its current allow/deny list as part of system context, so it knows in advance which tool calls will need permission. You can verify mid-session by asking: *"list the tools currently available to you."*

### MCP extensions

MCP servers extend the toolset for a specific project. Setup is one of:

- **Per project** — `.claude/settings.json` declares which MCP servers to load.
- **Per user** — `~/.claude/settings.json` for personal tools (e.g., your Linear account, your Figma access).
- **Built into the harness** — Claude Code ships some MCPs by default; others install via `npm` or a CLI command.

Typical install signals:

- Working on a web app → install a browser MCP (Claude in Chrome / Claude Preview)
- Doing UI design handoff → install Figma MCP
- Coordinating tickets → install Linear / Jira / Asana MCP
- Database-heavy work → install Supabase / Snowflake / BigQuery MCP

Don't install MCPs preemptively. Each adds tools to the agent's context list, taking tokens and adding decision overhead. Install reactively when a task genuinely needs the integration.

### When the agent picks the wrong tool

Common failures and how to steer:

**1. `Bash cat` instead of `Read`.** Older models default to shell habits. Steer via `CLAUDE.md`: *"Always prefer `Read` over `Bash cat` / `head` / `tail`."*

**2. `Write` for a one-line change.** The agent regenerates the whole file when an `Edit` would do. Steer: *"Show the change as a diff via `Edit`; do not regenerate the file."*

**3. Spawning no subagent when one was warranted.** The agent reads ten files inline that a subagent should have summarized. See [When the agent skips spawning a subagent](#when-the-agent-skips-spawning-a-subagent) for the predictable reasons and nudges.

**4. Running tests inline when the user wanted a plan.** *"Don't run tests; outline what you'd do first."* Or use Plan Mode.

**5. Excessive `Bash` for filesystem ops.** Find with `find`, list with `ls`, read with `cat` — when `Glob`, the harness-listing tool, and `Read` are cheaper. Steer in `CLAUDE.md`.

**6. `WebFetch` to an authenticated URL.** Returns a marketing page or a login wall, not the content. If you have an MCP-provided authenticated tool (`mcp__atlassian__fetch_page`, etc.), instruct the agent to use it explicitly.

**7. `AskUserQuestion` when the agent should just decide.** Excessive prompting fragments the conversation. *"Don't ask me which file; pick the most likely one and tell me what you chose."*

The general principle: the agent picks tools based on tool descriptions and prompt hints. If it consistently picks wrong, the fix is usually a line in `CLAUDE.md` (*"always prefer X over Y"*) rather than re-prompting in every session.

---

## When the Agent Loads a File — and Why It Sometimes Doesn't

A common question: *is the agent's file loading random?*

**No, but the feel of "randomness" is real.** The agent makes file-loading decisions based on:

- **Your prompt's specificity.** A vague prompt leads to broader, lower-confidence searches; the agent may read 4 files when 1 was needed, or read 0 when 1 was needed.
- **Available tools.** `Read` for known paths, `Grep` for keyword search, glob for pattern matching. The toolset determines what's reachable.
- **Token-budget pressure.** Long sessions make the agent stingier with file reads; it starts guessing instead of reading.
- **Patterns from training.** Files named `README.md`, `CLAUDE.md`, `package.json`, `Cargo.toml` get prioritized because the model has seen them millions of times.
- **The agent's judgment about likely relevance.** This is the squishy part — and the part that *looks* random from the outside.

### Why it feels random

1. LLM judgment is fuzzy. The same prompt across two runs can produce different file-read sequences.
2. The agent doesn't always show its full reasoning. A missed file looks like a coin flip.
3. Multiple files with similar names (`order-retry.md`, `order_retry.md`, `legacy-order-retry.md`) — the agent picks one, sometimes not the one you meant.
4. Some loads are "lazy" — the agent might skip reading a referenced file if it thinks it can answer without it.

### What you can do about it

- **Be explicit.** *"Read `CLAUDE.md`, `ARCHITECTURE.md § 'Order export'`, and the most recent spec in `specs/`"* beats *"look at the relevant docs."*
- **Stable, distinctive filenames.** A file the agent will be asked to read a hundred times deserves a name it can't confuse with another. No `notes.md`, `temp.md`, `v2.md`.
- **Hub-and-spoke from `CLAUDE.md`.** Put meta-rules ("for X, read Y") in one place the agent always loads. Then prompts can stay short — `CLAUDE.md` already encodes which docs apply when.
- **Verify what was read.** Mid-task, ask: *"list the files you've read so far in this task and why."* It will tell you. If a critical file is missing, point at it before continuing.
- **Don't rely on inference.** *"You should know this from the codebase"* is wishful thinking. If it matters, name it.

---

## How Many Files Is Too Many

The other common question: *if I keep adding markdown files, will the agent eventually get lost and stop being able to work with the repo?*

**Yes — but later than people fear, and only if certain disciplines slip.**

The failure mode is rarely "context window exhausted" — that's hard to hit on a modern model. The failure modes are subtler:

### Five ways too many files break the agent

1. **Attention dilution.** Even when everything fits in the context window, more content means less attention per chunk. The agent reads it but doesn't *weigh* it equally. Important conventions on page 30 get less attention than ones on page 3.

2. **Search noise.** When you have `order-retry.md`, `orders-retry.md`, `retry-orders.md`, and `legacy-order-retry.md`, the agent's keyword search returns four results and picks one — sometimes not the one you mean. The agent then writes confidently from the wrong source.

3. **Discovery cost.** With 100 markdown files in `docs/`, the agent burns tool calls (each one eats tokens) before it finds the right doc. Sometimes it gives up and guesses. Sometimes it reads three near-matches and conflates them.

4. **Stale-content drag.** A two-year-old ADR with `Status: Superseded` that the agent reads as authoritative because the supersede note is small and easy to miss. Old runbooks with wrong commands. Old specs whose decisions have been reversed.

5. **Conflicting authorities.** Two files say different things about the same convention. `ARCHITECTURE.md` says *"use Dapper"*, an old how-to says *"use EF"*. The agent reads both and picks one — sometimes wrong, sometimes inconsistently across the same session.

### Rough thresholds

These are squishy — your mileage depends on file sizes, naming clarity, and how you prompt.

| Doc count | Behavior | What you need |
|-----------|----------|---------------|
| < 10 | Agent scans the repo, reads what looks relevant, mostly OK | Decent file names |
| 10–30 | Agent needs a guide to where things live | `CLAUDE.md` as a hub |
| 30–100 | Agent gets lost without explicit pointers | `CLAUDE.md` hub + folder-level `README.md` index files |
| 100+ | Agent can't browse — you must hand-select context per task | Strict folder structure + explicit context selection in every non-trivial prompt |

### Signs you've crossed the threshold

- The agent generates code that contradicts a doc you *just* wrote → it didn't read it.
- The agent re-asks for context you already documented → it didn't find it.
- The same convention is violated repeatedly across sessions → no clear "authoritative" pointer exists.
- The agent picks an archived or superseded file as authoritative → no "currently active" signal distinguishes it from the live doc.

### Mitigations, in order of effectiveness

1. **`CLAUDE.md` as the explicit hub.** It points at every other doc the agent should know about, organized by task type. *"For SQL, read `DOMAIN.md`. For new endpoints, read `ARCHITECTURE.md § API`. For deployment, read `OPERATIONS.md`."* This single file replaces fifty acts of file discovery.

2. **Stable, distinctive file names.** No `notes.md`, `temp.md`, `draft.md`, `v2.md`, `final.md`. Each name should say what's inside; each name should be unique enough that keyword search returns the right one.

3. **One `README.md` per non-trivial folder.** It's an index. `docs/runbooks/README.md` lists every runbook with a one-line description. `docs/adr/README.md` lists ADRs by status. The agent reads the index instead of guessing filenames.

4. **Archive aggressively.** Move dead content to `docs/_archived/` or similar. The agent doesn't search there by default (and you can tell `CLAUDE.md` to skip it explicitly).

5. **Per-task context selection.** Don't load everything. For a feature, load: `CLAUDE.md` + one or two relevant docs + one or two example specs. The rest is noise.

6. **Periodic audit.** Quarterly, ask the agent: *"list every markdown file in this repo. For each, identify: last modified date, whether it's still authoritative, whether it conflicts with any other file."* Then prune.

The disciplines compound. With a 30-file repo and good hub-and-spoke organization, the agent works as well as it does on a 10-file repo. With a 30-file repo and no `CLAUDE.md` hub, it works worse than on a 5-file repo.

---

## Token Economy

Tokens are the agent's currency. Every file read, every prompt, every tool output spends them. Sessions get slower and more expensive as the budget burns down, and beyond a threshold the agent's attention starts to dilute even when budget remains.

The goal isn't to *minimize* tokens — it's to spend them where they earn back the most. Most teams over-pay in three or four predictable places.

### What a token is, and how they're counted

A **token** is the unit the model processes. It's not a character, not a byte, not a word — it's a sub-word piece chosen by the model's tokenizer. *"Hello"* is usually one token; *"tokenization"* is often two or three (e.g. `token` + `ization`). Whitespace and punctuation count too.

**Rough ratios you can rely on (English):**

| Unit | Approximate token count |
|------|------------------------|
| 1 token | ~4 characters |
| 1 token | ~0.75 words |
| 100 tokens | ~75 words ≈ 400 characters ≈ 1 short paragraph |
| 1,000 tokens | ~750 words ≈ 1.5 pages of prose |
| 10,000 tokens | ~7,500 words ≈ 15 pages, or a small source file |
| 100,000 tokens | ~75,000 words ≈ a short novel, or a large module |

**Code, JSON, and structured text** tokenize less efficiently than prose — roughly 3 to 4 characters per token, sometimes lower if the language is punctuation-heavy. **CJK and other non-Latin scripts** are often closer to 1 character per token.

Practical feel for SDD artifacts:

| Artifact | Typical token count |
|----------|---------------------|
| A 150-line `CLAUDE.md` | ~2,000–4,000 tokens |
| A 400-line `ARCHITECTURE.md` | ~6,000–10,000 tokens |
| A 500-line source file | ~5,000–10,000 tokens (more if punctuation-heavy) |
| A typical user prompt | ~50–500 tokens |
| The system prompt + tool descriptions in Claude Code | ~5,000–15,000 tokens (you don't control this; it's part of the harness) |
| A whole mid-sized codebase | 500,000+ tokens (won't fit; that's why selective context matters) |

#### Four kinds of tokens to know about

Tokens are charged differently depending on what kind they are:

1. **Input tokens** — everything sent *to* the model: system prompt, tool descriptions, `CLAUDE.md`, conversation history, your prompt, and the results of any tool calls so far in the turn.
2. **Output tokens** — everything the model writes *back*: prose responses plus the JSON of any tool calls. Output is more expensive than input (typically ~5× the input rate, model-dependent).
3. **Cached input tokens** — input tokens served from a prompt cache, when the conversation prefix matches a recent cache. Roughly **10% of the cost** of an uncached input token. See [Prompt caching](#prompt-caching) below.
4. **Reasoning tokens** (extended thinking) — internal "thinking" tokens generated before the visible response when extended-thinking mode is enabled. These count separately and are billed at the output rate. Often 2–10× the visible output for hard tasks.

The bill at the end of a session is: `input × rate + output × rate + reasoning × rate − (cached × 90% discount)`. In Claude Code, the harness handles most of this transparently; you can see it summarized via `/cost`-style commands or by asking the agent.

#### What "context window" means in tokens

The **context window** is the total token budget for *one model call* — input plus output combined. Current Claude models give you about **200,000 tokens** per call (a few support more in beta tiers). Once you approach the limit, the harness either truncates older messages, summarizes the conversation, or refuses to proceed.

Two things to know:

- **The hard limit is rarely the binding constraint.** Models start losing attention well before they run out of window. Dropping from 200k filled to 60k filled often improves output quality even though there was "room."
- **Each prompt replays the full history.** Long sessions don't just *sit* at 80k tokens — every new prompt re-sends those 80k as input. Token cost grows superlinearly with session length, which is why ending sessions early (see "End sessions early" above) is one of the highest-leverage saves.

#### Where input tokens come from in a typical session

Roughly, on a fresh turn:

```
System prompt                          ~3,000–5,000 tokens   (harness-defined; you don't control this)
Tool descriptions                      ~5,000–10,000 tokens  (one per available tool)
CLAUDE.md (auto-loaded)                ~2,000–5,000 tokens
Conversation history so far            varies — grows each turn
Your current prompt                    ~50–500 tokens
Tool results from this turn            varies — sometimes huge
---------------------------------
                              Total input for one call
```

Output tokens are usually a small fraction of input — most code-assistant work is read-heavy. The exception is generating large files (a 500-line code block can cost 5k+ output tokens on its own).

#### How to inspect token usage

Three ways, in order of cheapness:

1. **Harness-provided commands** — `/cost`, `/usage`, or equivalent. Claude Code surfaces input/output/cache totals per session. Look here first.
2. **Ask the agent** mid-session — *"how many tokens have we used in this task, and roughly where did they go? Sort largest first."* The agent gives an estimate based on what it can see.
3. **API logs** (for users hitting the API directly) — every response includes `usage.input_tokens`, `usage.output_tokens`, `usage.cache_creation_input_tokens`, `usage.cache_read_input_tokens`. Aggregate over a session to see the real shape.

The biggest items in a session are almost always:

- One or two oversized file reads
- Long unfiltered tool outputs (`find`, `git log`, big test runs)
- Replayed conversation history on every prompt

### Where tokens get wasted

1. **Re-reading the same files.** A long session that reads `CLAUDE.md` four times is doing the work three times for free (caching helps when the prefix is stable — see below).
2. **Loading entire files when you need one section.** *"Read `OrderRepository.cs`"* when you actually want the `GetOrders` method costs 5,000 tokens to find the 50 you wanted.
3. **Full-file rewrites instead of diffs.** Generating a 400-line file because you changed 3 lines is the single most common token sink. Diffs cost ~50× less and review faster.
4. **Verbose ceremony in prompts.** Twelve-line preambles (*"please, kindly, if you would…"*) are pure overhead. The agent doesn't reward politeness with better output.
5. **Long, unfiltered tool outputs.** `find` returning 800 files; `git log` with 2,000 commits. The agent reads them all, gets less useful per token, and burns budget on noise.
6. **Re-explaining context each session.** No `CLAUDE.md` → every session pays the *"what is this project"* tax from scratch.
7. **Iterating by regeneration instead of targeted edit.** *"Give me the whole file again with this one change"* instead of *"show me a diff for just this method."*

### Practical patterns that save tokens

**Be specific about what to read.** Point at file paths and ranges when you can:

```
Read the GetOrders method in src/Repositories/OrderRepository.cs.
Don't read the rest of the file.
```

beats

```
Read the order repository and find the relevant method.
```

The first costs a few hundred tokens. The second can easily cost 5,000+ on a large file.

**Use `grep` / `glob` before `read`.** Find first, then load only the matches:

```
Grep src/ for `processBatch`. From the matches, read only the file
with the most occurrences. Don't load the others.
```

**Ask for diffs, not rewrites.** Any prompt that involves changing code should specify:

```
Show me the change as a diff against the current file.
Don't reproduce unchanged lines.
```

For environments that don't render diffs natively, ask for the new code *with a few lines of context* on either side — not the full file.

**Front-load multi-part prompts.** One prompt with five questions is far cheaper than five prompts with one question each — every prompt re-replays system context, tool descriptions, and conversation history.

**Compact `CLAUDE.md`.** Target ~100–250 lines. A 2,000-line `CLAUDE.md` is loaded into every prompt and dilutes attention. The biggest improvement is usually *trimming*, not adding.

**End sessions early.** A 10-prompt session uses less than a 100-prompt session for the same work — each new prompt replays the full history. Save a session summary, start fresh, paste the summary as context.

**Filter tool output before the agent sees it.** Instead of:

```bash
find . -name "*.cs"
```

(which might return 2,000 lines), prefer:

```bash
find . -name "*Order*.cs" -not -path "*/bin/*" -not -path "*/obj/*"
```

The shell did the filtering; the agent didn't have to. You save the difference.

### Prompt caching

Long sessions can become **dramatically cheaper** as they go — but only if the conversation prefix stays stable. This is one of the highest-leverage optimizations in the entire Token Economy section, and it's almost free if you understand how it works.

#### What gets cached

The model caches **the prefix** of your conversation — everything from the start up to a marked cache point. On the next request, if the prefix matches an existing cache, the model reads it from cache (cheap) and only processes the *new* suffix from scratch.

In practice, the cacheable prefix is usually:

```
[System prompt] + [Tool descriptions] + [CLAUDE.md] + [stable docs you loaded first] + [conversation history]
                                                                                            ↑
                                                                              everything up to this point can hit cache
```

In Claude Code, **you don't set cache breakpoints manually** — the harness adds them automatically at sensible points. Direct-API users place `cache_control` markers explicitly.

#### Pricing (Anthropic, current model family)

The exact numbers shift over time, but the ratios are stable:

| Token kind | Cost relative to standard input |
|------------|--------------------------------|
| Standard input (uncached) | **1.00×** (baseline) |
| Cache write (first time, 5-minute TTL) | ~**1.25×** — small premium to seed the cache |
| Cache hit (subsequent reads, default TTL) | ~**0.10×** — **90% savings** |
| Cache write (1-hour TTL, beta tier) | ~**2.00×** — bigger seed cost, much longer reuse |
| Output tokens | ~**5.00×** (model-dependent) |

A long session with a stable 10k-token prefix saves roughly: `10,000 × (1.00 − 0.10) = 9,000 tokens per prompt` after the first. Across 20 prompts in a session, that's 180,000 cached tokens — about the same as eliminating two large file reads.

#### TTL — how long the cache lives

- **Default**: 5 minutes from last access. Every cache *hit* resets the timer, so an active session keeps the cache warm.
- **Long pauses kill it.** Step away for lunch, come back, the cache is gone; next prompt pays full input rate to re-warm.
- **1-hour beta tier** is available on some models for an extra cache-write premium — useful for batch workflows or long-running review sessions.

#### Minimum cacheable size

A prefix must be **at least ~1,024 tokens** to qualify for caching (~2,048 for smaller / faster models like Haiku). Prompts below the threshold are processed fresh every time.

In practice this means: tiny `CLAUDE.md` files (under ~50 lines) may not cache; the threshold is most reliably crossed by *system prompt + tool descriptions + CLAUDE.md combined*, which usually totals well above the limit. You don't have to engineer for this — just know the threshold exists.

#### What invalidates the cache

Each of these makes subsequent reads pay full input rate:

- **Any change before the cache point.** Edit a single word in `CLAUDE.md` mid-session → the cache for that session is gone; the next prompt re-warms it.
- **Reordering reads.** If session A reads `A.md` then `B.md`, and session B reads `B.md` then `A.md`, the cache won't transfer. Order matters.
- **TTL expiry.** No activity for 5+ minutes → cache evicted.
- **Different system prompt or tool list.** Most users don't change these mid-session, but if you switch projects, the new system context is a different cache.

#### How to maximize cache benefit

Three habits cover ~90% of the savings:

1. **Front-load stable docs at session start.** Read `CLAUDE.md` + `ARCHITECTURE.md` + the relevant ADR(s) *first*, in the same order every time. After the second prompt of the session, the cache is paying for itself.
2. **Don't shuffle the early reads across sessions.** Even small reorderings break the prefix match. If you have a session-start ritual (or a `/start-session` slash command), make it deterministic.
3. **Don't edit `CLAUDE.md` mid-session unless you need to.** Save edits for the end of a session, or for a brand-new session that needs them.

#### Multiple cache breakpoints (advanced)

Anthropic's API supports multiple `cache_control` markers per request — typically four. The model checks the *longest* matching prefix. Useful structuring:

```
[Layer 1: system prompt + tool descriptions]  — cache_control: stable
[Layer 2: project docs (CLAUDE.md, ARCHITECTURE.md)]  — cache_control: stable
[Layer 3: feature-specific context (active spec, current code)]  — cache_control: stable
[Layer 4: this turn's prompt + tool results]  — no cache (fresh each turn)
```

Each layer can hit cache independently. A session that switches features can keep layers 1–2 cached while layer 3 invalidates and re-warms.

In Claude Code the layering is automatic; on the API you set breakpoints by hand.

#### When caching doesn't help

- **One-shot prompts.** No second prompt to amortize the cache-write premium.
- **Highly variable conversation shape.** Reordering, frequent edits to early context — each session pays full rate.
- **Sub-threshold prompts** (<1,024 tokens cumulative early context). No cache eligibility.
- **Very short sessions** (< 5 prompts) where the write cost exceeds the savings.

For a 3-prompt session, caching is roughly break-even. For a 30-prompt session with stable context, it's a 50–70% saving on input tokens.

In practice: long sessions with consistent context get *cheaper* per prompt as more of the prefix is cached. This rewards session shapes that front-load context once and then iterate.

### Token-burning anti-patterns

1. **"Read all files in `src/` and summarize."** Agent reads 200 files, summary is generic, you've burned 50k tokens. Pick 5–10 files instead.
2. **"Make this production-ready."** Open-ended demands trigger ceremony — lots of generated code *just in case*. Be specific: *"add try/except around the network call, log on failure, return 500."*
3. **Loading full git diffs to find one bug.** `git log --oneline | head -20` first, then read the specific commit.
4. **Asking the agent to "remember" by re-pasting context every prompt.** Use a session summary file instead — paste once, reference by path.
5. **Letting the agent paste back the file you just sent.** Specify *"don't reproduce the input"* if you're worried it will.
6. **One-prompt-per-question over a 30-minute session** when a single front-loaded prompt would do.
7. **Asking for the same analysis twice.** Save the first answer somewhere; reference it instead of regenerating.

### Quick token audit

At any point in a session, ask:

```
How many tool calls have you made so far in this task, and approximately
how many tokens did each consume? Sort by cost descending.
```

The agent gives a rough breakdown. The biggest items are almost always:

- One or two large file reads
- Long tool outputs (find/grep without filters, full git diffs)
- Re-reads of the same content

That tells you where to cut.

### When NOT to optimize

Token-saving has a cost too — time spent phrasing prompts carefully, the friction of `grep`-before-`read`, the discipline of focused prompts. Skip the optimization when:

- The task is short (3 prompts or fewer)
- You're exploring and don't yet know what to look at
- You're on a flat-rate tier and your time matters more than per-token cost
- The first instinct is the right one and over-engineering the prompt slows you down

Token economy matters at scale and over long sessions. For one-shot prompts, just type.

---

## Starting and Ending Sessions

All the documentation in the world is useless if you don't actually use it when prompting the agent. The next sections cover concrete prompts for the recurring situations.

### Starting a New Session

Begin every non-trivial session by anchoring the agent in the relevant context. Don't assume it remembers anything from before — even in long-running tools like Claude Code, fresh context windows are the norm.

**For a new feature:**

```
Read CLAUDE.md, ARCHITECTURE.md, and DOMAIN.md.
Then read specs/_template/spec.md to understand our spec format.
We're starting a new feature: [short description].
Before writing any code, draft specs/[date-slug]/spec.md based on the template.
List open questions at the end — don't fill them in yourself.
```

**For modifying an existing feature:**

```
Read CLAUDE.md and ARCHITECTURE.md.
We're modifying [feature]. Read its original spec at specs/[folder]/spec.md
and the relevant code at [paths].
I want to: [change description].
Before changing anything, summarize:
1. Which files will need to change
2. Which conventions or ADRs apply
3. What you would do — wait for my approval before writing code.
```

The pattern is the same in both: **load context → state intent → propose plan → wait for green light**. Don't let the agent jump straight to code.

### Ending a Session

The last 60 seconds of a session matter more than the first 10 minutes — they decide whether next session has to start from scratch.

```
We're wrapping up. Please produce a session summary:
1. What we accomplished (with PR/spec references)
2. What's left in current tasks.md (with status)
3. Any decisions made that aren't yet in ADRs or specs
4. Any documentation drift you noticed and didn't address
5. Suggested next session starting prompt.

Save it as specs/[current-spec]/session-notes-[date].md
```

This becomes the perfect starter context for your next session — by you or by the agent.

---

## Working on Specs, ADRs, and Refactors

### After Adding a New ADR

When you add a new ADR (especially `Accepted` or one that `Supersedes` another), the agent doesn't magically know. Tell it explicitly. (For the mechanics of *how to write* the ADR itself — format, lifecycle, anti-patterns, worked examples — see [`adr-guide.md`](adr-guide.md). This section assumes the ADR is already written; it covers the post-write prompts.)

**For a new Accepted ADR:**

```
I just added docs/adr/ADR-014-[slug].md.
Read it carefully. Then:
1. Update the "Active decisions" list in CLAUDE.md (add ADR-014)
2. Check if any other docs reference the old approach and need updating
3. Tell me what code in the current codebase contradicts this ADR
   — do NOT change code yet, just list the locations.
```

**For an ADR that Supersedes another:**

```
ADR-014 supersedes ADR-002. I've already updated ADR-002's status header.
Please:
1. Update CLAUDE.md's "Active decisions" list — remove ADR-002, add ADR-014
2. Grep the codebase and specs/ for references to ADR-002
3. For each reference, recommend whether it stays (historical context)
   or needs updating to point to ADR-014.
```

**For documenting a decision after the fact:**

```
We just decided to use [X] instead of [Y] for [reason].
Draft an ADR following the template in docs/adr/_template.md.
Use the next available number. Status: Accepted, today's date.
Fill in Context, Decision, Consequences, and Alternatives Rejected.
Leave References empty — I'll add them.
Show me the draft before creating the file.
```

### After Creating a Spec — Starting Implementation

A spec is ready. Now you want the agent to implement it. Don't just say *"code it up."*

```
We're implementing specs/[date-slug]/.
Read spec.md, plan.md, and tasks.md in that folder.
Also read CLAUDE.md and the referenced source files in plan.md.

Work task by task from tasks.md:
1. Before each task, restate what you're about to do
2. Write the test first (if applicable per TESTING.md)
3. Implement
4. Wait for me to confirm before moving to the next task

Do NOT skip ahead. Do NOT add features not in the spec.
If something in the spec is unclear, stop and ask — don't guess.
```

For smaller specs you can compress this, but the principle holds: **task by task, with checkpoints**.

### After Implementing — Updating Documentation

This is the step most teams skip, and it's why repos rot. After merging code, the agent should help maintain the docs that just became outdated.

```
We just merged PR #123 implementing specs/[date-slug]/.
Please:
1. Append "STATUS: shipped (PR #123, [date])" to spec.md
2. Check if ARCHITECTURE.md needs updating (new component, changed boundary?)
3. Check if DOMAIN.md needs updating (new term introduced?)
4. Check if CLAUDE.md conventions section needs a line about [new pattern]
5. Show me proposed diffs — I'll approve each one separately.
```

This 5-minute habit prevents documentation drift over months.

### After a Major Refactor

Refactors invalidate large parts of the agent's mental model. Help it re-orient.

```
We just completed a major refactor of [subsystem].
Before we continue work:
1. Read the current state of [paths]
2. Compare against ARCHITECTURE.md — list discrepancies
3. Propose updates to ARCHITECTURE.md to reflect reality
4. Create docs/snapshots/[date]-post-refactor.md describing the new state
   (for future reference and rollback context).
```

Snapshots are cheap insurance. You almost never need them, but when you do they're priceless.

---

## Working on Runbooks and Operational Tasks

The runbook is a different beast from specs and ADRs — it's read under stress, with the agent often acting as a co-pilot rather than a code generator. The prompts reflect that.

### Drafting a new entry after an incident

```
We just resolved an incident: [one-line summary].
Root cause was [X]. Recovery steps were [Y, Z].

Draft a new runbook entry at docs/runbooks/[slug].md using the template at
docs/runbooks/_template.md.

Fill in:
- Symptoms (what the on-call would see at the start)
- Pre-checks, Diagnosis, Recovery
- Verification
- Escalation
- Related entries (search existing runbooks for any that should link)

Mark "Last verified: today's date" and "Owner: [team]".
Show me the draft before saving.
```

### Extracting a runbook from a post-mortem

```
Read docs/postmortems/[file].md.

Identify any *operational* lessons that should become runbook entries.
For each:
1. Propose a runbook filename
2. Propose a 5-line outline (symptoms → recovery)
3. Mark whether it's a NEW entry or an EXTENSION of an existing one

List them. I'll pick which to create.
```

### Updating after an infrastructure change

```
We just changed [X]: [description]. Files affected: [paths].

Search OPERATIONS.md and docs/runbooks/ for any:
1. Commands that reference the old name/path/port
2. Procedures that depend on the old behavior
3. Health checks or dashboards that need updating

List the locations and the proposed update for each — do NOT change docs yet.
```

### Generating a diagnostic script from runbook content

```
Read docs/runbooks/[slug].md.

Generate a single bash script that runs all the diagnostic steps from the
"Diagnosis" section, prints output for each, and exits with:
- 0 if all checks pass
- 1 if any check fails

Use exactly the commands and expected outputs in the runbook — do NOT invent.
If something in the runbook is unclear, list it as a question instead of guessing.
```

This pattern — *runbook is the spec, script is the artifact* — is much more reliable than asking the agent to invent diagnostics from scratch.

### Auditing the runbook for staleness

```
Read docs/runbooks/ and OPERATIONS.md.
For each entry, report:
1. Last verified date (or "not present")
2. Whether any referenced hostnames, file paths, or service names
   appear missing from the current codebase (check src/, config/, infra/)
3. Whether any referenced ADRs are now Superseded

Output a markdown table. Do not modify any files.
```

Run this quarterly. It surfaces rot before it bites.

### When the agent generates "production-ready" code without a runbook

If the agent proposes adding a feature with non-trivial operational concerns (new external integration, new background job, new persistent state), require:

```
Before this is "done," propose the runbook entries that need to exist for this
feature to be operable:
- What can break?
- What's the recovery procedure?
- What's the health check?
- What's the escalation path?

List entries to add to docs/runbooks/. We'll write them before merging.
```

This is the operational equivalent of *"tests must accompany the feature."* A new background job without a runbook entry is a 3 a.m. waiting to happen.

---

## Universal Prompting Patterns

These work across all situations, regardless of whether you're writing a spec, drafting a runbook, or fixing a bug.

**"Plan before code":**
```
Don't write code yet. First, outline what you'll do in 5-10 bullet points.
I'll approve or correct before you start implementing.
```

**"Cite your source":**
```
For each decision in your proposal, cite the document that supports it
(CLAUDE.md section, ADR number, spec path).
If a decision isn't grounded in any document, mark it as [ASSUMPTION]
and we'll discuss before proceeding.
```

**"Diff, don't replace":**
```
Show me proposed changes as diffs against the current file,
not as a full rewrite. I want to see exactly what changes.
```

**"Question before assumption":**
```
If anything in this task is ambiguous, list your questions before starting.
I'd rather answer 5 questions now than refactor 50 lines later.
```

**"Stay in scope":**
```
The spec is specs/[date-slug]/. Do not modify any file outside the paths
listed in plan.md. If you think a change outside scope is necessary,
stop and propose extending the spec — don't silently expand.
```

**"List what you read":**
```
Before continuing, list every file you've read in this task and one sentence
on why. If any of them are stale or wrong-source, tell me before you proceed.
```

The last one is underrated — it surfaces silent assumptions early.

---

## Claude Code Building Blocks

Plain prompting takes you a long way, but Claude Code (and similar agent tools) ship four building blocks that let you encode recurring workflows so you don't retype them every session: **skills**, **slash commands**, **subagents**, and **hooks**. Each one fits naturally into an SDD repo — and each has a trap you'll only see after using it for a week.

This section explains what each is, where it earns its place in your SDD workflow, an example, and the anti-pattern that comes with it. The decision table at the end is the *"which do I reach for?"* summary.

### Skills

**What they are:** self-contained, named capabilities Claude can discover and invoke. A skill lives in `.claude/skills/<name>/SKILL.md` (project-scoped) or `~/.claude/skills/<name>/SKILL.md` (user-scoped). The SKILL.md frontmatter describes *when* the skill applies; Claude auto-suggests or invokes it when a prompt matches.

**Where they fit in SDD:**

- Workflows you do often enough to name. *"Convert markdown to PDF"*, *"draft a new ADR from a decision summary"*, *"audit doc staleness across the repo"*.
- Procedures with multi-step instructions that you'd otherwise paste into a prompt every time.
- Repo-specific automation that benefits new contributors — the skill ships *with* the repo.

**Example: an `adr-draft` skill**

`.claude/skills/adr-draft/SKILL.md`:

```yaml
---
name: adr-draft
description: Draft a new ADR from a brief decision summary, using the project's
  template at docs/adr/_template.md. Auto-numbers from the highest existing ADR.
  Marks Status as Proposed; user must manually change to Accepted after review.
---

When invoked:
1. Read docs/adr/_template.md.
2. Scan docs/adr/ for the highest ADR number; the new one is N+1.
3. Generate ADR-{N+1} from the user's decision summary.
4. Fill Context, Decision, Consequences, Alternatives Rejected.
5. Save as docs/adr/ADR-NNN-<slug>.md with Status: Proposed.
6. Show the draft before considering it final.
```

The user says *"draft an ADR for switching from Postgres to MySQL"*, the skill fires, the draft follows your template — no re-explaining the template every time.

**Anti-pattern:** wrapping every prompt template in a skill. Skills are best for *procedures with steps*, not for *one-line prompt shortcuts*. For those, use slash commands.

### Slash commands

**What they are:** prompts stored in `.claude/commands/<name>.md` (project) or `~/.claude/commands/<name>.md` (user). Type `/<name>` and the file's content is sent as a prompt, with `$ARGUMENTS` replaced by anything you typed after the command. Lower ceremony than skills; faster to write; user-invoked, not auto-discovered.

**Where they fit in SDD:**

- The recurring prompt templates already documented in this repo (drafting an entry, auditing for staleness, generating diagnostics) become slash commands. *"I'd write a 30-line prompt every time"* becomes *"I type `/audit-docs`."*
- Per-repo workflow steps: `/start-session` reads `CLAUDE.md` and the most recent spec; `/end-session` writes a session summary.
- Team-shared commands ship in the repo and apply to every contributor.

**Example: a `/spec-new` command**

`.claude/commands/spec-new.md`:

```markdown
Read CLAUDE.md, ARCHITECTURE.md, and specs/_template/spec.md.

We're starting a new feature: $ARGUMENTS

Draft specs/YYYY-MM-<slug>/spec.md based on the template.
List open questions at the end — don't fill them in yourself.
Show me the draft before saving.
```

Then `/spec-new add rate limiting to the orders endpoint` does the whole setup in one keystroke.

**Difference from skills:** slash commands are *user-invoked, single-shot prompts*. Skills are *auto-discoverable, multi-step procedures*. If you're tempted to write a skill for *"just a longer prompt I use a lot"*, write a slash command instead — much lighter weight.

**Anti-pattern:** a repo with 30 slash commands. Discovery breaks down past ~10–15. Prefer a small set of high-leverage commands plus inline prompts for the rest.

### Subagents

**What they are:** delegated agents that handle a contained task in their own context window, then return a single result to the main session. Spawned via the `Agent` (a.k.a. `Task`) tool. The main session sees the delegated agent's final report — *not* its intermediate tool calls — which keeps the main context clean and the main session's token budget intact.

Subagents come in two flavors:

1. **Ad-hoc subagents** — spawned with a one-off prompt: *"Spawn a subagent with this prompt: …"* The prompt is the entire briefing; you get one shot.
2. **Configured subagents** — defined as files in `.claude/agents/<name>.md` (project) or `~/.claude/agents/<name>.md` (user). Each file has a `description` (when the agent should be used), a `tools` list (which tools it gets), and a prompt body. The main agent invokes them by name when the description matches the user's request.

Configured subagents are the right pattern when you'll use the same delegated workflow more than 2–3 times. Ad-hoc is fine for one-off explorations.

**Where they fit in SDD:**

- **Audit tasks.** Reading dozens of files to summarize state. The main session never sees the raw scan; it just gets the summary.
- **Cross-file research.** *"Find every reference to ADR-007 in specs and code; report which are still valid."*
- **Parallel investigation.** Multiple subagents working independently — one audits docs, one runs the test suite, one drafts the PR description, all in their own contexts.
- **Pre-implementation review.** Before implementing a spec, spawn a subagent to identify gaps in the plan.
- **Bounded code review.** Specialized review tasks (security, performance, accessibility) with their own toolset and prompt.

#### Example subagents (configured, in `.claude/agents/`)

**1. `doc-auditor.md` — staleness check across guides and runbooks**

```yaml
---
name: doc-auditor
description: Audit guides, runbooks, and CLAUDE.md for staleness against the
  current codebase. Use when the user asks "are our docs still accurate?",
  during quarterly reviews, or after a major refactor.
tools: Read, Grep, Glob, Bash
---

Walk docs/runbooks/, guides/, and CLAUDE.md. For each entry:
1. Check whether referenced service names, file paths, and commands still
   exist (verify against src/, config/, infra/).
2. Check whether any referenced ADRs are now Superseded.
3. Note the "Last verified" date; flag anything older than 6 months.

Return a markdown table: file, last verified, stale items, suggested action.
Do not modify any files. Return only the report.
```

**2. `spec-gap-finder.md` — pre-implementation sanity check**

```yaml
---
name: spec-gap-finder
description: Before implementing a spec, read it together with the affected
  source code and identify gaps in the plan. Use when the user is about to
  start implementing a spec.
tools: Read, Grep, Glob
---

The user gives you a spec folder path. Read spec.md, plan.md, and tasks.md
in that folder. Then read the source files referenced in plan.md.

Identify:
- Tasks that look incomplete or under-specified
- Dependencies on code that doesn't exist yet
- Conventions in CLAUDE.md the plan appears to violate
- Test coverage gaps not mentioned in tasks.md

Return a numbered list of concerns. Do not propose code changes — just flag.
```

**3. `adr-archaeologist.md` — find unwritten decisions worth ADR'ing**

```yaml
---
name: adr-archaeologist
description: Search the codebase for implicit decisions (custom implementations
  where a library exists, unusual workarounds, deliberate-looking patterns)
  that should probably be captured as ADRs. Use during SDD migration or
  quarterly.
tools: Read, Grep, Glob
---

Scan src/ for code that looks like a deliberate decision: custom code where
a standard library exists, workarounds with explanatory comments, unusual
structure, deliberate-looking patterns.

For each candidate:
1. Describe what was decided (one sentence)
2. Best guess at why (cite comments / commit messages where possible)
3. Mark whether this is worth a new ADR

Return a list. Don't draft the ADRs themselves — the human will pick which
to formalize.
```

**4. `test-coverage-scout.md` — find untested paths**

```yaml
---
name: test-coverage-scout
description: Identify functions or branches in a module that lack test
  coverage. Use before merging a non-trivial change, or when introducing
  a regression test for a bug.
tools: Read, Grep, Glob, Bash
---

The user gives you a module path. Read the source files and the matching
test files.

For each public function, identify:
- Whether it has any tests at all
- Whether error/edge-case branches are exercised
- Whether the happy-path tests actually assert what the function does

Return a table: function, has_tests, edge_cases_covered, suggested test
additions. Don't write the tests — just list them.
```

**5. `dependency-auditor.md` — check what's still used and what's outdated**

```yaml
---
name: dependency-auditor
description: Check whether dependencies in package.json / requirements.txt /
  *.csproj are still imported and whether outdated versions exist. Use
  before quarterly dependency-update windows.
tools: Read, Grep, Glob, Bash
---

Read the project's dependency manifest. For each dependency:
1. Grep src/ for imports — is it still actually used?
2. Check for known major-version updates (Bash to query the package
   registry if needed).
3. Flag: unused, outdated by 2+ major versions, security advisories.

Return a markdown table. Don't modify the manifest.
```

#### Ad-hoc subagent example: bounded research

For one-off research that doesn't justify a permanent agent file:

```
Spawn a subagent with this prompt:

> Find every place in src/ where we call the external acme-bank API.
> For each call site, return:
> - File and line number
> - Which endpoint is being called
> - Whether the call has retry logic
> - Whether errors are logged
>
> Return a markdown table. Don't modify anything.

I'll use the report to plan a refactor.
```

**Anti-pattern:** spawning a subagent for short tasks. Each subagent costs ~1–2k tokens just to set up its own context and tool registry. For anything under ~5 tool calls, inline is cheaper *and* faster.

### When the agent skips spawning a subagent

A common frustration: you write a prompt that clearly *would* benefit from delegation, the agent reads it, agrees with the plan, and then does the work inline anyway. The `.claude/agents/` file you carefully crafted sits unused.

This isn't random — there are predictable reasons.

**1. The model judged inline cheaper.** Subagents have setup overhead: a separate context window, a fresh tool registry, the prompt-to-subagent handoff, the return summary. If the model estimates the task is small (under ~5–10 tool calls), it usually skips delegation. From its perspective, that's the right call most of the time.

**2. The prompt didn't match any agent's `description`.** Configured subagents trigger on description matches. If your `.claude/agents/doc-auditor.md` says *"Use when the user asks 'are our docs still accurate?'"* but you wrote *"check if the runbooks are still good,"* the match may fail. Descriptions need to cover the phrasings you actually use, not the formal ones.

**3. The user prompt didn't suggest delegation.** Phrasing matters more than people expect. *"Spawn the `doc-auditor` subagent to audit the docs"* is a strong signal. *"Audit the docs"* leaves it open — and the model often interprets ambiguity as *"just do it in the main session"* because that's the lower-friction path.

**4. Path dependency from earlier tool calls.** If the model has been working in the main session for a while — reading files, making edits — it tends to continue in that mode. Switching mid-task to a subagent feels like a context break, so it often doesn't happen. Set the expectation at the *start* of a task, not in the middle.

**5. The `subagent_type` is ambiguous.** Claude Code's `Agent` tool takes a `subagent_type` argument (`general-purpose`, `Explore`, `Plan`, plus any project-defined ones). If your prompt doesn't make the right type obvious, the model may default to `general-purpose` — or skip the spawn entirely if no type fits cleanly.

**6. Permission scoping.** If the `Agent` tool isn't in the allowed-tools list for the current session (project `settings.json` or user `settings.json`), the model can't spawn it. You won't see an error; the spawn just silently doesn't happen.

**7. Agent definitions exist but aren't discoverable.** Some setups need an explicit pointer in `CLAUDE.md` or in the session start (*"Available subagents: doc-auditor, spec-gap-finder, …"*). If the agent file is in `.claude/agents/` but the main agent never sees a list, configured subagents stay invisible.

#### How to nudge it

- **Be explicit.** *"Spawn the `doc-auditor` subagent to handle this."* Naming the subagent by name forces the tool call.
- **Set expectations up front.** *"This is going to be a long audit — delegate to a subagent from the start. Don't read these files inline."*
- **Use slash commands.** A `.claude/commands/audit-docs.md` that says *"Spawn the doc-auditor subagent with the following prompt: …"* makes the delegation explicit every time the user types `/audit-docs`.
- **Tune the `description` field.** If your subagent isn't triggering, expand its description to cover more phrasings of how users actually ask for it. Test with the real prompt you'd type, not the one you'd write in a guide.
- **Verify tool availability.** Ask *"list the tools currently available to you"* mid-session — if `Agent` is missing, fix permissions, not the prompt.
- **List subagents in `CLAUDE.md`.** A small section: *"Available subagents (in `.claude/agents/`): doc-auditor (staleness check), spec-gap-finder (pre-implementation review), …"* — gives the main agent a discovery surface.

The general principle: subagents are an option the model considers, not an obligation. If you want them used consistently, remove the ambiguity from the prompt.

### Hooks

**What they are:** automation triggered by events, configured in `.claude/settings.json` (project) or `~/.claude/settings.json` (user). The *harness* runs them, not the agent. Hooks can **block** actions (return non-zero exit), **inject context** (output goes back to the model), or **run side effects** (lint, build, regenerate artifacts, log).

#### Events

| Event | Fires when | Typical use |
|-------|------------|------------|
| `PreToolUse` | Before any tool call | Block destructive actions, gate edits, lint inputs |
| `PostToolUse` | After a tool call completes | Trigger side effects (rebuild, regenerate, notify) |
| `UserPromptSubmit` | User submits a new prompt | Inject context, log session |
| `Stop` | Session ends normally | Session summary, cleanup, commit reminder |
| `SubagentStop` | A subagent finishes | Capture subagent output, validate report |
| `Notification` | Agent emits a notification | Forward to chat, log to file |

Each event accepts a `matcher` (which tool name or pattern triggers it — supports regex like `Edit|Write`) and a list of `hooks` (each with `type: command` and the shell command).

#### Where they fit in SDD

- **Enforce conventions.** A `PreToolUse` hook on `Edit|Write` for `.md` files runs `markdownlint`; malformed docs blocked at the source.
- **Automate doc maintenance.** A `PostToolUse` hook on `Edit|Write` for `guides/*.md` regenerates the PDF via pandoc + Prince. Agent edits markdown; PDF stays in sync without anyone remembering.
- **Inject context.** A `UserPromptSubmit` hook prepends *"the active ADR list is in `CLAUDE.md` § Active Decisions."* Agent gets the pointer for free.
- **End-of-session discipline.** A `Stop` hook checks for uncommitted spec changes, missing PR references in shipped specs, stale ADRs not updated this session.
- **Pre-commit gates.** Block commits that touch `src/` without a corresponding spec update, or modify ADRs with `Status: Accepted`.
- **Block dangerous tools.** A `PreToolUse` hook on `Bash` that catches `rm -rf` or `git push --force`.
- **Inject conventions just-in-time.** A `PreToolUse` hook on `Edit` for `*.cs` files prepends the team's C# style guide before the edit.

#### Several example hooks

**1. PDF auto-regen on guide changes**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "case \"$CLAUDE_FILE_PATHS\" in *guides/*.md*) pandoc guides/spec-driven-development-guide.md guides/working-with-agents-guide.md guides/runbook-operations-guide.md -o output.pdf --pdf-engine=prince -H pdf-style-compact.html --metadata title=\"claudecodeSDD\" ;; esac"
          }
        ]
      }
    ]
  }
}
```

Any time the agent edits a guide, the PDF rebuilds automatically.

**2. Block edits to ADRs already marked `Status: Accepted`**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ \"$CLAUDE_FILE_PATHS\" == *docs/adr/* ]] && grep -q '^Status: Accepted' \"$CLAUDE_FILE_PATHS\" 2>/dev/null; then echo 'Blocked: ADR is Accepted — only the Status header may change. Create a new ADR with Supersedes instead.' >&2; exit 1; fi"
          }
        ]
      }
    ]
  }
}
```

The agent tries to edit the ADR; the hook stops it; the agent sees the message on stderr and reconsiders.

**3. Inject the active ADR list on every prompt**

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Active ADRs:' && grep -l '^Status: Accepted$' docs/adr/*.md 2>/dev/null | sed 's|docs/adr/||'"
          }
        ]
      }
    ]
  }
}
```

The agent sees the active ADR list before every prompt; the human stops having to remember to include it.

**4. Markdown lint after every doc edit**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "case \"$CLAUDE_FILE_PATHS\" in *.md) markdownlint \"$CLAUDE_FILE_PATHS\" >&2 || true ;; esac"
          }
        ]
      }
    ]
  }
}
```

The `|| true` keeps the hook from blocking — issues just surface to stderr where the agent can read them. Drop `|| true` and ensure `markdownlint` exits non-zero to make it blocking.

**5. End-of-session checklist**

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "if git status --porcelain specs/ docs/adr/ | grep -q .; then echo 'Reminder: uncommitted changes in specs/ or docs/adr/.' >&2; fi"
          }
        ]
      }
    ]
  }
}
```

Surfaces the *"did you commit the spec / ADR?"* nudge at session end.

**6. Block `rm -rf` and `git push --force`**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'rm[[:space:]]+-rf|git[[:space:]]+push.*--force'; then echo 'Blocked: destructive command. If intended, run manually.' >&2; exit 1; fi"
          }
        ]
      }
    ]
  }
}
```

Catches the most common ways to lose work; bypassing requires explicit human action outside the agent.

#### Operational notes

- **Exit code = blocking behavior.** A hook that exits non-zero stops the tool call. Use this for gates; use stderr to communicate to the agent.
- **Hooks must be fast.** A hook that takes 5 seconds adds 5 seconds to every matching tool call. Aim for sub-100 ms. Push slow work into subagents or slash commands instead.
- **Hooks log to stderr by default.** The agent sees stderr output as part of the tool result. For your own debugging, also tee to a file (`>> ~/.claude/hooks.log 2>&1`).
- **Matchers support regex.** `Edit|Write` matches both. `Bash` matches all bash calls. `""` (empty) matches everything.
- **Useful environment variables.** Typically `$CLAUDE_FILE_PATHS` (paths from tool input), `$CLAUDE_TOOL_INPUT` (raw input), `$CLAUDE_TOOL_NAME`. Names vary by harness version — check current docs before relying on them.
- **Project vs user scope.** Hooks in `~/.claude/settings.json` apply only to you; the team doesn't see them. For team-shared invariants, put hooks in project `.claude/settings.json` and commit it.

#### Anti-patterns

- **Silent failures.** A hook that fails without logging is invisible damage. Always log to a file or stderr — the agent thinks the convention is enforced, your repo says otherwise.
- **Hooks doing real work.** A hook that runs a 30-second test suite is a misuse. Hooks validate / block / notify; for actual work, prefer subagents or slash commands.
- **Over-restrictive `PreToolUse`.** A hook that blocks all Bash calls except a whitelist will frustrate the agent into asking permission for everything. Block dangerous things, not anything that *could theoretically* be wrong.
- **Hook configuration not committed.** Hooks that only exist in one developer's `~/.claude/settings.json` aren't a team invariant — they're personal preference. For team-shared rules, commit `.claude/settings.json` to the repo.
- **No way to bypass.** A hook that absolutely blocks a tool call with no escape (e.g., environment variable to skip it) is painful when you legitimately need the action. Build in `CLAUDE_HOOKS_SKIP=1` or similar for the rare valid case.

### Which one to reach for

| Situation | Right tool |
|-----------|-----------|
| One-line prompt you'll type a lot | **Slash command** |
| Multi-step procedure with file lookups, conditional branches | **Skill** |
| Read-heavy task (audit, cross-repo research) that would bloat main context | **Subagent** |
| Something that should happen on a file change, session end, or tool call | **Hook** |
| One-off prompt you'll never repeat | **Just type it** |

The decision is mostly about *who triggers it* and *how isolated it should be*:

- **You trigger, single-shot, in main context** → slash command
- **Agent triggers, multi-step, in main context** → skill
- **You trigger, isolated context, returns summary** → subagent
- **System triggers on event, runs outside main context** → hook

### How these intersect with the SDD discipline

- **Skills and commands replace the prompt-template appendix.** This guide documents recurring prompts. In production, those prompts should live as `.claude/commands/*.md` and `.claude/skills/*/SKILL.md` files — discoverable, version-controlled with the repo, available to every contributor.
- **Hooks enforce SDD invariants that humans forget.** *"Update spec status after merge"*, *"regenerate the PDF when guides change"*, *"flag new conventions worth adding to `CLAUDE.md`"* — these are great hook candidates. The hook does the discipline; the human doesn't have to.
- **Subagents handle audit work that would otherwise be skipped.** Quarterly runbook audits, ADR-vs-code consistency checks, doc staleness reports. You'd never do them by hand; a subagent does them in five minutes.
- **`CLAUDE.md` is still the hub.** Skills, commands, and hooks are *implementations* of the discipline; `CLAUDE.md` is *where the discipline is declared*. The implementations should reference `CLAUDE.md`, not replace it.

### Anti-patterns across all four

1. **Wrapping every prompt in a skill.** Skills have ongoing cost — they live as files, need maintenance, get out of sync. Use them only for procedures you'll invoke 10+ times.
2. **A repo with 30 slash commands.** Discovery breaks down past ~10–15. Prefer a small high-leverage set plus inline prompts for the rest.
3. **Subagents for everything.** Each subagent burns its own tokens for context setup. For anything under ~5 tool calls, inline is cheaper and faster.
4. **Silent hooks.** A hook that fails without logging is invisible damage. Always log somewhere the team will read.
5. **Building blocks committed without docs.** A `.claude/skills/audit-everything/SKILL.md` with no comments is a black box. Document the *why* in the file, not just the *what*.
6. **Hook-driven over-automation.** Not every event needs a hook. The bar: it has to fix a problem that *actually* bit you, not a hypothetical one.

---

## When the Agent Drifts

The agent will eventually propose something that contradicts your conventions. This is not a failure; it's a fact of life. The fix is simple but requires discipline.

**Surgical correction:**

```
Stop. What you're proposing contradicts ADR-007 (we use Dapper, not EF).
Re-read ADR-007 and revise your approach.
```

**When the agent persists:**

```
You're still suggesting EF patterns. Two possibilities:
1. You forgot ADR-007 — re-read it now and confirm you understand
2. You think ADR-007 should change — if so, make the case for a new ADR
   that supersedes it. Don't sneak EF in via the back door.
```

**When the agent has a point:**

```
You're right that ADR-007 doesn't cover this case.
Draft an ADR-015 that extends ADR-007 for [specific scenario].
Don't supersede ADR-007 — extend it. Show me the draft.
```

The pattern across all three: **named documents in your prompts**. *"Per our conventions"* is weak. *"Per ADR-007 section Consequences"* is strong. The agent treats named documents as harder constraints than vague appeals to convention.

---

## Maintaining Documentation Proactively

You can train the agent to flag documentation gaps as it works, rather than waiting until you ask. Add this to `CLAUDE.md`:

```markdown
## Documentation Maintenance Rules
When working on any task, if you encounter:
- A convention not documented in CLAUDE.md → propose adding it
- A decision made implicitly that should be an ADR → flag it
- Terminology used inconsistently → propose a GLOSSARY entry
- An integration without docs in docs/integrations/ → propose creating one
- A new failure mode with no runbook entry → propose adding one

Always propose, never edit docs without explicit approval.
```

Then in sessions, occasionally ask:

```
Based on our work today, what documentation gaps did you notice?
List them with priority (critical / nice-to-have).
```

This converts the agent from a passive consumer of docs into an active contributor to keeping them fresh.

---

## Anti-Patterns

What NOT to do when working with the agent:

1. **"Just figure it out."** The agent will. The result will not match your conventions.

2. **"Use best practices."** Whose best practices? Yours live in `CLAUDE.md`. Reference them by name.

3. **"Make it production-ready."** Meaningless on its own. Say what production-ready means in your context (logging, error handling, tests, observability) — or point to `TESTING.md` and `CLAUDE.md`.

4. **Loading every doc on every prompt.** Wastes tokens, dilutes attention, makes the agent worse, not better. Pick relevant context per task.

5. **Skipping the post-implementation doc update.** This is how repos rot. The discipline is: spec → code → docs update, every time.

6. **Letting the agent write ADRs unsupervised.** ADRs are decisions you own. The agent drafts; you approve.

7. **Letting the agent invent diagnostic commands.** A runbook should be the source of truth. If the agent confidently generates `systemctl restart whatever-the-service-might-be-called`, you have a runbook gap, not a working diagnostic.

8. **Asking the agent to "remember" something across sessions.** It can't. Write it down in `CLAUDE.md`, or it doesn't exist.

9. **Auto-generating runbook entries without verification.** A confidently-worded runbook entry with wrong commands is worse than no runbook entry. Treat agent-drafted runbooks the same as agent-drafted ADRs — *draft only, you verify before saving*.

10. **Trusting tool output without spot-checks.** The agent reports *"I've read X.md."* Sometimes it skipped half of it. If a key fact is missing from the agent's response, point at the section explicitly.

---

## Golden Rules

1. **Agents don't discover — they follow pointers.** No pointer, no read.

2. **More docs aren't more context.** Past a threshold, they're less. Curate, don't accumulate.

3. **`CLAUDE.md` is the hub.** Every other doc hangs off it. Without a hub, the agent guesses.

4. **Stable, distinctive file names beat clever ones.** A name that survives two years is worth more than a name that's "obviously right" today.

5. **Context per task, not context per project.** Load what this task needs; the rest is noise.

6. **The agent's memory ends when the session ends.** Write it down or it's gone.

7. **Verify what the agent read.** Never assume. *"List the files you've read"* is a free correctness check.

8. **The agent drafts; you verify.** Especially for ADRs, runbooks, and anything irreversible.

9. **Archive aggressively.** Superseded ≠ deleted, but they look the same to the agent unless you separate them.

10. **Ambiguity is the enemy.** When in doubt, be more explicit, not less. Cost of clarity is one extra sentence; cost of ambiguity is a refactor.

---

*This guide complements [`spec-driven-development-guide.md`](spec-driven-development-guide.md) (design and decision layers) and [`runbook-operations-guide.md`](runbook-operations-guide.md) (operational layer). Together they describe what to write down; this guide describes how to put it in front of the agent.*
