# Runbook / Operations Documentation Guide

> A practical guide to writing the document that saves you at 3 a.m. — and that an AI coding agent reads when you ask it to generate a diagnostic script, a recovery procedure, or a maintenance task.

---

## Table of Contents

1. [Core Philosophy](#core-philosophy)
2. [RUNBOOK vs OPERATIONS — Naming and Scope](#runbook-vs-operations--naming-and-scope)
3. [The 3 a.m. Test](#the-3-am-test)
4. [What Belongs in a Runbook (and What Does Not)](#what-belongs-in-a-runbook-and-what-does-not)
5. [Anatomy of a Runbook Entry](#anatomy-of-a-runbook-entry)
6. [Full Entry Template](#full-entry-template)
7. [Common Entry Categories](#common-entry-categories)
8. [Organization: Single File vs Folder Structure](#organization-single-file-vs-folder-structure)
9. [Writing for AI Agents](#writing-for-ai-agents)
10. [Maintenance Discipline](#maintenance-discipline)
11. [Working with the Agent: Practical Commands](#working-with-the-agent-practical-commands)
12. [Anti-Patterns](#anti-patterns)
13. [Golden Rules](#golden-rules)

---

## Core Philosophy

A **runbook** is the operational layer of your project. It is not architecture, not requirements, not design history. It is the document that turns *"the system works"* into *"the system stays working."*

Three audiences read a runbook:

- **You at 3 a.m.** — half-awake, on-call, looking for the fastest path to a green dashboard.
- **A new team member** — week one, doesn't know which logs matter, doesn't know which knobs are dangerous to turn.
- **An AI coding agent** — generating a diagnostic script, a health-check endpoint, or a recovery procedure, and asking *"what's the actual SFTP health check command for this project?"*

A runbook fails if any of these three audiences can't act on it.

**The core insight:** documentation about *what code does* lives in code and tests. Documentation about *how to keep the system running when reality intrudes* lives in a runbook — because no test catches "the SFTP partner rotated their host key without telling us."

Related terms you'll see used interchangeably (with subtle differences):
- **Runbook** — step-by-step procedures for known operations and incidents
- **Operations manual / OPERATIONS.md** — broader scope: runbook + day-to-day maintenance + monitoring
- **Playbook** — typically incident-response specific, often security-focused
- **SOP (Standard Operating Procedure)** — formal, audit-oriented version of a runbook (regulated industries)

For most projects, one file or one folder under any of these names works. Don't dwell on the label — focus on whether someone can act on what's written.

---

## RUNBOOK vs OPERATIONS — Naming and Scope

The two filenames cover overlapping but not identical territory. Pick one early and stick with it.

### `RUNBOOK.md` — narrower, incident-oriented

Best when the content is mostly:
- *"X is broken, here's how to fix it"*
- *"Y is stuck, here's how to unstick it"*
- *"Z alert fired, here's what it means and what to do"*

Use this name if the file is overwhelmingly about *recovery* and *incident response*. Common in projects with on-call rotation.

### `OPERATIONS.md` — broader, lifecycle-oriented

Best when the content includes:
- Recovery procedures (the runbook part)
- Routine maintenance (clearing old data, rotating credentials, vacuuming, log retention)
- Deployment procedures (how to release, how to roll back, what to verify)
- Monitoring guide (what to watch, what's normal, what's not)
- Backup and restore procedures

Use this name if the file is the operational handbook for the whole system, not just the panic-button manual.

### My recommendation

For a mid-sized project: start with `OPERATIONS.md`. The moment that file crosses ~400 lines, split it:
- `OPERATIONS.md` stays as the high-level index + routine procedures
- `docs/runbooks/` becomes a folder of focused, per-incident entries
- `docs/deployment.md` extracts release procedures
- `docs/monitoring.md` extracts the watching guide

Resist creating five small files on day one. You'll wind up updating none of them. One file you actually open beats four perfectly organized ones you ignore.

---

## The 3 a.m. Test

Every runbook entry must pass the **3 a.m. test**: could a half-awake on-call engineer follow this to resolution without thinking?

This means:

- **Exact commands, not descriptions of commands.** `kubectl -n campaigns rollout restart deployment/scheduler` — not *"restart the scheduler deployment"*.
- **Concrete paths, not project-relative hints.** `/var/log/campaigns/scheduler.log` — not *"check the scheduler logs"*.
- **Copy-pasteable queries.** Real SQL with real table names, not `SELECT * FROM <log_table>`.
- **What "success" looks like at each step.** Otherwise the on-call doesn't know whether to continue or escalate.
- **An escalation path.** If step 5 fails, who do you call, and what time zone are they in?

If a step contains the word *"check"* without specifying *what command* and *what output to look for*, that step fails the test.

**Bad:**
> If the SFTP transfer is failing, check the connection and restart the job.

**Good:**
> 1. Verify SFTP reachability:
>    ```bash
>    nc -zv sftp.partner.example.com 22
>    ```
>    Expected: `Connection to sftp.partner.example.com port 22 [tcp/ssh] succeeded!`
> 2. If unreachable, escalate to network team (Slack `#net-oncall`) — do not proceed.
> 3. If reachable, check the most recent Quartz job log:
>    ```bash
>    tail -200 /var/log/campaigns/quartz.log | grep -E "SftpDeliveryJob|ERROR"
>    ```
> 4. If you see `AuthenticationException`, the host key may have rotated — go to entry `SFTP host key rotation`.
> 5. If you see `IOException: connection reset`, restart the Quartz job:
>    ```bash
>    curl -X POST http://localhost:8080/admin/quartz/jobs/SftpDeliveryJob/restart
>    ```
>    Expected response: `{"status":"restarted","nextFire":"..."}`

The difference is not length. It's *actionability*.

---

## What Belongs in a Runbook (and What Does Not)

A runbook attracts unrelated content unless you draw the line clearly.

### Belongs in the runbook

- **Recovery procedures** for known failure modes
- **Health checks** — how to tell if a subsystem is alive
- **Diagnostic commands** for narrowing down "something is wrong"
- **Restart / restore / replay procedures**
- **Routine maintenance** — log rotation, archive cleanup, credential rotation
- **Deployment and rollback procedures**
- **What "normal" looks like** for key metrics (so the on-call recognizes "not normal")
- **Escalation paths** — who to call, when, and how

### Does NOT belong in the runbook

- **Architecture explanations** → `ARCHITECTURE.md`
- **Why we chose X over Y** → ADR
- **Feature specifications** → `specs/`
- **Domain terminology** → `DOMAIN.md` / `GLOSSARY.md`
- **Setup for new developers** → `ONBOARDING.md`
- **Coding conventions** → `CLAUDE.md`
- **Long-term planning** → `ROADMAP.md`
- **Post-mortems** → `docs/postmortems/` (the runbook may *link* to them and absorb their lessons, but the analysis itself lives separately)

**Practical test:** if the information is useful only when nothing is broken, it doesn't belong in the runbook. The runbook earns its place by being the first thing you open when something is wrong.

---

## Anatomy of a Runbook Entry

Every entry — whether for an incident, a maintenance task, or a deployment — should follow the same shape. Predictable structure is what makes the runbook usable under stress.

### Required fields

1. **Title** — a short noun phrase describing the situation, not the solution. *"SFTP delivery stuck"* — not *"Restart SFTP job"*. The on-call searches for symptoms, not for fixes.
2. **Severity / Impact** — what is broken from the user's perspective, and how bad it is.
3. **Symptoms** — what an on-call will *see* (alerts, log messages, dashboard state). The matching surface.
4. **Pre-checks** — quick sanity checks before you start changing things. Often the issue is "this isn't actually broken, just slow."
5. **Diagnosis steps** — narrowing down the cause, in order.
6. **Recovery steps** — the actual fix, with exact commands and expected outputs.
7. **Verification** — how to confirm the fix worked.
8. **Escalation** — who to call and when, if any step fails.
9. **Post-incident actions** — what to do *after* the fix (notify users, write up incident, file follow-up ticket).

### Optional but valuable fields

- **Last verified** — date you last walked through the procedure end-to-end. Anything older than 6 months should be re-verified.
- **Related entries** — links to nearby procedures (so the on-call can jump if symptoms point elsewhere).
- **Related ADRs / specs** — if the procedure exists because of a specific decision (e.g., *"we don't auto-retry because ADR-014"*), link it.
- **Known gotchas** — things that bit someone before. *"Do NOT run the cleanup script during the 02:00–04:00 maintenance window — it deadlocks with the nightly archive."*

---

## Full Entry Template

````markdown
# Runbook: SFTP delivery stuck (campaigns queue building up)

**Severity:** High — outbound campaign delivery is halted. Customer SLAs at risk after 30 min.
**Last verified:** 2026-05-12 (incident replay)
**Owner:** Platform team (`#platform-oncall`)

## Symptoms

- Grafana alert `CampaignDeliveryLag > 15m` firing
- Dashboard `Campaigns > Delivery` shows `pending_in_queue` rising
- Recent entries in `T_CAMPAGINES_LOG` have `STATUS = 'QUEUED'` older than 10 min
- (Sometimes) Slack alert from `#campaigns-alerts` bot

## Pre-checks (30 seconds)

1. Confirm this is not a deploy in progress:
   ```bash
   gh run list --workflow=deploy.yml --limit 3
   ```
   If a deploy is *in progress* (status `in_progress`), wait 5 min and recheck — Quartz pauses briefly during deploys.

2. Confirm partner-side is not in announced maintenance:
   - Check `#vendor-status` Slack channel
   - Check partner status page: https://status.partner.example.com

If either is true → stop here. Document the wait in the incident channel.

## Diagnosis

### Step 1: Is Quartz running?
```bash
curl -s http://localhost:8080/admin/quartz/health | jq .
```
Expected:
```json
{ "status": "UP", "scheduler": "running", "lastFire": "..." }
```

If `scheduler: "paused"` or `status: "DOWN"` → jump to **Recovery A**.

### Step 2: Is the SFTP partner reachable?
```bash
nc -zv sftp.partner.example.com 22
```
Expected: `Connection to sftp.partner.example.com port 22 [tcp/ssh] succeeded!`

If connection refused or times out → escalate to network team (`#net-oncall`). Do not proceed.

### Step 3: Is the job throwing?
```bash
tail -500 /var/log/campaigns/quartz.log \
  | grep -E "SftpDeliveryJob" \
  | tail -50
```

Match on the most recent ERROR:
- `AuthenticationException` → **Recovery B** (host key rotation)
- `IOException: Connection reset` → **Recovery C** (transient network)
- `OutOfMemoryError` → **Recovery D** (heap exhausted — batch too large)
- Anything else → escalate (`#platform-oncall`, page lead engineer)

## Recovery

### Recovery A: Quartz paused or down
```bash
curl -X POST http://localhost:8080/admin/quartz/scheduler/resume
```
Expected: `{"status":"running"}`

If status remains `paused` after 30 seconds, restart the service:
```bash
sudo systemctl restart campaigns-scheduler
```

### Recovery B: SFTP host key rotation
1. Fetch the partner's new host key (from email or partner portal — never accept blindly).
2. Update `/etc/campaigns/known_hosts`:
   ```bash
   sudo ssh-keygen -R sftp.partner.example.com
   ssh-keyscan -H sftp.partner.example.com >> /etc/campaigns/known_hosts
   ```
3. Verify fingerprint against the partner's confirmation:
   ```bash
   ssh-keygen -lf /etc/campaigns/known_hosts | grep sftp.partner
   ```
4. Restart the scheduler (see Recovery A).

### Recovery C: Transient network
1. Restart only the affected job (avoids cold cache):
   ```bash
   curl -X POST http://localhost:8080/admin/quartz/jobs/SftpDeliveryJob/restart
   ```
2. Watch one cycle (jobs run every 5 min):
   ```bash
   tail -f /var/log/campaigns/quartz.log | grep SftpDeliveryJob
   ```
3. If next cycle succeeds → done. If next cycle also fails → escalate.

### Recovery D: Heap exhausted
1. Check current batch size in `appsettings.Production.json`:
   ```bash
   jq '.Campaigns.BatchSize' /etc/campaigns/appsettings.Production.json
   ```
2. If above 5000 — temporarily reduce:
   ```bash
   sudo /usr/local/bin/campaigns-config set Campaigns.BatchSize=2500
   sudo systemctl restart campaigns-scheduler
   ```
3. File ticket to investigate root cause (a 5000-batch should not OOM — see `docs/postmortems/2026-03-batch-oom.md`).

## Verification

After any recovery path:

1. Queue should drain within 2 cycles (~10 min):
   ```sql
   SELECT COUNT(*)
   FROM SCH_CAMPAGINES.T_CAMPAGINES_LOG
   WHERE STATUS = 'QUEUED'
     AND CREATED_AT < DATEADD(MINUTE, -10, GETUTCDATE());
   ```
   Expected: `0` (or steadily decreasing on repeated query).

2. Grafana alert `CampaignDeliveryLag` clears within 15 min.

3. Spot-check one campaign end-to-end:
   ```sql
   SELECT TOP 1 CAMPAIGN_ID, STATUS, DELIVERED_AT
   FROM SCH_CAMPAGINES.T_CAMPAGINES_LOG
   WHERE STATUS = 'DELIVERED'
   ORDER BY DELIVERED_AT DESC;
   ```
   `DELIVERED_AT` should be within the last 5 min.

## Escalation

- Within 15 min of incident start without resolution → page on-call lead via PagerDuty (`Campaigns > P1`)
- If partner-side issue confirmed → notify business owner: jane.doe@company.com (+CC `#campaigns-business`)
- If customer-visible impact confirmed → trigger Status Page incident (procedure: `docs/runbooks/status-page.md`)

## Post-incident

1. Add a comment to the incident channel summarizing root cause + fix.
2. If this is a new failure mode → add a new runbook entry or extend this one.
3. If recovery took > 30 min → schedule a post-mortem (template: `docs/templates/postmortem.md`).
4. Open a ticket if a permanent fix is needed (e.g., automate Recovery C if it happens > 1×/week).

## Related

- ADR-014: Quartz configuration from database
- `docs/postmortems/2026-03-batch-oom.md` — last OOM incident
- `docs/integrations/sftp-partner.md` — partner contact, contractual SLA
- Runbook: `quartz-job-stuck.md`
- Runbook: `database-connection-pool-exhausted.md`
````

This template looks long because it's *complete*. Real entries shorter than this exist (a routine credential rotation might be 30 lines), but the shape — symptoms, pre-checks, diagnosis, recovery, verification, escalation, post-incident — stays the same.

---

## Common Entry Categories

A mature runbook tends to grow these categories naturally. Use them as a checklist when bootstrapping.

### 1. Incident Recovery (the classic runbook)

One entry per known failure mode. Triggered by an alert or a user report.

Examples:
- `sftp-delivery-stuck.md`
- `database-connection-pool-exhausted.md`
- `quartz-job-stuck.md`
- `disk-space-low-on-app-server.md`
- `redis-memory-evicting.md`

### 2. Routine Maintenance (cron-eligible but often manual)

Procedures done on a schedule. Document them so they survive the original author leaving.

Examples:
- `quarterly-credential-rotation.md`
- `monthly-archive-cleanup.md`
- `weekly-failed-job-replay.md`
- `tls-certificate-renewal.md`

### 3. Deployment & Release

How to ship code safely.

Examples:
- `release-procedure.md` — full release walkthrough
- `rollback-procedure.md` — when a release goes bad
- `database-migration-procedure.md` — migrations that need coordination
- `feature-flag-rollout.md` — gradual rollout protocol

### 4. Backup & Restore

How to back up, how to restore, how to test that restore works.

Examples:
- `database-backup-schedule.md`
- `database-restore-from-backup.md`
- `disaster-recovery-drill.md`

### 5. Monitoring & Observability

What to watch and how to interpret it.

Examples:
- `dashboards-overview.md` — what each Grafana board shows
- `alerts-reference.md` — every alert, what it means, which runbook it points to
- `log-locations.md` — where every component's logs live
- `tracing-quickstart.md` — how to pull a distributed trace

### 6. Access & Onboarding (operations-side)

Not application onboarding (that's `ONBOARDING.md`) — *operational* access.

Examples:
- `granting-production-access.md`
- `vpn-setup-for-oncall.md`
- `secrets-management.md`

### 7. Diagnostic Recipes

Not tied to a specific incident — general "how do I figure out X" recipes.

Examples:
- `how-to-trace-a-stuck-campaign.md`
- `how-to-find-the-slow-query.md`
- `how-to-correlate-logs-across-services.md`

---

## Organization: Single File vs Folder Structure

There is no universally right answer. It depends on how big your runbook grows and how often people search it.

### Single `OPERATIONS.md` (early stage)

Good for projects with fewer than ~15 distinct procedures.

**Pros:**
- Everything is searchable in one Cmd+F
- Easy to keep the Table of Contents accurate
- Less navigation overhead
- Easy for an AI agent to load fully into context

**Cons:**
- Becomes unwieldy past ~600 lines
- Diff noise grows (every change touches the same file)
- Hard to assign per-section ownership

### Folder `docs/runbooks/` with index (mature stage)

Good once you have 15+ procedures and multiple owners.

```
docs/
├── OPERATIONS.md                    # index + global routines
└── runbooks/
    ├── README.md                    # alphabetical index with one-line descriptions
    ├── sftp-delivery-stuck.md
    ├── quartz-job-stuck.md
    ├── database-connection-pool-exhausted.md
    ├── disk-space-low.md
    ├── _template.md                 # blank entry to copy
    └── archived/                    # outdated procedures, kept for context
        └── 2025-old-load-balancer.md
```

**Pros:**
- Per-file ownership and per-file review
- Diffs stay scoped to the affected procedure
- Easier to assign "last verified" dates per entry
- New entries don't require rewriting one big TOC

**Cons:**
- Discovery requires good filenames and an index
- AI agents need pointers to load the right file
- Easier to let one entry rot without anyone noticing

### Transition strategy

Don't pre-organize. Start with `OPERATIONS.md`. When it becomes painful to navigate (typically around 500–700 lines or 10+ distinct procedures), do a single splitting refactor:

1. Create `docs/runbooks/`
2. Move each procedure to its own file, keeping the title as the filename slug
3. Replace each procedure in `OPERATIONS.md` with a one-line link
4. Add a `_template.md` and a `README.md` index to the folder

Do this once, when you feel the pain. Not in advance.

---

## Writing for AI Agents

The AI agent has two roles relative to your runbook:

1. **Consumer** — when you ask it to generate a diagnostic script, a recovery automation, or a "explain what's happening" summary, it reads your runbook to ground its answer in the system's reality.
2. **Author / co-author** — you ask it to draft a new entry, update an existing one, or extract lessons from a post-mortem into runbook form.

Both roles work best when the runbook is written for *machine readability as well as human readability*.

### Practices that help the agent

- **Stable, predictable section names.** "Symptoms / Diagnosis / Recovery / Verification" everywhere — not "What's broken" in one entry and "Issue description" in another. The agent learns the schema and uses it.
- **Code blocks with explicit language tags.** ` ```bash`, ` ```sql`, ` ```json` — not bare triple-backticks. This lets the agent extract and reuse commands cleanly.
- **Explicit expected outputs.** *"Expected: `{"status":"UP"}`"* — the agent can use this to generate matching tests or assertions.
- **Cross-references in `[[name]]` or full-path form.** *"see `docs/runbooks/quartz-job-stuck.md`"* — not *"see the Quartz runbook"*. The agent can follow a path; it can't follow an allusion.
- **No metaphors or insider jokes.** *"The Beast"* might be how the team refers to the legacy importer, but in the runbook call it `LegacyCsvImporter`. The agent doesn't know your nicknames.

### In `CLAUDE.md`, point the agent at the runbook

Add a section like:

```markdown
## Operational Procedures
Before generating any of the following, read the corresponding doc:
- Health checks → `OPERATIONS.md` § "Health Endpoints"
- Diagnostic scripts → `docs/runbooks/` (find by symptom)
- Restart / restore procedures → `docs/runbooks/`
- Deployment scripts → `OPERATIONS.md` § "Release Procedure"

Do NOT invent diagnostic commands. If a procedure isn't documented,
flag it as a missing runbook entry and propose adding one.
```

This single block prevents the agent from confidently hallucinating `systemctl restart quartz` when your actual service is `campaigns-scheduler.service`.

---

## Maintenance Discipline

A runbook rots faster than any other documentation. Code changes, services move, hostnames change, partners rotate keys — and the runbook silently lies until 3 a.m.

### The maintenance budget

Budget time explicitly:

- **After every production incident:** 15 minutes to update the relevant entry (or create one). If you skip this, the next on-call hits the same wall.
- **After every release that touches infrastructure:** 10 minutes to check whether any commands, paths, or hostnames in the runbook are now stale.
- **Quarterly:** a 1-hour walkthrough where someone *actually executes* one or two entries against staging. Mark them with the new "Last verified" date.
- **Annually:** archive obviously-dead entries to `docs/runbooks/archived/`. Don't delete — old entries are sometimes the only record of a system that lived once.

### Ownership

Every entry needs an owner — typically a team, not a person (people leave). Encode this in the entry header:

```markdown
**Owner:** Platform team (`#platform-oncall`)
**Last verified:** 2026-05-12
```

A runbook entry without an owner is dead the day its author changes jobs.

### The "last verified" discipline

Treat older-than-6-months entries as suspicious. Add a section to the runbook index:

```markdown
## Verification status

| Entry | Owner | Last verified | Status |
|-------|-------|---------------|--------|
| sftp-delivery-stuck | Platform | 2026-05-12 | OK |
| quartz-job-stuck | Platform | 2025-11-03 | Stale — re-verify Q3 |
| disk-space-low | Platform | 2024-08-15 | Stale > 12mo — owner please verify |
```

Stale-but-present is more dangerous than missing. Missing makes you investigate; stale-but-present makes you trust a lie.

### After a near-miss

Every time you say *"good thing I remembered to check X — that wasn't in the runbook"*, write it down before you forget. Near-misses are the most valuable runbook source — better than incidents, because no one was woken up to learn the lesson.

---

## Working with the Agent: Practical Commands

The runbook lives or dies by use. Here are prompts for the recurring situations.

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
Read docs/postmortems/2026-05-incident-X.md.

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
Read docs/runbooks/sftp-delivery-stuck.md.

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

This is the operational equivalent of "tests must accompany the feature." A new background job without a runbook entry is a 3 a.m. waiting to happen.

---

## Anti-Patterns

What makes runbooks fail in practice:

### 1. The "vibes-based" runbook
Entries that say *"check the logs"* without specifying *which* log, *what* to grep for, and *what* a healthy match looks like. The on-call ends up tailing six log files at random. Useless under stress.

### 2. Architecture in the runbook
Entries that explain *why* the system was designed a certain way. This belongs in `ARCHITECTURE.md` or an ADR. In the runbook, it pads the entry and delays the on-call from reaching the fix.

### 3. The append-only runbook
Every new failure gets a new entry. Old entries with stale commands stay forever. After two years the runbook is 80% lies and the team has learned to ignore it. The cure: explicit "Last verified" dates and quarterly walkthroughs.

### 4. The runbook nobody owns
Written once by someone who is now in a different team. Status: unknown. Recovery commands: unknown if they still work. Cure: assign team-level ownership at creation; review owners annually.

### 5. The runbook with no symptoms section
The entry is titled *"Restart Quartz scheduler"* — but the on-call doesn't know whether the symptom they're seeing means they should restart Quartz. Entries should be findable by what the on-call *sees*, not by what the engineer who wrote it thinks the fix is.

### 6. Untested entries
The procedure was correct *when it was written* but the system changed and no one re-ran it. The cure: the quarterly "actually execute one entry against staging" ritual.

### 7. Embedded secrets
Runbook entries with credentials, tokens, or internal URLs that are sensitive. The runbook lives in version control — secrets belong in your secret manager, with the runbook pointing to *how to retrieve them* (`vault read secret/campaigns/sftp/prod`), not embedding them.

### 8. Procedures that require tribal knowledge
*"Restart the cluster the usual way."* What's the usual way? The runbook exists precisely so that the *not-usual* engineer can do it.

### 9. Auto-generated runbooks from AI
You let the agent generate the runbook from "what it knows about the system" without verifying each command. Result: confident-sounding lies. Treat agent-drafted runbook entries the same as agent-drafted ADRs — *draft only, you verify before saving*.

### 10. Runbook entries triggered by alerts that no longer exist
Every alert in your monitoring should map to a runbook entry, and every runbook entry triggered by an alert should reference the alert by name. Cure: keep an `alerts-reference.md` that links each alert to its runbook entry and break the build (or just a periodic check) when one references a missing other.

---

## Golden Rules

1. **The runbook earns its place by being the first thing you open when something is wrong.** If it's not actionable under stress, it's not a runbook — it's prose.

2. **Symptoms first, fixes second.** Entries are found by what the on-call *sees*, not by what the author *knows*.

3. **Exact commands, real paths, copy-pasteable queries.** "Check the logs" is not a runbook step.

4. **Every entry has an owner and a "last verified" date.** Stale-but-present is more dangerous than missing.

5. **15 minutes after every incident.** Update or create the relevant entry while it's fresh. Skip this and your team relearns the same lesson next quarter.

6. **One file until it hurts, then split.** Start with `OPERATIONS.md`. Move to `docs/runbooks/` when navigation becomes the bottleneck — not before.

7. **Architecture, decisions, and specs do not belong in the runbook.** Link to them; do not duplicate them.

8. **Every new feature with operational surface ships with its runbook entry.** Background jobs, external integrations, persistent state — none of them are "done" without operational documentation.

9. **The agent drafts; you verify.** Auto-generated runbook entries with un-verified commands are worse than no runbook.

10. **Test your runbook before reality does.** A procedure no one has executed in a year is a hypothesis, not a procedure.

---

*This guide complements [`spec-driven-development-guide.md`](spec-driven-development-guide.md), which covers the design and decision layers (CLAUDE.md, ARCHITECTURE.md, ADRs, specs). The runbook is the operational layer that keeps those decisions running.*
