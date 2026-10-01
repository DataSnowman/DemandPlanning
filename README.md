# Demand Planning — Ingesting & Aggregating Data (E2E Demo)

A step-by-step demo that ingests retailer catalog spreadsheets for the **Surface Laptop
Ultra** and **Surface RTX Spark Dev Box**, then aggregates them into one summary workbook.

**The demo has three parts:**

| Part | Tool | What you do |
|------|------|-------------|
| 1 | **Copilot Chat** | Aggregate the 3 local `.xlsx` files in this repo |
| 2 | **Cowork** | Aggregate the spreadsheets sitting in a OneDrive folder (optionally on a Friday schedule) |
| 3 | **Cowork Prototype** | Build a read-only local app over the summary spreadsheet |

> The three catalogs (`BestBuy.xlsx`, `Walmart.xlsx`, `Costco.xlsx`) are **synthetic** demo
> data in the `spreadsheets\` folder. Product specs and source links are in
> [Reference: product specs](#reference-product-specs) at the bottom.

---

## Part 1 — Copilot Chat: aggregate the 3 files in this repo

**Goal:** append the three catalogs in `spreadsheets\` into one `summary<dateRun>.xlsx`.

**Step 1.** Open this repo in Copilot Chat.

**Step 2.** Paste this prompt:

```
Append the three spreadsheets in the `spreadsheets` folder (BestBuy.xlsx, Walmart.xlsx,
Costco.xlsx) into a single summary<dateRun>.xlsx in that same folder, where <dateRun> is
today's date. Keep one header row, add a "Source File" column so I can tell which retailer
each row came from, and drop any blank or total rows. Confirm the columns match across all
three files first, then show me the row count and which files were combined.
```

**Step 3.** Copilot creates `spreadsheets\summary<today>.xlsx`.

**Expected result:** 39 rows (13 per retailer), 22 columns (the 21 catalog columns + `Source
File`), no total rows. ✅

> **Under the hood (optional):** this is exactly what the bundled script does. You can run it
> directly instead of asking Chat:
> ```
> python tools/spreadsheet-aggregator/aggregate_spreadsheets.py --input spreadsheets
> ```

---

## Part 2 — Cowork: aggregate spreadsheets from a OneDrive folder

Same outcome as Part 1, but the source files live in a **OneDrive folder** and the work runs
in **Cowork** — optionally on a recurring **Friday** schedule.

### Step 1 — Put the spreadsheets in OneDrive

Pick (or create) a OneDrive folder and copy the catalogs into it, e.g.:

```
C:\Users\<you>\OneDrive - Microsoft\SurfaceCatalogs\
    BestBuy.xlsx
    Walmart.xlsx
    Costco.xlsx
```

You can drop in **3 or more** files — any `.xlsx` with the same columns will be included.
On Windows a synced OneDrive folder is just a normal local path, so Cowork reads it directly.

### Step 2 — Add the Cowork skill (one time)

This repo ships a reusable skill, `aggregate-spreadsheets`, at
`tools\spreadsheet-aggregator\SKILL.md`. Add it as a **custom skill** in Cowork:

1. In Cowork, open **Skills** → **Add custom skill**.
2. Name it `aggregate-spreadsheets`.
3. Paste the entire contents of `tools\spreadsheet-aggregator\SKILL.md` (the `---` frontmatter
   **and** the body) as the skill definition, then **Save**.

*(If your Cowork build asks for a file instead of pasted text, point it at
`tools\spreadsheet-aggregator\SKILL.md`.)*

### Step 3 — Run it (one-off)

In Cowork, paste this prompt — **replace the folder path with your OneDrive folder**:

```
Using the aggregate-spreadsheets skill, append every .xlsx in my OneDrive folder
"C:\Users\<you>\OneDrive - Microsoft\SurfaceCatalogs" into a summary<dateRun>.xlsx in that
same folder. Skip any existing summary*.xlsx, drop blank/total rows, add a "Source File"
column, and don't modify the source files. Then tell me the row count, column count, and
which files were combined.
```

**Expected result:** `summary<today>.xlsx` appears **in the OneDrive folder**, with one row
per SKU across all source files and a `Source File` column. ✅

### Step 4 (optional) — Schedule it for every Friday

To make it recur, paste this in Cowork instead:

```
Set up a recurring Cowork task every Friday at 5pm that uses the aggregate-spreadsheets
skill to append every .xlsx in my OneDrive folder
"C:\Users\<you>\OneDrive - Microsoft\SurfaceCatalogs" into a dated summary<dateRun>.xlsx in
that folder. Skip prior summary*.xlsx files, drop blank/total rows, add a "Source File"
column, and post a short summary (row count + sources) after each run.
```

Each Friday run writes a new dated file, giving you a weekly history of summaries.

---

## Part 3 — Cowork Prototype: read-only app over the summary

Once a `summary<dateRun>.xlsx` exists, build a local viewer. Paste in Cowork:

```
Build a read-only local app (runs on this machine, never writes to the data) that loads the
most recent summary*.xlsx in "<folder from Part 1 or 2>" and lets me browse and filter rows:
filter by Retailer, Product Line, and Memory (GB); sort by Price (USD); and show totals and
averages computed in the app (not written back to the file). Refresh from the newest summary
file on launch.
```

---

## Troubleshooting

- **"need at least 2 source spreadsheets"** — the folder has fewer than 2 `.xlsx` files (or
  only a `summary*.xlsx`, which is skipped). Add the catalogs.
- **Column mismatch warning** — a file's columns differ from the first file. Fix the file, or
  add `--strict` to the script to hard-stop instead of warn.
- **Totals showing up** — keep totals out of the source files; the tool drops common total
  rows automatically and computes totals only in the app (Part 3).
- **Script missing packages** — run `pip install pandas openpyxl`.

---

## Reference

### Files in this repo

```
spreadsheets\
    BestBuy.xlsx, Walmart.xlsx, Costco.xlsx      # synthetic source catalogs (13 SKUs each)
