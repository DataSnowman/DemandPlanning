# Surface Catalog Spreadsheets — Demand Planning

Synthetic retailer catalog data for the soon-to-launch **Surface Laptop Ultra** and
**Surface RTX Spark Dev Box**, used for the Demand Planning ingesting & aggregating demo.

## Files

| File | Retailer | Rows |
|------|----------|------|
| `spreadsheets\BestBuy.xlsx` | Best Buy | 13 SKUs |
| `spreadsheets\Walmart.xlsx` | Walmart | 13 SKUs |
| `spreadsheets\Costco.xlsx` | Costco | 13 SKUs |

The catalogs live in the `spreadsheets\` folder. All three share identical columns (21) in
identical order, so they append cleanly into a single `summary<dateRun>.xlsx` (see
`tools\spreadsheet-aggregator\`). No total/subtotal rows are stored in the files.

> **Note:** The values in these spreadsheets are **synthetic** (SKUs, pricing, stock,
> ratings, availability dates) generated for demo purposes. The hardware attributes below
> are modeled on the official product pages and the published GB10 base platform spec.

## Product references

- **Surface Laptop Ultra:** https://www.microsoft.com/en-us/surface/devices/surface-laptop-ultra
- **Surface RTX Spark Dev Box:** https://www.microsoft.com/en-us/surface/devices/surface-rtx-spark-dev-box?icid=SSM_Search_SurfaceRTXSparkDevBox_CTA1

## Base specifications

### Surface Laptop Ultra

On-device AI laptop with up to **128 GB unified memory** for running larger models and
datasets locally (greater privacy, lower latency), scaling to the cloud for frontier-scale
models when needed.

| Attribute | Spec |
|-----------|------|
| Form factor | Laptop (14.5") |
| Processor | Snapdragon X Elite (12-core) |
| Memory | 32 / 64 / 128 GB unified LPDDR5X |
| GPU / NPU | Integrated Qualcomm Adreno + NPU (45 TOPS) |
| Storage | 1 TB – 2 TB SSD |
| Operating system | Windows 11 Pro |
| Colors | Platinum, Graphite, Sapphire |

### Surface RTX Spark Dev Box (NVIDIA GB10 platform)

Developer box with VS Code, WSL, and PowerShell 7 pre-installed, AI-powered Intelligent
Terminal, and Coreutils for Windows — built on the NVIDIA RTX Spark GB10 hardware platform.

**NVIDIA RTX Spark GB10 base platform:** a 20-core ARM-based Grace CPU paired with a
Blackwell RTX GPU via NVLink-C2C, supporting up to 128 GB of unified LPDDR5X memory.

| Attribute | Spec |
|-----------|------|
| Processor (CPU) | 20-core NVIDIA Grace ARM CPU (co-developed with MediaTek, TSMC 3nm process) |
| Graphics (GPU) | Blackwell RTX architecture, up to 6,144 CUDA cores and 5th-gen Tensor Cores |
| AI performance | Up to 1 PetaFLOP of FP4 processing (with sparsity) |
| System memory | Up to 128 GB unified LPDDR5X (lower-tier configurations start capped at 64 GB) |
| Interconnect | NVIDIA NVLink-C2C chip-to-chip link connecting CPU and GPU |
| Power envelope | Scales from 18 W up to 80 W depending on manufacturer design |
| Form factor | Slim 14" – 16" laptop chassis, starting ~14 mm thick and ~3 lb |
| Operating system | Windows 11 optimized for ARM unified memory and heterogeneous workload scheduling |

## Columns

`Retailer`, `SKU`, `Product Line`, `Model Name`, `Form Factor`, `Processor`,
`Memory (GB)`, `Memory Type`, `Storage`, `GPU`, `AI Performance`, `Interconnect`,
`Screen Size (in)`, `Weight (lb)`, `Color`, `Operating System`, `Price (USD)`,
`Units In Stock`, `Availability Date`, `Customer Rating`, `Warranty`

## Aggregating these files

The **Cowork** step of the demo appends every `.xlsx` in a folder (a synced OneDrive folder
or a local path) into a single dated summary workbook.

```bash
python tools/spreadsheet-aggregator/aggregate_spreadsheets.py --input spreadsheets
```

Produces `summary<dateRun>.xlsx` with a `Source File` column (origin preserved), a formatted
frozen header, and an auto-filter. Source files are never modified; blank and total/subtotal
rows are dropped automatically.

### Options

| Flag | Default | Description |
|------|---------|-------------|
| `--input` | *(required)* | Folder containing the source `.xlsx` files. On Windows a synced OneDrive folder is just a local path — point `--input` at it. |
| `--output` | same as `--input` | Where to write the summary file. |
| `--date-format` | `%Y-%m-%d` | strftime format for the `<dateRun>` suffix. Use `%Y-%m-%d_%H%M` for multiple runs/day. |
| `--sheet-name` | `Summary` | Worksheet name in the output file. |
| `--strict` | off | Fail (instead of warn) if any file's columns differ from the first file. |

Requires `pip install pandas openpyxl`.

### Scheduling (e.g., every Friday)

- **Copilot CLI / Cowork**: schedule a recurring prompt (Friday cron `0 17 * * 5`) using the
  prompt in `tools\spreadsheet-aggregator\PROMPTS.md`.
- **Windows Task Scheduler**: create a weekly Friday task running the `python ... --input ...`
  command above. Each run yields a dated summary, building a weekly history.

### Cowork skill

The repo ships a Cowork/CLI skill, `aggregate-spreadsheets`, defined in
`tools\spreadsheet-aggregator\SKILL.md`. To add it as a **custom skill in the Cowork UI**:

1. Open Cowork → **Skills** (or **Settings → Skills**) → **Add custom skill**.
2. Name it `aggregate-spreadsheets` and paste the contents of
   `tools\spreadsheet-aggregator\SKILL.md` (the YAML frontmatter + body) as the skill definition.
3. Save. Invoke it with `/aggregate-spreadsheets` or a request like
   *"aggregate the spreadsheets in my OneDrive folder into a summary file."*

Prompts for every demo flow (Chat, Cowork one-off, Cowork scheduled, prototype app) are in
`tools\spreadsheet-aggregator\PROMPTS.md`.
