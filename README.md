# PFAS & Lobster Fishery Perception Survey – Data Wrangling & Ranking Analysis

## 1. Project Overview

This repository contains the full data wrangling and analysis pipeline for a survey of lobster fishery stakeholders’ perceptions of PFAS contamination and other fisheries-related issues.

Documentation and code are intended to be **fully reproducible** so that someone who has never worked with these data before can:

1. Obtain the raw data (or understand its structure and origin),
2. Recreate the cleaned dataset, and
3. Reproduce all analyses and visualizations, including the ranking analysis of perceived concerns.

This repo supports open and transparent science by:

- Clearly documenting data provenance and cleaning steps,
- Providing fully runnable code (R Markdown and R scripts),
- Explicitly describing how missing data and partial rankings are handled, and
- Making all modeling decisions and visualizations reproducible.

---

## 2. Data Source & Provenance

### 2.1 Original Data Collection

- **Instrument:** Google Forms survey
- **Population:** Lobster fishery stakeholders (e.g., harvesters) in Maine
- **Mode:** Online self-administered survey
- **Output format:** Google Forms → exported as CSV

The raw Google Forms export includes:

- Timestamp / response metadata
- Demographics
- Fishing zone information
- Knowledge/awareness questions (e.g., PFAS awareness)
- Attitudinal questions
- **Issue ranking questions** (the 6–7 issues that are analyzed in `ranking1.Rmd`)

### 2.2 Original Raw File

- **File type:** CSV exported from Google Forms
- **Original filename:** `INSERT_RAW_FILENAME_HERE.csv`
- **Storage location:** Documented in this repo under `data/raw/` (or described here if not shared directly for privacy reasons).

---

## 3. Manual / Excel-Based Cleaning (Preliminary Wrangling)

> **You will fill this section in based on what you did in Excel.**

We used Excel for preliminary cleaning of the Google Forms CSV before moving into R.

**Steps performed in Excel (to be completed):**

- [ ] **File import and basic checks**
  - Opened the Google Forms CSV in Excel.
  - Verified that all columns and rows were imported correctly.
- [ ] **Column renaming / re-ordering**
  - Renamed long auto-generated question labels to more concise variable names where needed.
- [ ] **Basic data cleaning**
  - Removed test responses / incomplete pilot responses (if any).
  - Fixed obvious text inconsistencies (e.g., `"Zone F, Zone G"` to standardized format).
- [ ] **Recoding / derived variables**
  - Created binary or categorical variables (e.g., Yes/No awareness, age groupings, etc.).
  - Marked or coded missing values consistently.
- [ ] **Saving cleaned data**
  - Saved the cleaned Excel file as  
    `JED_cleaned_LobstermenResponses.xlsx`  
    placed in a local project folder (and documented here).

> **Note:** Be explicit here about:
> - Which columns were dropped or kept,
> - Any manual recodes,
> - How missing or “Prefer not to answer” responses were treated.

---
