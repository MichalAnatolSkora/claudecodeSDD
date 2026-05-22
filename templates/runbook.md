# Runbook: [Symptom — what the on-call sees first]

> Title by symptom, not by fix. On-call searches for "delivery stuck", not "restart job".

**Severity:** [High | Medium | Low — and one line on user impact]
**Last verified:** YYYY-MM-DD
**Owner:** [Team name + on-call channel]

## Symptoms

- [Alert name that fires]
- [Dashboard signal]
- [Log message or query result that confirms the symptom]

## Pre-checks (30 seconds)

1. [Quick sanity check — is this actually broken, or is a deploy in progress?]
2. [Partner / external dependency status check]

If a pre-check is positive → stop and document; do not start recovery.

## Diagnosis

### Step 1: [What you're checking]

```bash
[exact command]
```

Expected: `[exact expected output]`

If [condition] → jump to **Recovery A**.
If [other condition] → escalate.

### Step 2: ...

## Recovery

### Recovery A: [Cause]

1. [Step with exact command]
   ```bash
   [command]
   ```
2. ...

### Recovery B: [Other cause]

...

## Verification

After any recovery path:

1. [How to confirm the fix worked — a query or command with expected output]
2. [How long to watch before declaring resolved]

## Escalation

- [When to page on-call lead]
- [When to notify business owner]
- [When to trigger status-page incident]

## Post-incident

1. [Update incident channel]
2. [File ticket for permanent fix, if applicable]
3. [Schedule post-mortem if criteria met]

## Related

- ADR-NNN: [linked decision]
- `docs/postmortems/[file].md` — [prior incident]
- `docs/runbooks/[other].md` — [related procedure]
