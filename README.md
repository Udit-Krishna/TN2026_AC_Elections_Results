# Tamil Nadu AC May 2026 — scraped CSV layout

This repository distributes CSV extracts of published Tamil Nadu Assembly Constituency results (“TN”, state code **S22**) under the folders below. Paths are relative to the project root.

---

## Disclaimer and affiliation

This project is **not affiliated with or endorsed by** the Election Commission of India (ECI). Published electoral totals remain official products of ECI (see **`NOTICE`**). Anything extracted here is provided **as is**, without warranty of correctness or completeness for any purpose—including journalism, litigation, or campaign messaging—and should be checked against [results.eci.gov.in](https://results.eci.gov.in/) before reliance.

Scraping can stop working whenever URLs or HTML change; snapshot CSVs may drift unless regenerated.

---

## License

Documentation in this repository (for example `README.md`) is released under the [MIT License](LICENSE).

Bundled or regenerated CSV extracts derive from official publications as noted in **`NOTICE`**; cite **both** this repository (when publishing derivatives you built here) **and** the ECI portal as the originating governmental source.

---

## Citation

Suggested wording:

> Derived counts assembled using **TN-2026-ECI-Scraping** (paste your repository URL here), extracting tables published by the Election Commission of India at https://results.eci.gov.in/.

Replace with your GitHub URL / Zenodo DOI if you mint tags or archival deposits.

---

## Methodology and known limitations

- Figures were extracted from ECI-published HTML; some non-Latin or malformed header/name rows were dropped during consolidation (English-focused sanity checks).
- **Statewide CSV**: trailing **`col_*`** columns mirror stray glued markup fragments (“party wise state trends…”). Prefer documented snake_case columns for joins (see below).
- **Party naming**: abbreviations differ across vote-share exports vs wins CSV vs candidate exports—normalize explicitly before merges (`party_abbr`, full strings).
- **Constituency label**: wins CSV uses `NAME(const_no)`; statewise uses bare names plus **`const_no`**—always align joins on **`const_no`** (1–234).

---

## Folder overview

| Folder | Role |
|--------|------|
| [`TN_AC2026_general_statistics/`](TN_AC2026_general_statistics/) | State-level summaries: constituency grid, vote share, seat totals, merged wins, AC number ↔ name |
| [`TN_AC2026_constituency_wins_by_party/`](TN_AC2026_constituency_wins_by_party/) | One CSV per party that won at least one seat (same row shape as the merged wins file) |
| [`TN_AC2026_candidates_by_ac/`](TN_AC2026_candidates_by_ac/) | One CSV per assembly constituency (`001` … `234`): full candidate list and votes |

---

## `TN_AC2026_general_statistics/`

### `TN_AC2026_statewise_by_constituency.csv`

Row grain: **one constituency**.

| Column (main) | Meaning |
|-----------------|--------|
| `constituency` | Constituency name (English; normalized in scrape) |
| `const_no` | Assembly constituency number **1–234** |
| `leading_candidate`, `leading_party`, `trailing_candidate`, `trailing_party` | Leading / runner-up from the statewise grid |
| `margin`, `round`, `status` | Margin, counting round fraction (e.g. `31/31`), declaration status |

Additional columns (`col_9`, `col_10`, …) come from extra cells in the published HTML table (party-wise trend snippets, etc.). Prefer the snake_case columns above for analysis; treat trailing `col_*` as optional / noisy.

### `TN_AC2026_constituency_no_to_name.csv`

Row grain: **one AC number**.

| Column | Meaning |
|--------|--------|
| `const_no` | **1–234** |
| `constituency_name` | Name aligned with the statewise scrape (deduped by `const_no`, sorted) |

This file is derived from the statewise CSV whenever that scrape succeeds.

### `TN_AC2026_vote_share_by_party.csv`

Row grain: **one party** (state-wide vote share).

| Column | Meaning |
|--------|--------|
| `Party` | Short party label as on the site (often abbreviation-style) |
| `Vote %` | Percentage string (e.g. `21.21%`) |
| `Total Votes` | Integer vote total |

### `TN_AC2026_party_seat_totals.csv`

Row grain: **one party**.

| Column | Meaning |
|--------|--------|
| `party_abbr` | Abbreviation used on detail pages (e.g. `TVK`, `DMK`) |
| `party_name` | Long name including abbreviation suffix |
| `seats_won` | Seats won |

### `TN_AC2026_all_party_constituency_wins.csv`

Row grain: **one seat won** (concatenation of all per-party win files).

| Column | Meaning |
|--------|--------|
| `party_name`, `party_abbr` | Party |
| `S.No` | Serial within that party’s list |
| `Constituency` | **String like `TIRUTTANI(3)`** — name + **`(const_no)`** |
| `Winning Candidate`, `Total Votes`, `Margin`, `Status` | Win stats |

**Extracting `const_no` for joins:** parse the trailing parenthetical, e.g. `TIRUTTANI(3)` → `3`.

---

## `TN_AC2026_constituency_wins_by_party/`

Files: `TN_AC2026_constituency_wins_<ABBR>.csv` (e.g. `TN_AC2026_constituency_wins_ADMK.csv`).

Same columns as **`TN_AC2026_all_party_constituency_wins.csv`**, but only rows for that party.

---

## `TN_AC2026_candidates_by_ac/`

Files: `TN_AC2026_ac_<NNN>_candidates.csv` where `<NNN>` is zero-padded **`const_no`** (`001` … `234`).

Row grain: **one candidate** in that constituency.

| Column | Meaning |
|--------|--------|
| `const_no` | Same numeric AC id (**matches statewise `const_no`**) |
| `page_title`, `source_url` | Provenance |
| `sn` | Ballot order / serial on the page |
| `candidate`, `party` | Candidate and party |
| `evm_votes`, `postal_votes`, `total_votes` | Votes |
| `%_of_votes` | Share in that constituency |

---

## How to join datasets

### Canonical constituency key: `const_no`

Use **`const_no`** (integer **1–234**) whenever possible:

- **Statewise** ↔ **AC name map**: `statewise["const_no"]` = `map_df["const_no"]`.
- **Statewise** ↔ **per-AC candidates**: `statewise["const_no"]` = `candidates["const_no"]`.
- **Merged / per-party wins** ↔ others: add a column by parsing `Constituency`:

```python
import pandas as pd

wins = pd.read_csv("TN_AC2026_general_statistics/TN_AC2026_all_party_constituency_wins.csv")
wins["const_no"] = wins["Constituency"].str.extract(r"\((\d+)\)$").astype(int)

names = pd.read_csv("TN_AC2026_general_statistics/TN_AC2026_constituency_no_to_name.csv")
wins_named = wins.merge(names, on="const_no", how="left")
```

### Constituency *names* differ by file

- Statewise / map: plain name (e.g. `TIRUTTANI`).
- Wins CSVs: `NAME(AC_NO)` in `Constituency`.

Do not rely on string equality between those two; join on **`const_no`** (or parse wins as above).

### Party identifiers differ by table

- **Seat totals**: `party_abbr`, full `party_name`.
- **Vote share**: short `Party` column — may not exactly equal `party_abbr`; fuzzy matching or a manual alias map may be needed for strict merges.
- **Candidates**: full party name text per candidate row.

Typical pattern: merge seat totals to wins on `party_abbr`, or on normalized `party_name` after stripping suffix noise.

### Load all candidate files into one table

```python
from pathlib import Path

paths = sorted(Path("TN_AC2026_candidates_by_ac").glob("TN_AC2026_ac_*_candidates.csv"))
cand_all = pd.concat([pd.read_csv(p) for p in paths], ignore_index=True)

state = pd.read_csv("TN_AC2026_general_statistics/TN_AC2026_statewise_by_constituency.csv")
combined = cand_all.merge(state, on="const_no", how="left", suffixes=("", "_statewise"))
```
