# Netflix Content Explorer

A menu-driven Python data-analysis application built on the **Netflix Titles
Dataset**. It loads the catalogue, cleans it, and answers ten business questions
about the content — what dominates the library, who produces it, which genres
and ratings are most common, how the catalogue grew over time, and how the
release years are distributed — then charts the results with Matplotlib.

Everything runs inside a single, well-commented Jupyter Notebook. There is no
web framework, no database, no API and no authentication.

---

## Overview

| | |
| --- | --- |
| **Project type** | Minor project — Artificial Intelligence / Data Science |
| **Deliverable** | `notebooks/netflix_analysis.ipynb` |
| **Dataset** | Netflix Titles Dataset (Kaggle) |
| **Libraries** | Pandas, NumPy, Matplotlib, Jupyter Notebook |
| **Interface** | Console menu with 10 analysis options + quit |

The notebook follows the standard data-analysis workflow:

1. **Import** the libraries.
2. **Load** `data/netflix_titles.csv` into a Pandas `DataFrame`.
3. **Explore** the raw data — shape, dtypes, head/tail, `info()`, missing values.
4. **Clean** it — fill missing text fields, parse dates, keep a date-valid view.
5. **Engineer features** — derive `year_added`, split multi-value cells.
6. **Analyse** it — ten functions answering ten business questions.
7. **Visualise** it — six charts that summarise the findings.
8. **Test** it — a `unittest` suite that runs when the notebook is executed.
9. **Present** the results through a menu, like a small tool a content
   strategist could actually use.

---

## Dataset

