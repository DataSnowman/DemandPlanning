# Surface Catalog Viewer

A small **read-only** Streamlit app that loads the most recent aggregated catalog summary
(`summary<YYYY-MM-DD_HHMM>.xlsx`) and lets you browse, filter, and sort the consolidated
retailer catalog. It **never writes** to the source data.

This is **Part 3** of the [Demand Planning demo](../../README.md): after Part 1/Part 2 produce a
`summary<dateRun>.xlsx`, this app turns it into a durable, shareable viewer.

## What it does

- Loads the **most recent** `summary*.xlsx` from `SUMMARY_FOLDER`. "Most recent" = the file
  whose name sorts last, because `summary<YYYY-MM-DD_HHMM>.xlsx` sorts chronologically as plain
  text (falls back to newest modified-time if a name has no timestamp).
- Shows which file is loaded, with row/column counts.
- Filters by **Retailer**, **Product Line**, and **Memory (GB)**.
- Sorts by **Price (USD)**.
- Shows **totals and averages computed in the app** (not written back to the file).
- **Reload** button re-scans the folder so you can pick up a newer summary without restarting.

## Run it

```powershell
pip install -r app/catalog-viewer/requirements.txt

# Point at your real summary folder (otherwise it uses the bundled sample_data):
$env:SUMMARY_FOLDER = "C:\Users\darsch\OneDrive - Microsoft\!SurfaceCatalogs\PromptSummary"

streamlit run app/catalog-viewer/app.py
```

- **`SUMMARY_FOLDER`** — folder containing the `summary*.xlsx` files. Defaults to
  `app/catalog-viewer/sample_data/` (a committed sample) so the app runs out of the box.
  - No-skill demo path: `…\!SurfaceCatalogs\PromptSummary`
  - Skill demo path: `…\!SurfaceCatalogs\SkillSummary`

## Notes

- Read-only: the app opens the workbook with pandas and never saves it.
- The sample workbook under `sample_data/` is generated from the three synthetic catalogs in
  `spreadsheets/`; regenerate it any time with:
  ```powershell
  python tools/spreadsheet-aggregator/aggregate_spreadsheets.py --input .\spreadsheets --output .\app\catalog-viewer\sample_data
  ```
