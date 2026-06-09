# CSV export format option per partner

> Feature 2 from the [PRD](../docs/prd/2026-01-order-export.md). One-file trio: spec / plan / tasks as three sections. **STATUS: shipped (PR #19, 2026-02).** Frozen.

## Spec — what & why

**Goal.** Some partners want CSV instead of XML. Let a partner's config choose the format; the export pipeline picks the matching formatter. Existing XML partners are untouched.

**In scope**
- A `format` column on `app.partner` (`xml` | `csv`), default `xml`
- `CsvWriter` renders HDR + DTL rows as CSV (the same data as the XML)
- The pipeline selects `BatchXmlMerger` or `CsvWriter` by partner config

**Out of scope**
- Per-partner column ordering / custom CSV dialects
- Any format beyond `xml` / `csv`

**Acceptance criteria**
- [x] AC1: a partner with `format=csv` receives a CSV file (HDR + DTL rows)
- [x] AC2: a partner with `format=xml` is unchanged (regression)
- [x] AC3: an unknown `format` value is rejected at config load with a clear error
- [x] AC4: the CSV HDR / DTL carry the same fields as the XML equivalent

## Plan — how

- Migration: add `format` to `app.partner` (`NOT NULL`, default `'xml'`)
- `CsvWriter` mirrors `BatchXmlMerger`'s `(orders, partner) → bytes` shape (pure)
- A small `FormatterSelector` maps `partner.format → formatter`; unknown → throw at startup
- Touch only: the migration, `CsvWriter.cs`, `FormatterSelector.cs`, the export handler

## Tasks — in what order

- [x] 1. Migration + `format` on `app.partner` → verify: applies on Testcontainers, defaults `xml`
- [x] 2. `CsvWriter` HDR + DTL → verify: unit test, fields match the XML sample
- [x] 3. `FormatterSelector` (xml / csv, unknown → throw) → verify: unit test incl. the reject case (AC3)
- [x] 4. Wire the selector into the export handler → verify: AC1–AC2 green (csv partner gets CSV, xml partner unchanged)

---
*Illustrative one-file trio (shipped).*