| Item | Value |
| --- | --- |
| Dataset name | Netflix Shows (Netflix Titles Dataset) |
| Source | Kaggle — [shivamb/netflixshows](https://www.kaggle.com/datasets/shivamb/netflixshows) |
| File name | `netflix_titles.csv` |
| Location | `data/netflix_titles.csv` |
| Size | 8,807 rows × 12 columns |
| Row meaning | one Netflix title (a Movie or a TV Show) |

### Columns

| # | Column | Description |
| --- | --- | --- |
| 1 | `show_id` | Unique identifier for the title |
| 2 | `type` | `Movie` or `TV Show` |
| 3 | `title` | Title name |
| 4 | `director` | Director name |
| 5 | `cast` | Main cast members |
| 6 | `country` | Country of production — may hold several comma-separated countries |
| 7 | `date_added` | Date the title was added to Netflix |
| 8 | `release_year` | Original release year of the work |
| 9 | `rating` | Content rating (`TV-MA`, `PG-13`, …) |
| 10 | `duration` | Length, e.g. `90 min` or `3 Seasons` |
| 11 | `listed_in` | Genres — holds several comma-separated genres |
| 12 | `description` | Short synopsis |

> **The dataset is in the repository** at `data/netflix_titles.csv` (8,807 rows).
> `data/README.txt` repeats the placement instructions for anyone who needs to
> restore it.

---

## Features

The menu printed by `main()`:

```
==================================================
NETFLIX CONTENT EXPLORER
==================================================
1. Count of Movies vs TV Shows
2. Number of Titles Added Each Year
3. Top 10 Producing Countries
4. Top 5 Most Common Genres
5. Rating Distribution
6. Oldest and Newest Release Years
7. Search Titles by Keyword
8. Content Added in a Specific Year
9. Total Unique Countries
10. Summary Statistics on Release Year
0. Quit
```

| Option | Function | What it does |
| --- | --- | --- |
| 1 | `count_movies_tvshows()` | Counts Movies vs TV Shows with `value_counts()` and shows each share as a percentage. |
| 2 | `titles_added_each_year()` | Counts the titles added in each calendar year using `year_added`, in chronological order. |
| 3 | `top_countries(n=10)` | Ranks the top 10 producing countries **after splitting** multi-country cells. |
| 4 | `top_genres(n=5)` | Ranks the top 5 genres **after splitting** the comma-separated `listed_in` values. |
| 5 | `rating_distribution()` | Shows every rating, its title count and percentage, sorted from most to least common. |
| 6 | `release_year_range()` | Reports the oldest and newest release year, the range, and the titles behind each. |
| 7 | `search_titles()` | Case-insensitive keyword search over `title`; asks the user for the keyword. |
| 8 | `content_added_in_year()` | Lists the content added in a year chosen by the user, with validated numeric input. |
| 9 | `unique_countries()` | Counts distinct countries after splitting, reporting both with and without `Unknown`. |
| 10 | `release_year_statistics()` | NumPy summary statistics: count, mean, median, min, max, standard deviation, Q1/Q3, IQR. |
| 0 | — | Quits the program. |

### Why multi-value cells are split

`country` and `listed_in` hold **lists inside a single cell**:

```
country   : "United States, Canada"
listed_in : "Dramas, International Movies"
```

A naive `df["country"].value_counts()` would report one row called
*"United States, Canada"* and hide both real countries, and a title in five
genres would count as one genre instead of five. The notebook therefore
**explodes** those cells into one row per value (`explode_multi_value()`) and
counts there. After splitting, one title counts once for every country it was
produced in and once for every genre it belongs to — which is what a business
user expects, and why the country and genre totals exceed the number of titles.

---

## Data Cleaning

Cleaning is applied to a **copy** of the data (`df = df_raw.copy()`); the CSV
file and `df_raw` are never modified.

| # | Operation | Reason |
| --- | --- | --- |
| A | Missing `country` → `"Unknown"` | Keeps the row usable for country analysis without inventing a country. |
| B | Missing `director` → `"Not Listed"` | Makes "no director listed" explicit and greppable. |
| C | Missing `cast` → `"Not Listed"` | Same idea for the cast column. |
| D | Missing `rating` → `"Not Rated"` | The distribution then shows the true count, including unrated titles. |
| E | Parse `date_added`; build a separate date-valid view | Turns the text into real datetimes and isolates the 10 rows without a date. |
| F | Create `year_added` from `date_added` | Turns a date into a number that can be counted per year. |

**Why fill for A–D but keep every row for E?** A–D are descriptive text
columns; deleting thousands of otherwise valid titles would distort every count
in the project. `date_added` is different: the year of addition drives options 2
and 8, so the notebook keeps **two views** of the same cleaned data:

* `df` — the **general dataset**, all **8,807** rows. Used by options 1, 3, 4,
  5, 6, 7, 9 and 10.
* `df_dated` — the **date-valid dataset**, **8,797** rows (the 10 titles with no
  `date_added` removed). Used only by options 2 and 8.

This way the 10 titles without a date are still counted by every question that
does not need one, instead of being dropped from the whole project.

Two details worth noting:

* `pd.to_datetime(..., errors="coerce", format="mixed")` turns unreadable dates
  into `NaT` instead of raising, and `format="mixed"` infers the layout of each
  value separately. Kaggle mirrors of this dataset store `date_added` either as
  ISO text (`2017-09-01`) or as English text (`September 1, 2017`); without
  `"mixed"` Pandas would assume one layout and silently coerce the other to
  `NaT`, deleting valid rows.
* A `PASS` / `FAIL` validation table in section 7.4 proves every cleaning rule
  held, so nothing is hidden or swallowed.

Feature engineering then adds `release_year` as a nullable integer, plus
`release_decade`, `duration_minutes` and the exploded `country_rows` /
`genre_rows` tables. On the real data these hold **10,843** country rows
(**123** distinct countries) and **19,323** genre rows (**42** distinct genres).

---

## Visualizations

Section 12 turns the key results into six Matplotlib charts, each built from the
already-cleaned data (nothing is hard-coded) and each guarded against an empty
dataset with a friendly "No data available" message:

| Chart | Figure | Insight shown |
| --- | --- | --- |
| 1 | Movies vs TV Shows | The movie-led shape of the catalogue (6,131 vs 2,676). |
| 2 | Number of Titles Added Each Year | The steep climb to a 2019 peak, then the slowdown. |
| 3 | Top 10 Producing Countries | Horizontal bars keep long country names readable. |
| 4 | Top 5 Most Common Genres | Genre concentration after splitting multi-genre cells. |
| 5 | Rating Distribution | The adult-oriented skew (TV-MA and TV-14 dominate). |
| 6 | Release-Year Distribution | A histogram showing the 2013–2019 concentration. |

The rendered charts are stored as PNG output inside the executed notebook.

---

## Technologies

| Technology | Version | Purpose |
| --- | --- | --- |
| Python | 3.12 (3.10+ works) | Application logic and the menu |
| Pandas | >= 2.0 | Loading, cleaning, grouping and counting |
| NumPy | >= 1.24 | Release-year summary statistics |
| Matplotlib | >= 3.7 | The six charts in section 12 |
| Jupyter Notebook | >= 1.0 | The executable deliverable |

No other third-party package is used — no web framework, database, API or
terminal-UI library. `pandas>=2.0` is required because `date_added` is parsed
with `pd.to_datetime(..., format="mixed")`, which only exists from Pandas 2.0.
The automated tests use the standard-library `unittest` module, so they add no
dependency.

---

## Project Structure

```
Netflix_Content_Explorer/
│
├── data/
│   ├── netflix_titles.csv          <- the dataset (8,807 rows x 12 columns)
│   └── README.txt                  <- where the dataset comes from
│
├── notebooks/
│   └── netflix_analysis.ipynb      <- PRIMARY DELIVERABLE (executed, 0 errors)
│
├── output/
│   └── sample_output.txt           <- real console output of a full run
│
├── README.md
├── requirements.txt
└── .gitignore
```

### Notebook sections

| # | Section |
| --- | --- |
| 1 | Project Title |
| 2 | Project Objective |
| 3 | Dataset Information |
| 4 | Import Libraries |
| 5 | Load Dataset |
| 6 | Initial Dataset Exploration |
| 7 | Data Cleaning |
| 8 | Feature Engineering |
| 9 | Analysis Functions |
| 10 | Menu-Driven Program |
| 11 | Sample Run |
| 12 | Visualizations |
| 13 | Three Business Insights |
| 14 | Automated Tests |
| 15 | Conclusion |

---

## Installation

Requires **Python 3.12** (3.10 or newer also works). Run the following in
PowerShell or Command Prompt from the project root.

```bat
:: 1. Create the virtual environment
python -m venv .venv

:: 2. Activate it (Windows)
.venv\Scripts\activate

:: 2b. On macOS / Linux instead:
:: source .venv/bin/activate

:: 3. Install the dependencies
pip install -r requirements.txt

:: 4. Place netflix_titles.csv inside data\
::    (download from https://www.kaggle.com/datasets/shivamb/netflixshows)

:: 5. Launch Jupyter Notebook
jupyter notebook

:: 6. In the browser that opens, select notebooks\netflix_analysis.ipynb
:: 7. Run every cell: Kernel > Restart and Run All
```

Deactivating the environment when you are done:

```bat
deactivate
```

---

## Running

```bat
jupyter notebook
```

Then open `notebooks/netflix_analysis.ipynb` and choose
**Kernel → Restart and Run All**.

The notebook locates the CSV on its own, so it works whether Jupyter was
started from the project root or from inside `notebooks/`. If the file is
missing, the load cell stops with an explicit `FileNotFoundError` explaining
where to put it instead of a bare traceback.

### Using the interactive menu

Section 10 defines `main()` but leaves the call commented out, so an automated
notebook run never blocks waiting for keyboard input. To use the menu by hand,
uncomment this line in the last cell of section 10.1 and run the cell:

```python
main()
```

### Regenerating the sample output

Section 11 of the notebook drives `main()` with a fixed list of answers
(`["1", …, "7", "dark", "8", "2019", "9", "10", "0"]`) by temporarily patching
`builtins.input`, then writes the captured console text to
`output/sample_output.txt`. So the sample output is produced automatically
whenever the notebook is run — no copy-pasting required.

The same run can be done without opening Jupyter:

```bat
jupyter nbconvert --to notebook --execute --inplace notebooks\netflix_analysis.ipynb
```

---

## Sample Output

`output/sample_output.txt` holds a **real** session captured from the actual
dataset, exercising options 1–10 and the quit option, with realistic input for
the two interactive options (option 7 → `dark`, which matches 39 real titles;
option 8 → `2019`, the peak year with 2,016 titles).

It was produced automatically by section 11.1 of the notebook and carries a
header recording the dataset path, the two row counts and the exact keystrokes
used, followed by the untouched console transcript.

```
NETFLIX CONTENT EXPLORER - SAMPLE RUN
=======================================
Dataset used    : data/netflix_titles.csv
General rows    : 8,807  (used by options 1, 3, 4, 5, 6, 7, 9, 10)
Date-valid rows : 8,797  (used by options 2 and 8)
Rows dropped for a missing date_added: 10
Keystrokes      : 1, 2, 3, 4, 5, 6, 7, dark, 8, 2019, 9, 10, 0
Covers          : menu options 1-10 and 0 (Quit)
```

Key figures in the transcript: **6,131 Movies / 2,676 TV Shows**; additions
rising from 24 (2014) to a peak of **2,016 (2019)**; **123** unique countries;
NumPy release-year statistics of mean **2014.18**, median **2017**, min **1925**,
max **2021**, std **8.82**. The file is 221 lines long.

Section 11.2 of the notebook additionally demonstrates the error handling with
an invalid menu choice (`99`), a non-numeric year (`abc`), an out-of-range year
(`1800`) and an empty keyword — none of which raise an error. That transcript is
printed in the notebook but not written to the file, so the file stays a clean
"one happy path" run.

---

## Error Handling

| Situation | Behaviour |
| --- | --- |
| Dataset missing or in the wrong folder | `FileNotFoundError` with the expected path and step-by-step download instructions |
| Menu choice outside 0–10 | `Invalid choice. Please enter a number from 0 to 10.` and the menu is shown again |
| Non-numeric year (`abc`) | `Invalid year. Please type digits only, for example 2019.` and the year is asked again |
| Out-of-range year (`1800`) | `Invalid year. Please enter a year between 1900 and <current year + 1>.` |
| Empty year input | `No year entered. Returning to the menu.` |
| Empty keyword | `No keyword entered. Nothing to search for.` |
| Keyword with no matches | `No titles found.` |
| Year with no content | `No content found for that year.` plus the range of years that do have content |
| Unreadable date | Coerced to `NaT` with `errors="coerce"`, then reported in the cleaning summary |
| `Ctrl+C` or closed input | `Goodbye!` instead of a traceback |
| One option fails unexpectedly | Reported, and the menu stays open |

---

## Insights

All figures below come from a real run of `notebooks/netflix_analysis.ipynb` on
`data/netflix_titles.csv` (**8,807 rows**) and are reproduced in
`output/sample_output.txt`. The same three paragraphs are in section 13 of the
notebook.

**Insight 1 — A movie-led, recency-weighted library whose headline range is
misleading.** Movies dominate the catalogue at **6,131 titles (69.62%)** against
**2,676 TV Shows (30.38%)**. The NumPy statistics show what that catalogue is
made of: mean release year **2014.18**, median **2017**, and an interquartile
range of only **6 years** (Q1 2013, Q3 2019) — so the library is really a
2013–2019 collection. The advertised span of **1925–2021** is misleading: the
1925 minimum is a *single* title, *Pioneers: First Women Filmmakers\**. That one
row is why the mean sits 2.82 years below the median and why the standard
deviation (**8.82**) is larger than the IQR (6). For planning purposes the
quartiles describe the catalogue far better than the min/max range.

**Insight 2 — The catalogue scaled roughly 84× in five years, then cooled by
about 26%.** Additions went from **24 titles in 2014** to **429 (2016)**, **1,188
(2017)**, **1,649 (2018)** and a peak of **2,016 in 2019** — an 84-fold increase
— before falling to **1,879 (2020)** and **1,498 (2021)**, which is **25.69%**
below the peak. The mix also shifted: TV Shows were 29.4% of 2019 additions but
33.7% of 2021 additions, so the slowdown is concentrated in Movies (1,424 → 993)
rather than in series (592 → 505). Growth has clearly plateaued.

**Insight 3 — The library is international and adult-oriented, and some of its
metadata is not yet trustworthy.** The three largest genres are all broad,
mainstream categories — **International Movies (2,752)**, **Dramas (2,427)** and
**Comedies (1,674)** — and the audience skews adult: **TV-MA (3,207 / 36.41%)**
plus **TV-14 (2,160 / 24.53%)** make up **60.94%** of the catalogue, while
**G and PG together account for only 328 titles (3.72%)**. Geographically the
United States dominates with **3,690 titles**, **3.53×** India's 1,046, yet
**"Unknown" ranks third with 831 titles** because 9.44% of rows have no country
recorded. Three rows (`s5542`, `s5795`, `s5814` — the Louis C.K. specials) carry
a *duration* value ("74 min", "84 min", "66 min") in the `rating` column, which
is why the distribution shows 18 distinct ratings. Rating and country analytics
should therefore be cleaned at the source before they drive any decision.

---

## Automated Tests

Section 14 of the notebook runs a small `unittest` suite (**24 tests**) the
moment the notebook is executed, so a regression is caught without a separate
test command. Most tests use a tiny, made-up dataset — deliberately mixing
single- and multi-value cells and missing values — so the cleaning, splitting
and analysis rules are exercised independently of the real CSV. A handful of
menu tests drive `main()` through `run_scripted_session()` to check the on-screen
messages.

| Test class | Covers |
| --- | --- |
| `TestCleaning` | Placeholder filling, date parsing, `year_added`, missing-file error |
| `TestAnalysisLogic` | All ten option functions on the made-up dataset |
| `TestEdgeCases` | Empty data does not crash any function |
| `TestMenuBehaviour` | Quit, invalid choice, bad year, empty keyword, no-match messages |

A successful run prints `ALL TESTS PASSED (24 run, 0 failures, 0 errors)`.

---

## Testing Checklist

Every item below was executed against the **real** `data/netflix_titles.csv`
using `jupyter nbconvert --to notebook --execute --inplace`, which reported
**0 errors** across all 42 code cells (including the 24 automated tests).

| # | Check | Result |
| --- | --- | --- |
| 1 | Dataset loads successfully | **PASS** — 8,807 rows read from `data/netflix_titles.csv` |
| 2 | Row count verified | **PASS** — 8,807 raw; 8,807 general; 8,797 date-valid |
| 3 | Column count verified | **PASS** — 12 columns, names match the brief exactly |
| 4 | Missing dataset produces a clear message | **PASS** — tested by `TestCleaning.test_missing_dataset_raises` |
| 5 | Data cleaning works | **PASS** — 4,307 missing cells → 23 remaining (all intentional) |
| 6 | `country` missing → `"Unknown"` | **PASS** — 831 filled; `Unknown` totals 831 |
| 7 | `director` missing → `"Not Listed"` | **PASS** — 2,634 filled |
| 8 | `cast` missing → `"Not Listed"` | **PASS** — 825 filled |
| 9 | `rating` missing → `"Not Rated"` | **PASS** — 4 filled |
| 10 | `date_added` missing handled | **PASS** — 10 rows kept in `df`, excluded only from `df_dated` |
| 11 | `year_added` created | **PASS** — 8,797 non-null, range 2008–2021 |
| 12 | `release_year` / `release_decade` / `duration_minutes` | **PASS** — 0 invalid years; `duration` keeps its 3 genuine nulls, which are reported |
| 13 | `country_rows` / `genre_rows` exploded correctly | **PASS** — 10,843 country rows → 123 countries; 19,323 genre rows → 42 genres |
| 14 | Validation table | **PASS** — 10 of 10 checks `PASS` |
| 15 | Option 1 — Movies vs TV Shows | **PASS** — 6,131 / 2,676 |
| 16 | Option 2 — Titles added each year | **PASS** — 14 years, 2008–2021 |
| 17 | Option 3 — Top 10 countries | **PASS** — US 3,690 … Germany 226 |
| 18 | Option 4 — Top 5 genres | **PASS** — International Movies 2,752 … Documentaries 869 |
| 19 | Option 5 — Rating distribution | **PASS** — 18 distinct ratings, TV-MA 3,207 (36.41%) |
| 20 | Option 6 — Oldest / newest release year | **PASS** — 1925 / 2021, titles identified |
| 21 | Option 7 — Keyword search | **PASS** — `dark` → 39 real titles |
| 22 | Option 8 — Content added in a year | **PASS** — 2019 → 2,016 titles |
| 23 | Option 9 — Total unique countries | **PASS** — 123 (122 named + `Unknown`) |
| 24 | Option 10 — NumPy release-year statistics | **PASS** — mean 2014.18, median 2017, std 8.82 |
| 25 | Invalid menu input handled | **PASS** — `99` rejected |
| 26 | Non-numeric year handled | **PASS** — `abc` rejected, re-prompted |
| 27 | Out-of-range year handled | **PASS** — `1800` rejected |
| 28 | Empty year input handled | **PASS** — returns to the menu |
| 29 | Empty keyword handled | **PASS** — "Nothing to search for" |
| 30 | No-match search handled | **PASS** — "No titles found." |
| 31 | Option 0 exits correctly | **PASS** — "Goodbye!" |
| 32 | EOF / Ctrl+C handled | **PASS** — caught in `main()` |
| 33 | Six visualizations render | **PASS** — 6 PNG charts embedded in section 12 |
| 34 | Automated tests | **PASS** — 24 tests, 0 failures, 0 errors |
| 35 | Sample output created | **PASS** — `output/sample_output.txt`, 221 lines of real output |
| 36 | Three insights written | **PASS** — notebook section 13 and this README |
| 37 | Notebook executes with ZERO errors | **PASS** — 42 code cells, 0 errors |
| 38 | Notebook contains actual outputs | **PASS** — executed in place, outputs stored |
| 39 | `README.md` complete | **PASS** |
| 40 | `requirements.txt` verified | **PASS** — `pandas>=2.0,<3.0`, `numpy>=1.24,<3.0`, `matplotlib>=3.7,<4.0`, `jupyter>=1.0.0` |
| 41 | `.gitignore` present | **PASS** — dataset deliberately not ignored |

### Notes on the real data

Two findings that only appear on the genuine dataset, both reported by the
notebook rather than hidden:

* **3 titles have a duration in the `rating` column** (`s5542`, `s5795`,
  `s5814` — the Louis C.K. specials, whose `rating` reads "74 min", "84 min",
  "66 min"). This is why the rating distribution shows 18 distinct values.
* **3 rows have a missing `duration`**, part of the 23 missing cells that survive
  cleaning (alongside the 10 `date_added`/`year_added` pairs). They are listed in
  the remaining-missing report rather than filled, because the brief defines no
  placeholder for that column.
