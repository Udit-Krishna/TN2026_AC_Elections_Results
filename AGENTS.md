# Agent / LLM context

Use this file as a fast orientation when editing or answering questions about this repository.

## What this repo is

- **Published artifact:** CSV extracts of **Tamil Nadu Assembly Constituency (May 2026)** results as shown on the Election Commission of India portal (`results.eci.gov.in`, state code **S22**, 234 ACs).
- **Not an ECI product:** See **`NOTICE`** (provenance, no endorsement) and **`README.md`** (disclaimer, citation, methodology).
- **Documentation license:** **`LICENSE`** (MIT) applies to repo documentation (e.g. `README.md`). Tabulated counts remain official ECI publications; see **`NOTICE`**.

## Layout (authoritative detail in `README.md`)

| Path | Contents |
|------|----------|
| `TN_AC2026_general_statistics/` | Statewise grid, vote share, seat totals, merged all-party wins, `const_no` ↔ name map |
| `TN_AC2026_constituency_wins_by_party/` | Per-party win tables (same columns as merged wins file) |
| `TN_AC2026_candidates_by_ac/` | One file per AC (`TN_AC2026_ac_NNN_candidates.csv`) |

## Joins and analysis

- **Primary key across files:** assembly **`const_no`** in **1…234** (integer). Prefer joins on `const_no`, not raw constituency strings.
- **Wins CSVs:** `Constituency` is often `NAME(N)`; parse trailing `(N)` to get `const_no` when merging with statewise or candidate files.
- **Noise:** `TN_AC2026_statewise_by_constituency.csv` may include trailing `col_*` columns from HTML glue; prefer documented snake_case columns (`README.md`).

## Local-only tooling (do not assume in git)

- **`notebook.ipynb`** may exist on a maintainer machine for **private** regeneration; it is listed in **`.gitignore`** and is **not** part of the public dataset story. Do not add `requirements.txt` or notebook-run instructions unless the maintainer explicitly wants them back.
- There is **no** checked-in scraper package beyond optional local notebook work.

## Conventions for changes

- **README / NOTICE / LICENSE / this file:** Keep wording accurate, short, and aligned with “dataset + docs only” publishing.
- **CSV data:** Treat edits as intentional data updates; avoid drive-by reformatting of large CSVs unless asked.
- **Scope:** Prefer minimal diffs; do not invent new output folders or rename existing CSV stems without explicit instruction (downstream users may depend on paths in `README.md`).

## Quick pointers

- Full column semantics and pandas join examples: **`README.md`**.
- Legal / attribution boilerplate: **`NOTICE`**, **`README.md`** (Disclaimer, License, Citation).
