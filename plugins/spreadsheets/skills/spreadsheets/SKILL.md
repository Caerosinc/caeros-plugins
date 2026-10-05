---
name: spreadsheets
description: Use this skill when a user asks to create, modify, analyze, visualize, or verify spreadsheet files (.xlsx, .csv, .tsv) or Google Sheets-targeted workbooks with formulas, formatting, charts, and recalculation.
---

# Spreadsheets

Build and edit `.xlsx` workbooks yourself with Caeros's native `workbook_create_or_edit` tool, from quick edits to financial models, dashboards, and multi-sheet analyses. It sends typed operations to Caeros's pinned Office engine in one copy-on-write transaction, backs up an existing workbook, validates the package, and reopens it before replacing the original. Never call OfficeCLI through the shell and never edit OOXML directly. Apply the conventions below to any spreadsheet you produce.

**Opening/viewing an existing workbook is NOT a build task.** To show a workbook to the user, state its absolute file path on its own line in your reply: the app renders it as a clickable link that opens the built-in spreadsheet editor (sheet tabs, formula bar). To summarize contents, read it with `office_inspect` (`outline` for structure, `text` for cell values) and answer directly. Never use `read_file` or `read_text_file` on an XLSX.

## Authoring rules

- Author with `workbook_create_or_edit`, following the bundled `xlsx` skill's operation schema: cell paths like `/Sheet1/A1`, a new workbook's `/Sheet1` renamed to your first sheet before you add the others, one cell per `set` with `props.value` (singular) and `"type":"number"` for numbers, and the style keys `fill`, `halign`, and `numberformat`. Group a logical edit into one operations array; build a large workbook in a few batches in sheet order. Use `office_help` with `format="xlsx"` before an unfamiliar element or property; do not guess schema names.
- Formula text omits the leading `=`: send `{"formula":"B2-C2"}`, not `{"formula":"=B2-C2"}`. The formula examples below are written the way you send them.
- Use `pandas` for substantial data cleaning or aggregation before the Office transaction, and `import` under a worksheet for bulk CSV/TSV data. Use `openpyxl` only as a compatibility fallback when `workbook_create_or_edit` or `office_help` confirms the pinned engine cannot represent a required feature. For any Python step, write ONE script per task and patch/rerun it; do not scatter one-off snippets or heredocs.
- Derived values must be FORMULAS, never hardcoded results. Reference cells instead of magic numbers: `A5*(1+$A$6)`, not `A5*1.05`.
- Keep formulas simple and auditable; use helper cells for intermediate steps so a user can trace inputs → outputs.
- Cross-sheet references always quote the sheet name: `'Sheet Name'!A1`. Create every worksheet before writing formulas that reference it.
- Store real typed values (numbers, dates), not strings. Use locale-invariant number format codes: `#,##0`, `0.0%`, `"$"#,##0.00`, `yyyy-mm-dd`. Never swap `.` and `,` to mimic locales.
- Avoid volatile INDIRECT/OFFSET; avoid full-column ranges (`A:A`) inside SUMIFS/COUNTIFS; guard IRR/XIRR so templates never surface `#NUM!`.

## Workbook structure (analytical / financial workbooks)

Sheet flow: **Cover → Assumptions → Data → Model/Statements → Outputs/Valuation → Sensitivities → Checks → Sources**.

- **Cover**: title, generated date, fiscal basis, key outputs table (Metric | Value | Unit | Source), "what to look at first" guide, and the overall model status mirrored from Checks.
- **Assumptions**: every editable input in one place, styled distinctly.
- **Checks**: formula-driven assertions, one per row: `Check | Actual | Expected | Difference | Tolerance | Status` with `IF(ABS(diff)<=tol,"OK","Review")` and an aggregate status `IF(COUNTIF(status_range,"Review")=0,"OK","Review")`. Include tie-outs (balance sheet balances, FCF = OCF − capex, totals cross-foot) and sanity checks (WACC > terminal growth, prices positive).
- **Sources**: item, value, units, period/as-of date, source name and plain-text URL for every external input. Cite compact source IDs in data rows, full URLs here.

Styling conventions: blue font = editable inputs; black = formulas; green = links to other sheets; number columns right-aligned; borders above totals; hide gridlines on presentation sheets; freeze header panes; only style the used range.

## Verification (required before reporting completion)

A successful `workbook_create_or_edit` result already includes the authoritative package validation and reopen/inspection, so do not call `office_validate` after it and do not run a routine outline inspection. That validation does not evaluate formulas, so finish with this pass; do not claim success without it:

1. From the final mutation result's inspection, confirm every required sheet exists and contains data and that each requested chart exists. Check that the chart series ranges you sent cover the intended data (no off-by-one).
2. **Formula scan**: run `office_inspect` with `mode: "issues"` on the finished workbook, and once more after any fix batch. It evaluates formulas and reports error results (`#REF!`, `#DIV/0!`, `#VALUE!`, `#NAME?`, `#N/A`, `#NUM!`, `#NULL!`), formulas that reference a sheet that doesn't exist, broken defined names, and chart series that point at missing sheets. Fix every error result, missing-sheet reference, and broken name with another batch; do not suppress errors. A `formula_not_evaluated` entry means the engine could not compute that formula: check it for a misspelled function or bad syntax (Excel would show `#NAME?`), and keep it only when it is a valid Excel function the engine does not support.
3. Read back the Checks sheet (`office_inspect` with `mode: "text"`, which shows evaluated values) and confirm every status is "OK". If any row says "Review", fix the model, not the check.
4. In Caeros desktop, the app opens an editable workbook preview automatically; do not call `office_render` just to show it. In CLI/headless runs, call `office_render` once only when visual verification or an export is actually needed.

If the openpyxl fallback wrote the file, the native tool did not validate it: run `office_validate` once on the saved file and use `office_inspect` with `mode: "outline"` for step 1.

Report the workbook with its absolute path and a one-line summary of the scan/check results (e.g. "Formula scan found no errors; workbook checks are OK").

## Google Sheets handoff

For a native Google Sheets deliverable: build and verify a local `.xlsx` first, then upload it via the connected Google Drive app (`apps_execute_tool`, Drive upload with the local path — Caeros stages the file). Do not build sheet-by-sheet through Sheets write APIs; the locally verified XLSX is the interchange format.
