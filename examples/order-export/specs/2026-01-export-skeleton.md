# Export skeleton — one partner's orders to XML over SFTP

> Feature 1 (walking skeleton) from the [PRD](../docs/prd/2026-01-order-export.md). One-file trio: spec / plan / tasks as three sections. **STATUS: shipped (PR #12, 2026-01).** Frozen — later changes get a new spec.

## Spec — what & why

**Goal.** Prove the thinnest end-to-end path: take one partner's `RDY` orders, render them to the partner's XML file (HDR + DTL), deliver over SFTP, and record the batch. It hardcodes the boring parts (one partner, XML only, manual trigger); later slices thicken it.

**In scope**
- Load `RDY` orders for one configured partner (`acme-bank`)
- Render one HDR + one DTL per order to XML (`BatchXmlMerger`)
- Upload the file to the partner's SFTP
- On success, mark those orders `SENT` and write a batch row

**Out of scope**
- More than one partner, or partner config (later slices)
- CSV (feature 2), retry on failure (feature 3), scheduling (feature 4)

**Acceptance criteria**
- [x] AC1: `RDY` orders for the partner render to one HDR + N DTL rows in XML
- [x] AC2: the file is uploaded to the partner's SFTP path
- [x] AC3: on a successful upload the orders flip `RDY → SENT`
- [x] AC4: a batch row is recorded (partner, order count, timestamp)

## Plan — how

- `OrderLogRepository.GetReadyOrders(partnerId)` — Dapper, SQL in a `const string`
- `BatchXmlMerger` renders `(orders, partner) → XML bytes` (pure)
- `SftpDeliveryService.Upload(file, partner)` — straight upload, no retry yet
- A minimal-API endpoint `POST /export/{partner}` triggers it (manual; no scheduler yet)
- Touch only: `OrderLogRepository.cs`, `BatchXmlMerger.cs`, `SftpDeliveryService.cs`, `ExportController.cs`

## Tasks — in what order

- [x] 1. Failing integration test: `RDY` orders → expected XML (Testcontainers) → verify: fails (no merger yet)
- [x] 2. `BatchXmlMerger` HDR + DTL rendering → verify: unit test on a sample order set
- [x] 3. `OrderLogRepository.GetReadyOrders` + `MarkSent` → verify: repo test against Testcontainers
- [x] 4. `SftpDeliveryService.Upload` against a stub SFTP → verify: file lands on the stub
- [x] 5. Wire `POST /export/{partner}`, mark `SENT`, record the batch → verify: AC1–AC4 green end to end

---
*Illustrative one-file trio (shipped). The richer feature 3 uses the full three-file trio — see [`2026-02-delivery-retry/`](2026-02-delivery-retry/).*
