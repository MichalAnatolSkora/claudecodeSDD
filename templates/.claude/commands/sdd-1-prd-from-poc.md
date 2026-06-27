---
description: Read a kept proof-of-concept, interrogate me for the why, and backfill a lean PRD (product what & why) — stops at the PRD layer, never fabricates a spec
argument-hint: [path to the PoC] (optional, default: src/)
---

# PRD from a kept PoC

You built a working proof-of-concept with nothing written down, it's good, and you want to start doing SDD. I'll read what the PoC does, then ask you the why-level things code can't answer, and backfill a **lean PRD** at `docs/prd/YYYY-MM-slug.md` — reviewed next via `/sdd-1-prd-review` (see `guides/prd-guide.md`). PoC to read: `$ARGUMENTS` (default: `src/`, or the repo's main source dir — tell me which I used).

This is the PoC→PRD sibling of `/sdd-1-prd-new` (idea→PRD). It's for a PoC you intend to **keep**, not a large/legacy system — that audience uses `/sdd-2-architecture-from-code` and `guides/legacy-to-sdd-migration-guide.md`.

**Code gives the *what*, not the *why*.** The PoC shows what you built; it can't tell me the problem, who the user is, what success means in numbers, or what's deliberately out of scope. So I scan to sketch the *what*, then lead the Q&A with the *why*. I draft and ask; **you make the product calls.**

## Steps

1. **Scan** the PoC to learn what it does today: entry points (CLI, API routes, UI, jobs), the main capabilities it actually performs, the inputs it takes and outputs it produces, and the external things it touches. Enough to describe behavior — not an architecture map (that's a separate, optional step below).
2. **Sketch** a lean PRD with these sections, filling only what the PoC and your answers support:
   *Summary · The problem · Target users · Solution overview (what it does today) · Success metrics · Constraints · Out of scope · Risks & assumptions.*
   Write **Solution overview** from the scan; leave the why-level sections thin until step 4. **No implementation detail** — no language, framework, database, API, or class names; that lives in code now and in specs later. Mark anything you inferred from the code `[VERIFY]`.
3. **Don't launder the PoC.** Where it made an implicit, undocumented, or questionable decision — or took a shortcut — surface it and mark it `[VERIFY]`. Don't tidy it into a doc that looks more deliberate than the PoC actually was; the draft should read like a spike captured honestly, not like a plan that was followed.
4. **Ask me** the things code can't supply, why-level first, because these are the genuinely undecided ones: the exact problem and its cost today, who precisely the user is (job title + context), what success looks like in numbers, what's explicitly out of scope, any hard constraints. Group them, a few at a time, and wait for my answers.
5. **Fill** in my answers, show the updated draft, and ask another round only if real gaps remain. Stop when nothing important is still a guess.

## Saving (only after I accept)

Show me the full draft first. Save to `docs/prd/YYYY-MM-slug.md` only after I accept it. The PRD is documentation/backfill — it does not retroactively make this code spec-driven.

## The boundary — this stops at the PRD

- **This produces a PRD, never a spec.** A PRD is product-level *what & why* — descriptive, why-level, and no agent ever writes code *from* it, so reverse-capturing one is legitimate. A spec ships *before* the code it describes — reverse-capturing one is fiction.
- **Do NOT fabricate acceptance criteria.** No spec, no AC, no plan, no tasks — nothing that pretends to have been written before the PoC existed. That's this repo's strongest anti-pattern (`spec retro-fitted from completed code` in `guides/spec-plan-tasks-guide.md`; *"reverse-engineered specs are fiction"* in `guides/legacy-to-sdd-migration-guide.md`).
- **I stop at the PRD layer.** If you want a spec, it's for the *next* change — not a backfill of this one.

## From here forward

The PoC is a spike you've decided to keep; this PRD is backfill. The **spec-before-code discipline starts with the NEXT change**, not retroactively. Pick one honest path:

- **(a) Harden** — rebuild the kept feature properly through a real spec → trio (`/sdd-3-spec-new` → plan → tasks), so the spec genuinely precedes the hardened code. The PoC was the throwaway spike that showed you what good looks like — write the acceptance criteria fresh in that spec, before the hardened code.
- **(b) Treat it as legacy from day one** — forward-only specs; don't retro-fit specs to what already exists (`guides/legacy-to-sdd-migration-guide.md`).

Optional next step, **offered not auto-run**: `/sdd-2-architecture-from-code` to capture the PoC's structure. That's descriptive too — same boundary, no specs.

Keep the PRD **factual and tight** — claims, numbers, and decisions, not marketing prose or filler. A human reviews this next (`/sdd-1-prd-review`), so every sentence has to earn its place; verbosity is where gaps hide. Keep it to a page or two.
