---
name: spreadsheets
description: Use this skill when a user asks to create, modify, analyze, visualize, or verify spreadsheet files (.xlsx, .csv, .tsv) or Google Sheets-targeted workbooks with formulas, formatting, charts, and recalculation.
---

# Spreadsheets

For substantial workbook builds (financial models, dashboards, multi-sheet analyses), delegate to the `cae_excel` specialist via the task tool — it plans, builds, and is blocked from finishing until the workbook passes validation. Use this skill directly for quick edits, CSV work, and to enforce the conventions below on any spreadsheet you produce.

**Opening/viewing an existing workbook is NOT a build task — never delegate it to cae_excel.** To show a workbook to the user, state its absolute file path on its own line in your reply: the app renders it as a clickable link that opens the built-in spreadsheet editor (sheet tabs, formula bar). To summarize contents, read it with `analyst_read_table_file` and answer directly.

## Authoring rules

- Author with Python (openpyxl; pandas for data prep). Write ONE builder script per task and patch/rerun it — do not scatter one-off snippets or heredocs.
- Derived values must be FORMULAS, never hardcoded results. Reference cells instead of magic numbers: `=A5*(1+$A$6)`, not `=A5*1.05`.
- Keep formulas simple and auditable; use helper cells for intermediate steps so a user can trace inputs → outputs.
- Cross-sheet references always quote the sheet name: `='Sheet Name'!A1`. Create every worksheet before writing formulas that reference it.
- Store real typed values (numbers, dates), not strings. Use locale-invariant number format codes: `#,##0`, `0.0%`, `"$"#,##0.00`, `yyyy-mm-dd`. Never swap `.` and `,` to mimic locales.
- Avoid volatile INDIRECT/OFFSET; avoid full-column ranges (`A:A`) inside SUMIFS/COUNTIFS; guard IRR/XIRR so templates never surface `#NUM!`.

## Workbook structure (analytical / financial workbooks)

Sheet flow: **Cover → Assumptions → Data → Model/Statements → Outputs/Valuation → Sensitivities → Checks → Sources**.

- **Cover**: title, generated date, fiscal basis, key outputs table (Metric | Value | Unit | Source), "what to look at first" guide, and the overall model status mirrored from Checks.
- **Assumptions**: every editable input in one place, styled distinctly.
- **Checks**: formula-driven assertions, one per row: `Check | Actual | Expected | Difference | Tolerance | Status` with `=IF(ABS(diff)<=tol,"OK","Review")` and an aggregate status `=IF(COUNTIF(status_range,"Review")=0,"OK","Review")`. Include tie-outs (balance sheet balances, FCF = OCF − capex, totals cross-foot) and sanity checks (WACC > terminal growth, prices positive).
- **Sources**: item, value, units, period/as-of date, source name and plain-text URL for every external input. Cite compact source IDs in data rows, full URLs here.

Styling conventions: blue font = editable inputs; black = formulas; green = links to other sheets; number columns right-aligned; borders above totals; hide gridlines on presentation sheets; freeze header panes; only style the used range.

## Verification (required before reporting completion)

Run a verify pass on the saved file — do not skip it, do not claim success without it:

1. Reopen the file (`openpyxl.load_workbook(path)`) and confirm every required sheet exists and contains data.
2. **Formula scan**: iterate all cells; flag any value or cached result in {`#REF!`, `#DIV/0!`, `#VALUE!`, `#NAME?`, `#N/A`, `#NUM!`, `#NULL!`} and any formula referencing a sheet that doesn't exist. (Caeros also enforces a recalculating formula scan server-side — a workbook with formula errors will be rejected, so fix them, don't suppress them.)
3. Read back the Checks sheet and confirm every status is "OK". If any row says "Review", fix the model, not the check.
4. Confirm charts exist where requested and ranges cover the intended data (no off-by-one).

Report the workbook with its absolute path and a one-line summary of the scan/check results (e.g. "Formula scan found no errors; workbook checks are OK").

## Google Sheets handoff

For a native Google Sheets deliverable: build and verify a local `.xlsx` first, then upload it via the connected Google Drive app (`apps_execute_tool`, Drive upload with the local path — Caeros stages the file). Do not build sheet-by-sheet through Sheets write APIs; the locally verified XLSX is the interchange format.
