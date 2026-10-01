# Prompts — Demand Planning: Ingesting & Aggregating Data

Copy-paste prompts for each flow in the demo. Replace `<...>` placeholders with your values.

---

## Flow 1 — Chat: aggregate the 3 local `.xlsx` files

> Aggregate the data in the three spreadsheets in the `spreadsheets` folder
> (`BestBuy.xlsx`, `Walmart.xlsx`, `Costco.xlsx`) into a single summary workbook.
> Append the rows (don't merge by key), add a `Source File` column so I can tell which
> retailer each row came from, and drop any blank or total rows. Save it as
> `summary<dateRun>.xlsx` in the `spreadsheets` folder, where `<dateRun>` is the current date
> and time (e.g. `2026-10-01_1408`) so multiple runs a day don't overwrite each other.
> Confirm the columns match across all three files first.

**Shorter version:**

> Append `BestBuy.xlsx`, `Walmart.xlsx`, and `Costco.xlsx` from `.\spreadsheets` into one
> `summary<dateRun>.xlsx` (name includes the current date and time, e.g. `2026-10-01_1408`),
> keeping a single header row and adding a `Source File` column.

---

## Flow 2 — Cowork: aggregate a OneDrive folder (one-off)

> Using the `aggregate-spreadsheets` skill, aggregate every `.xlsx` in my OneDrive folder
> `<C:\Users\<you>\OneDrive - Microsoft\SurfaceCatalogs>` into `summary<dateRun>.xlsx`
> (name includes date + time, e.g. `2026-10-01_1408`) saved in a `SkillSummary` subfolder.
> Skip any existing `summary*.xlsx`, drop total/blank rows, add a `Source File` column, and
> tell me the row count, column count, and which files were combined. Don't modify the source
> files.

**No-skill equivalent (write to `PromptSummary` instead):**

> In my OneDrive folder `<folder>`, find every `.xlsx` (skip names starting with `summary`),
> confirm the columns match, and append all rows into `summary<dateRun>.xlsx` (date + time in
> the name) inside a `PromptSummary` subfolder (create it if needed). Add a `Source File`
> column, keep one header row, drop blank/total rows, don't modify the sources, and report the
> row count and files combined.

**Script-based equivalent:**

> Run:
> `python tools/spreadsheet-aggregator/aggregate_spreadsheets.py --input "<OneDrive folder>" --output "<OneDrive folder>\SkillSummary"`
> then show me a summary of the result.

---

## Flow 2 (scheduled) — Cowork: aggregate the OneDrive folder every Friday

> **Reminder:** before running this prompt, point/attach Cowork to your OneDrive
> `SurfaceCatalogs` folder (the folder chip) so the scheduled task knows where to look. Easy to
> forget on the first run.

> Set up a recurring Cowork task that runs **every Friday at 5pm**. Each run should use the
> `aggregate-spreadsheets` skill to append every `.xlsx` in my OneDrive folder
> `<OneDrive folder path>` into a **new** `summary<dateRun>.xlsx` whose name includes the date
> **and** the time down to the minute (e.g. `2026-10-01_1700`), saved in the `SkillSummary`
> subfolder. **Always create a new file every run — never overwrite or refresh an existing
> summary, even if one already exists for today.** Skip prior `summary*.xlsx` files, drop
> total/blank rows, and add a `Source File` column. Post a short summary (row count + sources)
> after each run.

**Windows Task Scheduler alternative (one line):**

> Create a weekly Friday Task Scheduler job that runs:
> `python "<repo>\tools\spreadsheet-aggregator\aggregate_spreadsheets.py" --input "<OneDrive folder>" --output "<OneDrive folder>\SkillSummary"`

---

## Flow 3 — Cowork Prototype: read-only local app over the summary

> Build a **read-only** local app (runs on this machine, no writes to the data) that loads
> the most recent `summary*.xlsx` in `<output folder>` and lets me browse and filter the
> rows — filter by `Retailer`, `Product Line`, and `Memory (GB)`, sort by `Price (USD)`,
> and show totals/averages computed in the app (not written back to the file). Refresh from
> the newest summary file on launch. Do not modify any spreadsheet.

---

## Tips

- On Windows, a **synced OneDrive folder is a normal local path** — just point `--input` at it.
- Keep **totals out of the source files**; compute them in the summary/app instead.
- The default file name includes **date + time** (`%Y-%m-%d_%H%M`) so multiple runs a day don't
  overwrite; pass `--date-format %Y-%m-%d` if you want one file per day.
- No-skill prompts write to a `PromptSummary` subfolder; the skill writes to `SkillSummary`.
- Add `--strict` to hard-stop when a file's columns don't match the others.
