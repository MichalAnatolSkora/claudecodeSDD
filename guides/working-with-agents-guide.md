# Working with AI Coding Agents

> How the agent actually reads your repo, why it sometimes misses things, how far you can push the document count before it breaks down — and the prompting patterns that keep it on track.

---

## Table of Contents

1. [Core Philosophy](#core-philosophy)
2. [How the Agent Reads Your Repo](#how-the-agent-reads-your-repo)
3. [When the Agent Loads a File — and Why It Sometimes Doesn't](#when-the-agent-loads-a-file--and-why-it-sometimes-doesnt)
4. [How Many Files Is Too Many](#how-many-files-is-too-many)
5. [Token Economy](#token-economy)
6. [Starting and Ending Sessions](#starting-and-ending-sessions)
7. [Working on Specs, ADRs, and Refactors](#working-on-specs-adrs-and-refactors)
8. [Working on Runbooks and Operational Tasks](#working-on-runbooks-and-operational-tasks)
9. [Universal Prompting Patterns](#universal-prompting-patterns)
10. [Claude Code Building Blocks](#claude-code-building-blocks)
11. [When the Agent Drifts](#when-the-agent-drifts)
12. [Maintaining Documentation Proactively](#maintaining-documentation-proactively)
13. [Anti-Patterns](#anti-patterns)
14. [Golden Rules](#golden-rules)

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

Modern agent platforms (Anthropic's API, Claude Code) cache stable conversation prefixes. A cache hit is roughly **~10% of the cost** of an uncached read on the same content; sessions can become dramatically cheaper per prompt the longer they run, *if* the prefix stays consistent.

To benefit:

- **Put stable docs at the start.** `CLAUDE.md`, `ARCHITECTURE.md`, reference files — load them first, in the same order, every session.
- **Don't shuffle file-reading order across prompts.** Cache hits depend on exact prefix matches; reordering reads invalidates the cache.
- **Keep stable files stable.** Editing `CLAUDE.md` mid-session invalidates the cached prefix for that session.
- **Be aware of TTL.** Caches expire after a few minutes of inactivity (typically 5 min on default tiers). Long pauses reset the saving.

In practice: long sessions with consistent context get *cheaper* per prompt as more of the prefix is cached. This rewards a session shape where you front-load context once and then iterate.

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

When you add a new ADR (especially `Accepted` or one that `Supersedes` another), the agent doesn't magically know. Tell it explicitly.

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

**What they are:** delegated agents that handle a contained task in their own context window, then return a single result to the main session. Spawned via the Agent tool. Used for work that would bloat the main session if done inline.

**Where they fit in SDD:**

- **Audit tasks.** *"Audit `docs/runbooks/` for staleness against current `src/`."* The audit reads dozens of files; you don't want them in your main context.
- **Cross-file research.** *"Find every reference to ADR-007 in specs and code; report which are still valid."*
- **Parallel investigation.** Multiple subagents working independently. One audits docs, one runs the test suite, one drafts the PR description.
- **Pre-implementation review.** *"Before I implement spec `2026-05-x`, spawn a subagent to read the spec + the affected code and identify gaps in the plan."*

**Example: a doc-audit subagent**

```
Spawn a subagent with this prompt:

> Read every file in docs/runbooks/ and every file in src/. For each runbook,
> check whether referenced service names, file paths, and commands still match
> the current codebase. Return a markdown table: runbook, last verified date,
> stale items found, suggested action.
>
> Do not modify any files. Return only the report.

After the subagent reports back, I'll decide which runbooks to update.
```

The main session stays focused; the audit happens in parallel.

**Anti-pattern:** spawning a subagent for short tasks. The handoff has overhead — context setup, tool authorization, result transmission. For anything under ~5 tool calls, just do it inline.

### Hooks

**What they are:** automation triggered by events, configured in `settings.json` (project or user). Common events: `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`, `SubagentStop`. Hooks run shell commands or scripts; the harness executes them, not the agent itself. They can block actions, inject context, or run side effects.

**Where they fit in SDD:**

- **Enforce conventions.** A `PreToolUse` hook on `Edit|Write` for `.md` files runs `markdownlint`; commits to malformed docs get blocked at the source.
- **Automate doc maintenance.** A `PostToolUse` hook on `Edit|Write` for `guides/*.md` regenerates the PDF via pandoc + Prince. The agent edits markdown; the PDF stays in sync without anyone remembering.
- **Inject context.** A `UserPromptSubmit` hook prepends a reminder (*"the active ADR list is in `CLAUDE.md` § Active Decisions"*). The agent gets the pointer for free.
- **End-of-session discipline.** A `Stop` hook checks for uncommitted spec changes, missing PR references in shipped specs, or stale ADRs not updated this session.
- **Pre-commit gates.** Block commits that touch `src/` without a corresponding spec update, or that modify an ADR with `Status: Accepted` (only headers may be edited).

**Example: PDF auto-regen on guide changes**

`.claude/settings.json`:

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

Now any time the agent edits a guide, the PDF rebuilds automatically.

**Anti-pattern:** hooks that fail silently. A broken hook that swallows errors is worse than no hook — the agent thinks the convention is enforced, your repo says otherwise. Always log to a file or print to stderr.

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
