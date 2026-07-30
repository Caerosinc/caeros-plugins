---
name: crm-pipeline
description: Manage the sales pipeline in the CRM (Twenty) — deals and opportunities, stage moves, amounts, close dates, forecasts, and pipeline reports. Use for anything about deal flow, revenue, win/loss, or "what's closing".
---

# CRM Pipeline

Read the `crm` skill first for conventions and filter syntax. This workspace
has both **opportunities** (standard) and **deals** (custom object); check
`Get Object Metadata` for which one the team actually works in before writing,
and ask the user if both are populated and the target is ambiguous.

## Core fields

- `name`, `amount` (currency composite: `amountMicros` = value × 1,000,000,
  plus `currencyCode`), `closeDate` (ISO 8601), `stage` (select — read the
  exact option values from the tool schema or object metadata; never invent a
  stage name), `companyId`, `pointOfContactId` (person id).

## Common operations

- **Find a deal**: `Search opportunities` / `Search deals` by name or
  `companyId[eq]:<id>`.
- **Create**: search the company first (`crm-records`), link via `companyId`,
  set stage to the pipeline's entry stage and a realistic `closeDate`.
- **Move stage**: `Update opportunity`/`Update deal` with the new `stage`
  value. On close-won/close-lost, also confirm final `amount` and
  `closeDate`, and offer to log a note recording the outcome.
- **Bulk hygiene**: plural `Update` tools for batch changes (e.g. push all
  stale close dates); `Upsert` for syncing from an external source.

## Pipeline reporting

Answer reporting questions with `Group` calls, not by fetching every record:

- Pipeline by stage: `Group opportunities` by `stage`, aggregating count and
  sum of `amount.amountMicros` (divide by 1,000,000 for display).
- Closing this month: filter
  `closeDate[gte]:<month-start>,closeDate[lt]:<next-month-start>` and group
  or list.
- Win rate: group by stage over a closed-date range and compare won vs lost.
- Per-owner or per-company breakdowns: group by the relation id field, then
  resolve names with a batched Find/Search.

Present amounts in the record's `currencyCode`; do not mix currencies in one
total without flagging it.

## Timeline and context

`Search timeline activities` filtered by the deal/opportunity id reconstructs
history (stage changes, notes, emails). Use it before summarizing "where does
this deal stand". Link prep work to the deal with note/task targets
(`crm-productivity`).