tools\spreadsheet-aggregator\
    aggregate_spreadsheets.py                    # the append/clean/format engine
    SKILL.md                                     # Cowork/CLI custom-skill definition
    PROMPTS.md                                   # copy-paste prompts for every flow
README.md                                        # this guide
```

### Script options

| Flag | Default | Description |
|------|---------|-------------|
| `--input` | *(required)* | Folder with the source `.xlsx` files (a OneDrive folder is a local path). |
| `--output` | same as `--input` | Where to write the summary file. |
| `--date-format` | `%Y-%m-%d` | strftime format for `<dateRun>`. Use `%Y-%m-%d_%H%M` for multiple runs/day. |
| `--sheet-name` | `Summary` | Worksheet name in the output file. |
| `--strict` | off | Fail if any file's columns differ from the first file. |

### Catalog columns (21)

`Retailer`, `SKU`, `Product Line`, `Model Name`, `Form Factor`, `Processor`,
`Memory (GB)`, `Memory Type`, `Storage`, `GPU`, `AI Performance`, `Interconnect`,
`Screen Size (in)`, `Weight (lb)`, `Color`, `Operating System`, `Price (USD)`,
`Units In Stock`, `Availability Date`, `Customer Rating`, `Warranty`

### Reference: product specs

Source pages:

- **Surface Laptop Ultra:** https://www.microsoft.com/en-us/surface/devices/surface-laptop-ultra
- **Surface RTX Spark Dev Box:** https://www.microsoft.com/en-us/surface/devices/surface-rtx-spark-dev-box?icid=SSM_Search_SurfaceRTXSparkDevBox_CTA1

**Surface Laptop Ultra** — on-device AI laptop with up to 128 GB unified memory.

| Attribute | Spec |
|-----------|------|
| Form factor | Laptop (14.5") |
| Processor | Snapdragon X Elite (12-core) |
| Memory | 32 / 64 / 128 GB unified LPDDR5X |
| GPU / NPU | Integrated Qualcomm Adreno + NPU (45 TOPS) |
| Storage | 1 TB – 2 TB SSD |
| Operating system | Windows 11 Pro |
| Colors | Platinum, Graphite, Sapphire |

**Surface RTX Spark Dev Box (NVIDIA GB10 platform)** — 20-core ARM Grace CPU + Blackwell RTX
GPU over NVLink-C2C, up to 128 GB unified LPDDR5X.

| Attribute | Spec |
|-----------|------|
| Processor (CPU) | 20-core NVIDIA Grace ARM CPU (co-developed with MediaTek, TSMC 3nm process) |
| Graphics (GPU) | Blackwell RTX, up to 6,144 CUDA cores and 5th-gen Tensor Cores |
| AI performance | Up to 1 PetaFLOP of FP4 processing (with sparsity) |
| System memory | Up to 128 GB unified LPDDR5X (lower-tier configs start capped at 64 GB) |
| Interconnect | NVIDIA NVLink-C2C chip-to-chip link |
| Power envelope | 18 W – 80 W depending on manufacturer design |
| Form factor | Slim 14" – 16" laptop chassis, ~14 mm thick, ~3 lb |
| Operating system | Windows 11 optimized for ARM unified memory |

> Catalog values (SKUs, pricing, stock, ratings, availability) are synthetic demo data; the
> specs above are modeled on the official pages and the published GB10 base platform spec.
