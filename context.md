# Project Context — Surface Catalog Aggregation Demo

_Last updated: 2026-10-01_

## Overview

This repo is a **Demand Planning demo for Microsoft Copilot (Chat + Cowork)**. It ingests
synthetic retailer catalog spreadsheets for the soon-to-launch **Surface Laptop Ultra** and
**Surface RTX Spark Dev Box**, then aggregates them into a single timestamped summary workbook.

The flow is documented as a step-by-step end-to-end demo in the root `README.md`.

## What's in the repo

| Path | Purpose |
|------|---------|
| `README.md` | Single source-of-truth E2E demo guide (Parts 1–3, screenshots, reference). |
| `spreadsheets\BestBuy.xlsx` | Synthetic BestBuy catalog — 13 SKUs, 21 columns. |
| `spreadsheets\Walmart.xlsx` | Synthetic Walmart catalog — 13 SKUs, 21 columns. |
| `spreadsheets\Costco.xlsx` | Synthetic Costco catalog — 13 SKUs, 21 columns. |
| `tools\spreadsheet-aggregator\aggregate_spreadsheets.py` | Aggregation engine. |
| `tools\spreadsheet-aggregator\SKILL.md` | Cowork custom skill `aggregate-spreadsheets`. |
| `tools\spreadsheet-aggregator\PROMPTS.md` | Copy-paste prompts for each demo flow. |
| `docs\cowork-concept-product-launch.png` | Concept slide. |
| `docs\part1-chat-output.png` | Part 1 Chat output screenshot. |
| `docs\part2-cowork-output.png` | Part 2 Cowork output screenshot. |

## Data schema

**21 identical columns** across all 3 catalogs (append-ready, no total rows):

`Retailer, SKU, Product Line, Model Name, Form Factor, Processor, Memory (GB),
Memory Type, Storage, GPU, AI Performance, Interconnect, Screen Size (in), Weight (lb),
Color, Operating System, Price (USD), Units In Stock, Availability Date,
Customer Rating, Warranty`

The aggregated summary prepends a **`Source File`** column → 22 columns, 39 rows (13 × 3).

## Product specs

- **Surface Laptop Ultra**: Snapdragon X Elite, 14.5", 32/64/128 GB, 45 TOPS NPU, Windows 11 Pro.
- **Surface RTX Spark Dev Box (NVIDIA GB10)**: 20-core NVIDIA Grace ARM CPU (TSMC 3nm, co-dev
  MediaTek), Blackwell RTX (up to 6,144 CUDA cores, 5th-gen Tensor Cores), up to 1 PFLOP FP4,
  up to 128 GB unified LPDDR5X (lower tier capped at 64 GB), NVLink-C2C, 18–80W, 14–16" chassis
  ~14mm ~3 lb, Windows 11 ARM. SKUs use 64/128 GB only.

Reference links:
- https://www.microsoft.com/en-us/surface/devices/surface-laptop-ultra
- https://www.microsoft.com/en-us/surface/devices/surface-rtx-spark-dev-box

## Aggregation script behavior

`aggregate_spreadsheets.py` args: `--input` (required), `--output`, `--date-format`
(default `%Y-%m-%d_%H%M`), `--sheet-name` (default `Summary`), `--strict`.

- Non-recursive `glob` for `*.xlsx`; skips names starting with `summary` and `~$` lock files
  → output subfolders are safely ignored on reruns.
- Auto-creates output folder (`os.makedirs(exist_ok=True)`).
- Drops blank + total rows (via `TOTAL_MARKERS`); inserts `Source File` as first column.
- Warns on column mismatch (or `SystemExit` with `--strict`).
- **Timestamp default** `%Y-%m-%d_%H%M` → e.g. `summary2026-10-01_1408.xlsx`, allowing multiple
  runs per day. Pass `--date-format %Y-%m-%d` for one file per day.

## Conventions

- **No-skill prompts** write the summary to a `PromptSummary\` subfolder of the source folder.
- **The skill** writes to a `SkillSummary\` subfolder of the source folder.
- All summary filenames are timestamped (date + time) to support multiple runs per day.

## Demo structure (README)

- **Part 1 — Copilot Chat**: Option A (attach files) / Option B (reference a folder).
- **Part 2 — Cowork**: Step 1 OneDrive setup, Step 2 no-skill prompt → `PromptSummary`,
  Step 3 check results + screenshot, Step 4 Friday 5pm schedule, then an *optional* upgrade
  to register the reusable custom skill → `SkillSummary`.
- **Part 3 — prototype app** flow.
- Plus Troubleshooting and a Reference section (file map, script options, columns, specs, links).

## Known environment notes

- **LibreOffice is NOT installed** on this Windows machine; `scripts/recalc.py` fails
  (`socket.AF_UNIX` missing). Source catalogs intentionally contain **no formulas/totals**, so
  recalculation is unnecessary.
- **openpyxl append gotcha**: appending data rows before writing the header overwrites the first
  data row. Fix: `ws.append(HEADERS)` first, then data rows.

## Cowork scheduling (researched)

- Natural-language scheduled prompts; manage in **My tasks → Scheduled** tab.
- ~10 active scheduled tasks max; no one-click "schedule a past run" button in Preview.
- You can reuse the Step 2 prompt and add timing (e.g., every Friday at 5pm).
