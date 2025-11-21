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

## R Processing/Categorical and Continuous Visualizations
1. Loading + Pre-processing the Raw CSV
Goals

Read the Google-Forms-derived CSV (lobster_clean2.csv)

Standardize and sanitize non-machine-friendly column names

Fix missing names, duplicate names, and punctuation issues

Recover specific key survey fields reliably, even after name sanitation

Key Steps

read.csv(check.names = FALSE) prevents R from auto-mangling names.

Multiple gsub() passes remove tabs, punctuation, slashes, parentheses, question marks, and collapse repeated underscores.

make.unique() ensures no two columns share the same cleaned name.

Specific columns (e.g., Are you aware…?, Lobster Zone) are located using grep() patterns that tolerate trailing _ or punctuation removed during cleaning.

Blank or missing names are given placeholder names (V1, V2, …) to preserve column count integrity.

Outcome: the dataset is now header-stable, name-consistent, and safe to reference programmatically.

2. Extracting and Harmonizing Binary Yes/No Variables

Multiple survey questions originally had:

inconsistent capitalization ("Yes", "yes", "Y", "1"…),

TRUE/FALSE-like entries,

missing responses ("", NA),

or multi-select answers.

The code implements a universal recoding pattern:

case_when(
  lower %in% c("yes","y","true","1") → "Yes"
  lower %in% c("no","n","false","0") → "No"
  else → NA
)


This yields reproducible binary factors used for crosstabs, McNemar tests, logistic regression, Fisher tests, and Sankey diagrams.

3. Handling Multi-Zone Responses (Lobster Zone)

Respondents often listed multiple zones separated by:

commas,

“and”,

slashes /,

ampersands &.

The pipeline:

Normalizes "and" → ,

Uses separate_rows() to explode multi-zone responses

Trims whitespace

Filters out empty entries

This produces a long-format dataset: one row per person × per zone, enabling zone-by-zone awareness plots.

A zone frequency table is built to annotate x-axis labels with (n=...) counts.

4. Visualization: Awareness by Lobster Zone

A proportionally stacked bar chart is produced:

geom_bar(position = "fill") shows within-zone percentages

Y-axis is % aware/unaware

X-axis labels include zone and sample size

A minimal theme supports publication-ready readability

This is the main “Are you aware of the human-health risks?” zone breakdown.

5. Cross-Tabulation: Awareness vs Reconcat (Point-Source Awareness)

The survey included a free-text field “Reconcat” where respondents listed pollution sources.

The code:

Counts commas → source_count (# of sources)

Treats "No" as zero-sources

Builds aware_bin (Yes/No) = whether any source was listed

Constructs a contingency table vs. the human-health-risk question

Produces a tile heatmap (counts + % row)

Result: a rapid check of internal consistency between two independent awareness questions.

6. Awareness vs Preference for Monitoring Research (McNemar Test)

For the question:

“Would you like research to monitor PFAS levels in lobsters?”

the script:

recodes missing as “No”

builds a paired Yes/No variable set

constructs a 2×2 contingency table

runs mcnemar.test() to detect within-person discordance

visualizes with a tile heatmap

This evaluates whether individuals who are aware are more likely to support monitoring.

7. Ordinal Scoring for Multi-Select or Multi-Intensity Questions

Several questions require ordinal interpretation:

“How concerned are you?” → {Not = 0, Somewhat = 1, Very = 2}

“How involved…” → {Not involved = 0, Somewhat = 1, Very = 2}

“Dependence on fishery” → {Not = 0, Some = 1, Highly = 2}

“Ease of establishing a new territory” → {Not possible = 0, Possible = 1, Very easy = 2}

The pipeline creates:

ordered factors,

numeric scores (subtracting 1),

10-year age-bin boxplots,

jitter overlays,

non-parametric tests: kruskal.test(),

monotonic trends: Spearman’s ρ.

This enables consistent across-question comparisons of how age relates to intensity-based responses.

8. Age-Based Analyses (Core Reproducible Section)

For each major awareness / belief / perception question, the script performs:

A) Age binned (10-year groups)

Boxplot of response by bin

Jittered points for raw visibility

Lines marking per-bin means (for binary outcomes)

Non-parametric tests (Kruskal–Wallis) for any difference across bins

B) Age as continuous

Scatter with jitter

Logistic regression for binary outcomes

Linear regression for ordinal-count outcomes

Annotated lines showing regression formula, R², and p-value

Color-coding points by age bin for visual clarity

This section repeats for:

heard of PFAS,

PFAS as pollutant,

PFAS in Maine,

human-health risk awareness,

concern levels,

co-management involvement,

dependence on the fishery,

trust in state agencies,

ease of establishing new fishing territory,

voicing concerns.

For each, the README makes it clear which trends were significant or not, and how visualization was standardized.

9. Behavioral Response Section (PFAS Detected Scenarios)

For seven “If PFAS were detected…” questions (continue fishing, sell, consume, new area, seek compensation, lobby management, leave fishery):

The script:

Standardizes the Likert responses {not, somewhat, very likely}

Creates ordered factors (or binary recodes)

Builds:

column-wise proportion matrices

stacked bar charts

2D mosaics

binary collapsed analyses

Sankey diagrams showing joint transitions

Why Sankey diagrams?

They show consistency/inconsistency across:

selling vs consuming,

seeking compensation vs lobbying,

fishing in new area vs ease of establishing new territory.

Several high-quality Sankeys with fixed layout, gradient coloring, and custom SVG post-processing are included.

10. Importance Factors: Community Resilience Themes

Survey free-text responses (e.g., “strong community”, “apathy”, “resourcefulness”, etc.) are:

split on commas,

cleaned and normalized,

expanded to long format (one row per tag),

cross-tabulated vs PFAS-handling expectations (y/n/m),

visualized as 100% stacked bars,

colored by conceptual group (positive = green, negative = red, neutral = purple/grey).

This section produces the “resilience factors” figure and table.

11. Handling Conditional Questions (ZBINARY_Y_N + Category Fields)

For the question:

Have you or people you know had to stop lobstering…? If yes, how did you deal with it?

The code:

pulls the binary “yes/no” indicator (ZBINARY_Y_N)

splits multi-tag categories (ZCAT_WHY_YES___…)

builds a tag × Yes/No table

visualizes stacked proportions

provides overall tag distribution

This allows assessing whether certain coping strategies differ across groups.

---
