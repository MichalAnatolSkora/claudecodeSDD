# Example: the Order Export platform

> A complete, **illustrative** spec-driven paper trail for one small fictional app — so you can see how all the documents fit together around a single system, not in isolated snippets. **Docs only: no runnable code here** — the app is the *subject* the documents describe. Code appears only as short snippets inside the spec / plan, the way the guides show it.

This is the **golden thread**: a PRD's goals flow into feature slices, one slice becomes a `spec → plan → tasks` trio, the trio produces an ADR, and the ADR lands in `CLAUDE.md`'s active-decisions index. Follow it top to bottom.

## Read it in this order

1. **[`docs/prd/2026-01-order-export.md`](docs/prd/2026-01-order-export.md)** — *why* the platform exists, its goals, and the feature breakdown. Note feature 3.
2. **[`DOMAIN.md`](DOMAIN.md)** + **[`ARCHITECTURE.md`](ARCHITECTURE.md)** — the vocabulary (HDR / DTL / RDY) and the shape of the system.
3. **[`CLAUDE.md`](CLAUDE.md)** — the conventions the agent follows; see its *Active decisions* list.
4. The trio for feature 3 — **[`specs/2026-02-delivery-retry/`](specs/2026-02-delivery-retry/)**, written in order:
   - [`spec.md`](specs/2026-02-delivery-retry/spec.md) — goal + 7 acceptance criteria (the test contract)
   - [`plan.md`](specs/2026-02-delivery-retry/plan.md) — the technical *how*, and the decision that became ADR-015
   - [`tasks.md`](specs/2026-02-delivery-retry/tasks.md) — ordered steps; the Verification table maps every AC to a task
5. **[`docs/adr/ADR-015-delivery-retry-backoff.md`](docs/adr/ADR-015-delivery-retry-backoff.md)** — the decision the plan made — then find it again in `CLAUDE.md` § Active decisions. Thread closed.
6. **[`TESTING.md`](TESTING.md)** — the conventions those tests follow.

## Trace the thread

| You see… | …pointing at | which shows |
|----------|--------------|-------------|
| PRD goal G3 "resilience" | feature 3 in the breakdown | goals drive features |
| `spec.md` acceptance criteria | `tasks.md` Verification table | every AC has a task |
| `plan.md` "use Polly" decision | `ADR-015` | hard decisions become ADRs |
| `ADR-015` | `CLAUDE.md` § Active decisions | the ADR list is the agent's gate |

## Which guide explains each file

| File | Guide |
|------|-------|
| `docs/prd/…` | [`prd-guide.md`](../../guides/prd-guide.md) |
| `spec.md` / `plan.md` / `tasks.md` | [`spec-plan-tasks-guide.md`](../../guides/spec-plan-tasks-guide.md) ★ |
| `CLAUDE.md` | [`claude-md-guide.md`](../../guides/claude-md-guide.md) |
| `ADR-015` | [`adr-guide.md`](../../guides/adr-guide.md) |
| `TESTING.md` | [`testing-guide.md`](../../guides/testing-guide.md) |
| the order you run it all in | [`flow-guide.md`](../../guides/flow-guide.md) |

## Notes

- **The full spec picture** — which PRD features have specs and which are still backlog — is in [`specs/README.md`](specs/README.md).
- **Only ADR-015 is included in full.** `CLAUDE.md` lists ADR-001/003/007 too — deliberately: it shows *Active decisions* is a curated index of what's in force, not a copy of every ADR.
- **The stack is illustrative** (.NET / Dapper / Quartz / SFTP) — the patterns transfer to any stack.
- The product, partners (`acme-bank`), and data are fictional.

---
*Part of the [SDD guides repo](../../README.md). The method itself starts at [`spec-plan-tasks-guide.md`](../../guides/spec-plan-tasks-guide.md).*
