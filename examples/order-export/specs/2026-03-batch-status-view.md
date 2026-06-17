# Batch status view — what needs a human

> Feature 5 from the [PRD](../docs/prd/2026-01-order-export.md). One-file trio, **discovery-shaped**: the acceptance criteria came out of a spike, not from up-front guessing. **STATUS: shipped (PR #41, 2026-03).** Frozen — later changes get a new spec.

> **Why discovery.** *"A view of batch status"* had no testable AC up front — nobody knew what the ops team actually needed to see until they looked at real batches. So we spiked first (see Tasks, step 0), let the AC emerge, then wrote this trio. See [`spec-plan-tasks-guide.md` § Two modes](../../../guides/spec-plan-tasks-guide.md#two-modes-delivery-and-discovery).

## Spec — what & why

**Goal.** Give the ops team one view of the export batches that need a human — which, after the spike, means **stuck** (orders still `RDY` more than 2h after the batch was created) or **partial** (some orders `SENT`, some still `RDY` in the same batch). Fully-sent batches don't appear.

**In scope**
- A read-only view of batches needing attention, with the signal (`stuck` / `partial`) and the count behind it
- Read from existing tables only (`order_batch`, `order_log`) — no schema change, no dependency on later features

**Out of scope**
- Fully-sent / healthy batches (a list of everything is exactly what we're replacing)
- Per-batch actions (resend / cancel buttons) — a later slice
- Real-time push; a manual refresh is fine

**Acceptance criteria**
- [x] AC1: a batch whose orders are still `RDY` more than 2h after it was created shows as `stuck`, with the hours-idle
- [x] AC2: a batch with some orders `SENT` and some still `RDY` shows as `partial`, with the unsent count
- [x] AC3: fully-sent batches are absent (not greyed out, not zero-count rows)
  - ~~AC1 (v1, WRONG): list all batches ordered by status, newest first~~
    **CHANGED during implementation (2026-03-04):** building it, the ops reviewer
    pointed out they'd still have to eyeball every row to find the bad ones — a list
    of statuses isn't a "needs attention" view. Replaced with the `stuck` / `partial`
    signals the spike had actually surfaced. The spec was still **Active**, so this
    was an in-place edit + a re-run of `/sdd-6-trio-check`, not a new spec.

## Plan — how

- One read-model query, no new tables: a batch is `stuck` if its orders are all `RDY` and `NOW() - created_at > INTERVAL '2 hours'`; `partial` if it has both `RDY` and `SENT` orders
- Read from `order_batch` + `order_log` (both exist since feature 1, the walking skeleton) — deliberately no dependency on feature 3's delivery tables, so this ships on the skeleton alone
- `GET /ops/batches/attention` → `BatchAttentionDto[]`; a minimal server-rendered table
- Touch only: `BatchAttentionRepository.cs`, `OpsController.cs`

## Tasks — in what order

- [x] 0. **Spike (throwaway, 2026-03-02):** dumped every batch with its order statuses and age into a scratch table and sat with an ops operator. Learned that *"needs attention"* = stuck or half-finished (`partial`); a flat status list was noise. Branch deleted. *(This is what produced the AC above — and showed the first one was wrong.)*
- [x] 1. Failing test: an all-`RDY` batch created 3h ago → `stuck` → verify: red (no query yet)
- [x] 2. `BatchAttentionRepository` query + unit tests for both signals → verify: AC1, AC2 green
- [x] 3. Wire `GET /ops/batches/attention`, exclude fully-sent → verify: AC3 green end to end

---
*Illustrative one-file discovery trio (shipped). The struck-through AC and the `CHANGED during implementation` note are kept on purpose — the messy reality of a first spec that was wrong is the point, not something to tidy away.*
