---
name: aggregate-spreadsheets
description: "Aggregate (append) 3 or more Excel spreadsheets from a OneDrive or local folder into a single summary<dateRun>.xlsx file. Use for demand-planning data ingestion when the user says 'aggregate the spreadsheets', 'append these xlsx files', 'combine the retailer catalogs', 'roll up the OneDrive folder', or asks for a summary workbook across multiple source files. Supports scheduled runs (e.g., 'every Friday'). Produces a review-ready summary with a Source File column; the raw source files are never modified."
---

# /aggregate-spreadsheets — Append spreadsheets into a summary workbook

Combine every `.xlsx` in a folder (a synced OneDrive folder or a local path) into one
`summary<dateRun>.xlsx`. Built for the Demand Planning "Ingesting & Aggregating Data"
flow, where multiple retailer catalogs (BestBuy, Walmart, Costco, …) are appended into a
single review-ready workbook.

## Core principles

- **Read-only on sources.** Never edit, move, or overwrite the source spreadsheets.
- **Append, don't merge-by-key.** Rows are stacked; a `Source File` column preserves origin.
- **No totals in the summary.** Blank rows and leftover total/subtotal rows are dropped automatically.
- **Deterministic output name.** `summary<dateRun>.xlsx`, where `<dateRun>` is the run date.
- **Fail loud on structural drift.** Warn (or stop, with `--strict`) when a file's columns differ.
- **3 or more files.** Works with any count ≥ 2; the demand-planning demo uses 3.

## Inputs to confirm with the user

1. **Input folder** — the OneDrive-synced local path (e.g.
   `C:\Users\<you>\OneDrive - Microsoft\SurfaceCatalogs`) or a repo path like `.\spreadsheets`.
   On Windows, a synced OneDrive folder is just a normal local path — use it directly.
2. **Output folder** — defaults to the input folder.
3. **Schedule** (optional) — e.g. "every Friday". If requested, set up a recurring run
   rather than a one-off (see *Scheduling* below).

## Procedure

1. **Resolve the folder.** Confirm the OneDrive/local path exists and list the `.xlsx`
   files, skipping any existing `summary*.xlsx` and Excel lock files (`~$*`).
2. **Validate columns.** Read the first file as the reference schema; compare the rest.
   Report any missing/extra columns. Stop if the user asked for strict matching.
3. **Clean.** Drop fully blank rows and total/subtotal rows (first-cell markers like
   "Total", "Average", "Subtotal", "Total Units In Stock", "Average Price").
4. **Append.** Concatenate all rows, adding a `Source File` column as the first column.
5. **Write.** Save `summary<dateRun>.xlsx` with a formatted header row, frozen header,
   and an auto-filter. Do not add total rows to the summary.
6. **Report.** State the output path, total rows, column count, and sources combined.

## Preferred execution

Use the bundled script (no bespoke code needed):

```bash
python tools/spreadsheet-aggregator/aggregate_spreadsheets.py \
    --input "<OneDrive or local folder>" \
    --output "<optional output folder>" \
    --date-format %Y-%m-%d \
    [--strict]
```

- `<dateRun>` comes from `--date-format` (strftime). Use `%Y-%m-%d_%H%M` to avoid
  overwriting when running multiple times a day.
- The script requires `pandas` and `openpyxl` (`pip install pandas openpyxl`).

If you must do it inline instead of the script, follow the same clean → append →
`Source File` → formatted-header rules, and never write totals into the summary.

## Scheduling (e.g., "aggregate every Friday")

When the user wants a recurring roll-up:

- **In Copilot CLI / Cowork**, schedule a recurring prompt (e.g., a Friday cron like
  `0 17 * * 5`) whose body is: *"Aggregate every .xlsx in `<OneDrive folder>` into
  summary<dateRun>.xlsx using the aggregate-spreadsheets skill."*
- **OS-level fallback**: wrap the script in Windows Task Scheduler (weekly, Friday) or a
  cron job calling the same command.
- Each run produces a dated file, giving you a weekly history of summaries.

## Verification checklist

- [ ] Output file name is exactly `summary<dateRun>.xlsx`.
- [ ] Row count equals the sum of source data rows (minus totals/blanks).
- [ ] `Source File` column present and correctly attributes every row.
- [ ] No total/subtotal/blank rows carried into the summary.
- [ ] Source files unchanged on disk.

## Installing this as a skill

Copy this folder's `SKILL.md` (and keep the script path reachable) into
`~/.copilot/skills/aggregate-spreadsheets/SKILL.md` to make it discoverable as a
first-class Cowork/CLI skill. The skill references the repo script at
`tools/spreadsheet-aggregator/aggregate_spreadsheets.py`.
