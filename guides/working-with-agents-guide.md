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
8. [Universal Prompting Patterns](#universal-prompting-patterns)
9. [Claude Code Building Blocks](#claude-code-building-blocks)
10. [When the Agent Drifts](#when-the-agent-drifts)
11. [When the Agent Can't Build from the Spec](#when-the-agent-cant-build-from-the-spec)
12. [Maintaining Documentation Proactively](#maintaining-documentation-proactively)
13. [Anti-Patterns](#anti-patterns)
14. [Golden Rules](#golden-rules)

---

> **Convention used in this guide:** code blocks preceded by a **Prompt:** label (and tagged ` ```text`) are messages you would paste / type into the agent. Code blocks with other language tags (` ```json`, ` ```yaml`, ` ```bash`, ` ```markdown`, …) are configuration, scripts, or content artifacts — not prompts. ASCII diagrams and directory trees use plain code fences.

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
3. **It follows a reference.** It reads `SPEC.md`, sees *"see ADR-007"*, and reads ADR-007. Cross-references in your docs become a discovery graph the agent can walk.
4. **It follows a meta-rule.** Your `CLAUDE.md` says *"before generating SQL, read DOMAIN.md."* The agent obeys when the task triggers that rule.

There is no fifth path. The agent does **not** "scan the repo to understand the project" the way you would on day one. If you don't point at it, name it, link to it, or rule-mention it — the agent may never read it.

This is why `CLAUDE.md` matters disproportionately. It's the only file the agent reliably loads without being asked, and it's the only place where you can register meta-rules that fire automatically.

**Its tools shape what it reads.** Each session the harness hands the agent a toolset — read, search, edit, shell, delegate to a subagent, plus whatever MCP servers you've enabled — and each tool's description steers when it gets used. The exact list changes between versions; when it matters, ask *"list the tools currently available to you."* Two habits are worth keeping. When the agent keeps picking the wrong tool — a shell `cat` instead of its file reader, a full-file rewrite where an edit would do, ten inline reads a subagent should have summarized — fix it with one line in `CLAUDE.md` (*"always prefer X over Y"*) rather than re-prompting every session. And add MCP servers reactively, when a task actually needs one: each adds its tool descriptions to every prompt.

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
| < 10 | Search plus a few `CLAUDE.md` pointers find everything; mostly OK without extra structure | Decent file names |
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
- **Each prompt replays the full history.** Long sessions don't just *sit* at 80k tokens — every new prompt re-sends those 80k as input. Token cost grows superlinearly with session length, which is why ending sessions early (see "End sessions early" below) is one of the highest-leverage saves.

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

Output tokens are usually a small fraction of input — most code-assistant work is read-heavy. The exception is generating large files (a 500-line code block can cost 5k+ output tokens on its own). That gap quietly rewards spec-first work: a spec is cheap *input* that prevents expensive *output* — a discarded wrong generation bills at the premium output rate, while the spec that would have prevented it is input you write once (and cache thereafter).

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
3. **Full-file rewrites instead of diffs.** Generating a 400-line file because you changed 3 lines is the single most common token sink. Diffs cost dramatically less — ~50× is an order-of-magnitude rule of thumb, not a measured constant — and review faster.
4. **Verbose ceremony in prompts.** Twelve-line preambles (*"please, kindly, if you would…"*) are pure overhead. The agent doesn't reward politeness with better output.
5. **Long, unfiltered tool outputs.** `find` returning 800 files; `git log` with 2,000 commits. The agent reads them all, gets less useful per token, and burns budget on noise.
6. **Re-explaining context each session.** No `CLAUDE.md` → every session pays the *"what is this project"* tax from scratch.
7. **Iterating by regeneration instead of targeted edit.** *"Give me the whole file again with this one change"* instead of *"show me a diff for just this method."*

### Practical patterns that save tokens

**Be specific about what to read.** Point at file paths and ranges when you can. **Prompt (effective):**

```text
Read the GetOrders method in src/Repositories/OrderRepository.cs.
Don't read the rest of the file.
```

beats **prompt (wasteful):**

```text
Read the order repository and find the relevant method.
```

The first costs a few hundred tokens. The second can easily cost 5,000+ on a large file.

**Use `grep` / `glob` before `read`.** Find first, then load only the matches. **Prompt:**

```text
Grep src/ for `processBatch`. From the matches, read only the file
with the most occurrences. Don't load the others.
```

**Ask for diffs, not rewrites.** Any prompt that involves changing code should specify it. **Prompt:**

```text
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

A long session with a stable 10k-token prefix saves roughly: `10,000 × (1.00 − 0.10) = 9,000 tokens per prompt` after the first. Across 20 prompts in a session, that's 180,000 cached tokens — at ~5–10k tokens per large file read, the equivalent of a couple dozen large file reads you never paid for.

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

At any point in a session, ask. **Prompt:**

```text
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

**For a new feature** — run [`/sdd-3-spec-new`](../templates/.claude/commands/sdd-3-spec-new.md) `<one-paragraph idea>`. It drafts `specs/<date-slug>/spec.md` from the spec template before any code exists, and lists the open questions instead of answering them itself. (The trio's other steps each have their own command — see [`spec-plan-tasks-guide.md` § AI-assisted authoring](spec-plan-tasks-guide.md#ai-assisted-authoring-one-command-per-step).)

**For modifying an existing feature** — **Prompt:**

```text
Read CLAUDE.md and ARCHITECTURE.md.
We're modifying [feature]. Read its original spec at specs/[folder]/spec.md
and the relevant code at [paths].
I want to: [change description].
Before changing anything, summarize:
1. Which files will need to change
2. Which conventions or ADRs apply
3. What you would do — wait for my approval before writing code.
```

The pattern is the same in both — the command bakes it in, the prompt spells it out: **load context → state intent → propose plan → wait for green light**. Don't let the agent jump straight to code.

### Ending a Session

The last 60 seconds of a session matter more than the first 10 minutes — they decide whether next session has to start from scratch.

**Prompt:**

```text
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

**For a new Accepted ADR** — **Prompt:**

```text
I just added docs/adr/ADR-014-[slug].md.
Read it carefully. Then:
1. Update the "Active decisions" list in CLAUDE.md (add ADR-014)
2. Check if any other docs reference the old approach and need updating
3. Tell me what code in the current codebase contradicts this ADR
   — do NOT change code yet, just list the locations.
```

**For an ADR that Supersedes another** — **Prompt:**

```text
ADR-014 supersedes ADR-003. I've already updated ADR-003's status header.
Please:
1. Update CLAUDE.md's "Active decisions" list — remove ADR-003, add ADR-014
2. Grep the codebase and specs/ for references to ADR-003
3. For each reference, recommend whether it stays (historical context)
   or needs updating to point to ADR-014.
```

**For documenting a decision after the fact** — use the *"Draft a new ADR from a decision summary"* prompt in [`adr-guide.md` § Agent-assisted drafting](adr-guide.md#agent-assisted-drafting); it lives there only, so it can't drift.

### After Creating a Spec — Starting Implementation

A spec is ready. Now you want the agent to implement it. Don't just say *"code it up."*

(For *authoring* the spec / plan / tasks themselves with AI help — the command for each artifact, the trio consistency check, the iteration loops — see [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md). This assumes the trio is already written and reviewed.)

Run [`/sdd-7-implement`](../templates/.claude/commands/sdd-7-implement.md) `[spec folder]`. It reads the trio plus `TESTING.md` and `CLAUDE.md`, refuses to build on a trio that clearly hasn't passed `/sdd-6-trio-check`, then works `TASKS.md` one task at a time: failing test first, the minimum code to turn it green, the task's `→ verify:` step, one commit per green task — touching only the files listed in `plan.md`. When a task can't be built it stops and names the spec gap instead of improvising, and after the last task it runs the break-the-code check.

Every commit is a checkpoint you can roll back to. If you'd rather approve each task before the next one starts, say so when you invoke it — either way the principle holds: **task by task, with checkpoints**.

### After Implementing — Updating Documentation

This is the step most teams skip, and it's why repos rot. After merging code, the agent should help maintain the docs that just became outdated.

**Prompt:**

```text
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

**Prompt:**

```text
We just completed a major refactor of [subsystem].
Before we continue work:
1. Read the current state of [paths]
2. Compare against ARCHITECTURE.md — list discrepancies
3. Propose updates to ARCHITECTURE.md to reflect reality
```

---

## Universal Prompting Patterns

These work across all situations, regardless of whether you're writing a spec, drafting a runbook, or fixing a bug.

All six are **prompts** — paste them verbatim or adapt to your situation.

**"Plan before code":**

```text
Don't write code yet. First, outline what you'll do in 5-10 bullet points.
I'll approve or correct before you start implementing.
```

**"Cite your source":**

```text
For each decision in your proposal, cite the document that supports it
(CLAUDE.md section, ADR number, spec path).
If a decision isn't grounded in any document, mark it as [ASSUMPTION]
and we'll discuss before proceeding.
```

**"Diff, don't replace":**

```text
Show me proposed changes as diffs against the current file,
not as a full rewrite. I want to see exactly what changes.
```

**"Question before assumption":**

```text
If anything in this task is ambiguous, list your questions before starting.
I'd rather answer 5 questions now than refactor 50 lines later.
```

**"Stay in scope":**

```text
The spec is specs/[date-slug]/. Do not modify any file outside the paths
listed in plan.md. If you think a change outside scope is necessary,
stop and propose extending the spec — don't silently expand.
```

**"List what you read":**

```text
Before continuing, list every file you've read in this task and one sentence
on why. If any of them are stale or wrong-source, tell me before you proceed.
```

The last one is underrated — it surfaces silent assumptions early.

---

## Claude Code Building Blocks

Plain prompting takes you a long way. Claude Code (and similar agent tools) adds four ways to encode a recurring workflow so you don't retype it every session: **slash commands**, **skills**, **subagents**, and **hooks**. For a solo dev the trio plus the shipped `/sdd-*` commands already cover the day-to-day; reach for the others only when a workflow genuinely recurs.

### Which one to reach for

| Situation | Right tool | Where it lives |
|-----------|-----------|----------------|
| A prompt you type a lot | **Slash command** — you invoke it; `$ARGUMENTS` is whatever you type after it | `.claude/commands/<name>.md`. This repo ships fourteen — [`sdd-3-spec-new.md`](../templates/.claude/commands/sdd-3-spec-new.md) shows the shape |
| A multi-step procedure with file lookups and branches | **Skill** — the agent can pick it up when a prompt matches its description | `.claude/skills/<name>/SKILL.md` (frontmatter says *when* it applies, the body is the procedure) |
| A read-heavy job (an audit, cross-repo research) that would bloat the main context | **Subagent** — its own context window; the main session sees only its final report | Ad hoc (*"spawn a subagent to…"*), or configured in `.claude/agents/<name>.md` with a `description`, a `tools` list, and a prompt body |
| Something that must happen on an event — a tool call, a prompt, session end | **Hook** — run by the harness, not the agent | `.claude/settings.json` — mechanics and the ready SDD hooks: [`quality-gates-guide.md` § Claude Code hooks](quality-gates-guide.md#claude-code-hooks-in-session) |
| A one-off you'll never repeat | **Just type it** | — |

The decision is mostly about *who triggers it* and *how isolated it should be*:

- **You trigger, single-shot, in main context** → slash command
- **Agent triggers, multi-step, in main context** → skill
- **You trigger, isolated context, returns a summary** → subagent
- **System triggers on an event, outside the agent** → hook

**In an SDD repo:** `CLAUDE.md` is where the discipline is *declared*; these are *implementations* of it, so they reference `CLAUDE.md` rather than replace it. Commit them with the repo so every contributor gets them, and keep the set small — beyond the shipped `/sdd-*` pipeline, a handful of your own, not thirty. A subagent re-pays the system prompt and tool registry (realistically ~10k+ tokens) before it does any work, so anything under ~5 tool calls is cheaper inline; if a configured subagent never fires, name it in the prompt (*"spawn the `<name>` subagent"*) or wrap it in a slash command. Add a block only when a real recurring need asks for it, not a hypothetical one.

---

## When the Agent Drifts

The agent will eventually propose something that contradicts your conventions. This is not a failure; it's a fact of life. The fix is simple but requires discipline.

**Surgical correction** — **Prompt:**

```text
Stop. What you're proposing contradicts ADR-007 (we use Dapper, not EF).
Re-read ADR-007 and revise your approach.
```

**When the agent persists** — **Prompt:**

```text
You're still suggesting EF patterns. Two possibilities:
1. You forgot ADR-007 — re-read it now and confirm you understand
2. You think ADR-007 should change — if so, make the case for a new ADR
   that supersedes it. Don't sneak EF in via the back door.
```

**When the agent has a point** — **Prompt:**

```text
You're right that ADR-007 doesn't cover this case.
Draft an ADR-015 that extends ADR-007 for [specific scenario].
Don't supersede ADR-007 — extend it. Show me the draft.
```

The pattern across all three: **named documents in your prompts**. *"Per our conventions"* is weak. *"Per ADR-007 section Consequences"* is strong. The agent treats named documents as harder constraints than vague appeals to convention.

---

## When the Agent Can't Build from the Spec

Sometimes the agent stalls, goes in circles, or emits broken or obviously-wrong code *even though* you handed it a finished spec (and a plan, and tasks). The instinct is to drop the discipline and prompt-and-pray your way to something that runs — **vibe coding.** Don't. That failure is almost always a *signal that the spec or plan has a gap*, not a license to abandon them. Vibe coding is the exact thing the spec was protecting you from.

Work the ladder instead, cheapest rung first:

1. **Ask whose gap it is.** A spec the agent "can't implement" is usually underspecified, self-contradictory, or too big. If a competent human would also stop and ask questions here, the gap is in the spec — fix that, not the agent.
2. **Shrink the unit.** Point it at *one* acceptance criterion or *one* task, not the whole feature. Big asks overwhelm; small ones succeed. This is what `TASKS.md` is for — **Prompt:**

   ```text
   Ignore the rest of the feature. Implement only AC1 from spec.md.
   Write the failing test first, then the minimum code to pass it. Stop there.
   ```
3. **Feed the missing context.** The agent improvises when it lacks the file, convention, or example. Name the thing to mirror — *"follow the pattern in `AuthMiddleware.cs`"*, the relevant ADR, the integration doc. Most "wrong code" is a missing-context problem, not a missing-capability one.
4. **Resolve the open question it's tripping on.** An unanswered Open Question in the spec makes the agent fabricate — and the fabrication is what's breaking. Decide it, write it down, re-run.
5. **Listen when it pushes back.** Sometimes *"I can't"* means the agent caught a real contradiction — the plan violates an ADR, an AC is impossible. That's a free spec review; see [When the Agent Drifts](#when-the-agent-drifts) § "When the agent has a point." Fix the spec, not the agent.
6. **If it's genuinely stuck, write the hard kernel yourself — then feed it back.** When a piece truly exceeds the agent (a gnarly algorithm, subtle concurrency), *you* write that part. That is **targeted manual coding, not vibe coding**: the spec already pinned down what "done" means, and the acceptance criteria are your tests. Fold what you learned back into the spec/plan (a note, a constraint, a new ADR) so the next change isn't blind — then hand the rest back.

**The line that matters:**

- **Vibe coding** — no spec; prompt-and-pray; you accept whatever runs; the intent lived only in your head and is now gone.
- **Targeted manual coding** — you write the tricky part *because the spec already defined the target*; the spec stays the source of truth; the ACs are still the finish line.

A spec that's hard to implement is the method *working* — it surfaced the ambiguity before that ambiguity shipped as mystery code. The recovery is always to sharpen the spec, shrink the task, or hand-write one kernel with the spec still in charge — never to throw the spec away.

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

Then in sessions, occasionally ask. **Prompt:**

```text
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
