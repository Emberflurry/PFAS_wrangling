# PFAS & Lobster Fishery Perception Survey – Data Wrangling & Ranking Analysis

## 1. Project Overview

This repository contains the full data wrangling and analysis pipeline for a survey of lobster fishery stakeholders’ perceptions of PFAS contamination and other fisheries-related issues.
1. Obtain the raw data (or understand its structure and origin),
2. Recreate the cleaned dataset, and
3. Reproduce all analyses and visualizations, including the ranking analysis of perceived concerns.

- Data provenance and cleaning steps
- See fully runnable code (R Markdown and R scripts)
- How missing data and partial rankings are handled
- All modeling decisions and visualizations are reproducible
---
## 2. Data Source & Provenance

### 2.1 Original Data Collection

- **Instrument:** Google Forms survey, Google Docs with longform text interview transcriptions
- **Population:** Lobster fishery stakeholders "Lobstermen" in Maine, n=34 at time of project.
- **Mode:** Online self-administered survey with in-person extra notes and conversations recorded as supplementary longform interview notes
- **Output format:** Google Forms, exported as CSV from app.

The raw Google Forms export includes:
(see more detail in 3: Excel-based cleaning)
- Timestamp / response metadata
- Demographics: Gender, age, Town
- Fishing zone information (A-G)
- Knowledge/awareness questions (Pollution, PFAS awareness)
- Attitudinal questions
- **Issue ranking questions** (the 6–7 issues that are analyzed in `ranking1.Rmd`)

### 2.2 Original Raw File

- **File type:** CSV exported from Google Forms
- **Original filename:** `JED_cleaned_LobstermenResponses.csv`
- **Storage location:** Local folder (small file), accessible on desktop by VSCODE.
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
  - Removed test responses / incomplete pilot responses (Grady Welsh, for some questions, including rankings).
  - Several entries were marked as Gender: Female when all interviewees were Male (simple manual conversion to Male)
  - Fixed obvious text inconsistencies (ie Zone 6 is really Zone F, to standardized format).
- [ ] **Recoding / derived variables**
  - Created binary or categorical variables (e.g., Yes/No awareness, age groupings, etc.).
  - Marked or coded missing values consistently.
**Pollution awareness & PFAS knowledge**
- As a lobsterman, are you concerned about pollution? If yes, what types of pollution concern you most?  
- Are you aware of any sources of pollution near your fishing grounds?  
- Have you previously heard of PFAS (“forever chemicals”)?  
- Do you know if PFAS is an issue in Maine?  
- When you think about pollution in fisheries, do you think about PFAS as a pollutant?  
- Do you know if PFAS have been detected in lobsters?  
- Have you heard any news, research, or state communications about PFAS in lobsters?  
- If yes, what source/info?  
- Are you aware of human-health risks associated with PFAS exposure?  
- Do you know anyone impacted by PFAS?  

**Concern questions**
- How concerned are you about PFAS contamination in lobsters?  
- How concerned are you about PFAS negatively affecting lobster health?  
- Do you think PFAS could pose a health risk to consumers?  
- Do you think consumers would eat less lobster if PFAS were present?  
- Do you think PFAS exposure decreases lobster populations?  
- How concerned are you about PFAS research affecting marketability?  

**Trust & engagement**
- How much trust do you have in scientists?  
- What experiences shaped your answer?  
- Would you like research done to monitor PFAS levels in lobsters?  
- Would you want to work with scientists (communication, sampling, meetings)?  
- How involved are you with the co-management system?  
- In what way are you involved (if applicable)?  
- How much trust do you have in state agencies?  
- If PFAS were found in your catch, who would you reach out to for support?  
- Do you feel you could voice concerns regarding pollution to management?  
- Explain your answer  

**Behavioral/hypothetical PFAS responses**
- If PFAS were detected in your fishing area, would you:  
  - continue fishing?  
  - consume the lobster?  
  - sell it?  
  - fish in a new area?  
  - seek compensation/support?  
  - lobby management to clean up?  
  - leave the fishery?  

**Livelihood**
- How easy would it be to establish a new fishing territory?  
- How dependent are you on the fishery?  
- Importance of passing down the fishery  
- How PFAS could impact future generations  

**Social dynamics**
- If some areas were contaminated, how would territoriality/social dynamics change?  

**Ranking task**
- Rank the following from most to least concerning:  
  - shifts in lobster habitat  
  - right whale regulations  
  - market crashes  
  - cost of supplies (bait)  
  - labor costs  
  - PFAS  
  - wind farms  

**Open-ended prompts**
- How the community responds to problems  
- Past experiences stopping lobstering  
- Referral for additional interviews  

---

## 2. Core Cleaning & Standardization Steps

### 2.1 Lobster Zone Normalization
Google-Forms free-text entries (“Zone 6”, “6”, “F”, etc.) were standardized into a consistent zone coding system  
(e.g., **Zone 6 → Zone F**).

---

## 3. Pollution Concern Variable (Q1) Cleaning

The column  
**“As a lobsterman, are you concerned about pollution? If yes, what types…?”**  
contained comma-separated free text with inconsistent casing, spacing, and synonyms.

### 3.1 Automated normalization formula
A canonical list of “base” pollution categories was defined:
No
Mercury
Microplastics
PFAS
Sewage/wastewater treatment discharge
Industrial discharges
Pesticides and herbicides
Fertilizers
Oil/fuel spills
Trash/marine litter
CO2 / ocean acidification
Thermal pollution

Excel formula used to standardize and extract non-typical items:

```excel
=LET(
  norm, LAMBDA(s, LOWER(TRIM(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(s,CHAR(160)," "),", ",",")," ,",","),",  ",",")))),

  txt_norm, norm(K2),
  hay, "," & txt_norm & ",",

  base, {"No";"Mercury";"Microplastics";"PFAS";"Sewage/wastewater treatment discharge";"Industrial discharges (mills, factories, shipyards etc.)";"Pesticides and herbicides";"Fertilizers";"Oil/fuel spills";"Trash/marine litter";"CO2, which causes ocean acidification";"Thermal pollution"},
  base_norm, MAP(base, LAMBDA(b, norm(b))),

  pruned, REDUCE(hay, base_norm, LAMBDA(acc,term, SUBSTITUTE(acc, ","&term&",", ","))),
  cleaned, MID(pruned, 2, LEN(pruned)-2),

  items, FILTER(TEXTSPLIT(cleaned, ","), TEXTSPLIT(cleaned, ",")<>""),

  TEXTJOIN(", ", TRUE, items)
)
```
3.2 Cross-question subtraction (Q1 − Q2)

To remove overlaps with pollution sources listed in the next question:
```excel
=LET(
  k, IFERROR(TEXTSPLIT(K2, ","), ""),
  l, IFERROR(TEXTSPLIT(L2, ","), ""),
  kT, TRIM(k),
  lT, TRIM(l),
  keep, FILTER(kT, ISNA(XMATCH(kT, lT))),

  IF(ROWS(keep)=0, "", TEXTJOIN(", ", TRUE, keep))
)
```
3.3 Merging Fields
```excel
=IF(ISBLANK([@Column1]),[@Column2],CONCAT([@Column2],", ",[@Column1]))
```
Final merged column = Final Q1.


4. Pollution Source (Q2) Cleaning
Base categories:
military bases
fire stations
wastewater treatment plants
general runoff (fertilizers, pesticides, herbicides etc)
Same normalization → prune → extract-nonbase workflow as in Q1.
Recombination:
```excel
=IF(ISBLANK([@Column4]),[@Column5],CONCAT([@Column5],", ",[@Column4]))
```

6. Binary Variables Created
| New variable                      | Original question                                | Excel rule                                                                                                                                            |
| --------------------------------- | ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **PFAS_issue_in_Maine_BINARY**    | Do you know if PFAS is an issue in Maine?        | `=IF([@[Do you know…]]="No","No","Yes")`                                                                                                              |
| **Know_someone_impacted_BINARY**  | Do you know anyone impacted by PFAS?             | `=IF([@[Do you know anyone…]]="No","No","Yes")`                                                                                                       |
| **Willing_to_communicate_BINARY** | Scientist-involvement question                   | `=IF(TEXTBEFORE([@[Would you want to work with scientists…]],",")="Would you want to further communicate with scientists on this topic?","Yes","No")` |
| **Had_to_stop_lobstering_BINARY** | Derived from long-form past-experience responses | Manual                                                                                                                                                |

6. Manually Interpreted Variables
These required contextual reading and could not be automated reliably:

6.1 Support categories

From: “If PFAS were found in your catch, who would you reach out to?”
Coded into:

Private associations
State agencies
Scientists
No one
Other

6.2 Community-response classification

Y/N + qualitative reasoning extracted from:
“Would PFAS be dealt with the same way as other fishery problems?”

6.3 Topic extraction

Two exploratory variables:

Importance factors mentioned/implied

Other factors/categories

Generated through light topic modeling / word grouping.
Not used in final analysis due to vagueness.

6.4 Stopping-lobstering experiences

Had to stop (Y/N)

Reason category (e.g., right whale regulations, warming waters, gear rules, oil spills)

- [ ] **Saving cleaned data**
  - Saved the cleaned Excel file as  
    `JED_cleaned_LobstermenResponses.xlsx`  
    placed in a local project folder (and documented here).



### R Processing/Categorical and Continuous Visualizations
#Survey Cleaning + Analysis Pipeline

Below is a detailed, step-by-step documentation of the full data-cleaning and statistical analysis pipeline used in this project. This section corresponds to the R code contained in the main analysis script and allows full replication of the workflow.
---
## 1. Loading + Pre-processing the Raw CSV

- Import the raw Google-Forms-derived CSV using `read.csv(check.names = FALSE)` to avoid automatic name mangling.
- Sanitize column names by removing:
  - tabs, punctuation, parentheses, slashes, question marks, and repeated underscores.
- Detect and repair:
  - blank column names → assigned placeholders (`V1`, `V2`, …),
  - duplicate names → resolved with `make.unique()`.
- Important survey fields are identified using robust `grep()` patterns, ensuring correct mapping even after header cleaning.
- Output: a stable, machine-friendly dataset ready for extraction/recoding.

---

## 2. Extracting and Harmonizing Binary Yes/No Variables

- Google Forms outputs Yes/No fields inconsistently (`"Yes"`, `"yes"`, `"Y"`, `"1"`, etc.).
- For safety (I don't trust myself sometimes and am lazy to check), All such fields are standardized using:
  - **YES** → `"yes"`, `"y"`, `"true"`, `"1"`
  - **NO** → `"no"`, `"n"`, `"false"`, `"0"`
  - Everything else → `NA`
- Produces clean and consistent binary factors used in:
  - Fisher’s Exact Tests   
  - Logistic regression  
  - Proportion plots  
---

## 3. Handling Multi-Zone Lobster Fishing Areas

- Respondents often reported multiple zones using commas, “and”, slashes, or ampersands.
- The pipeline:
  1. Normalizes `"and"` → `,`
  2. Splits into multiple rows with `separate_rows()`
  3. Trims whitespace and removes empty zones
- Creates a long-format dataset: **one (respondent × zone) row per zone**, enabling zone-level analyses.
- Zone sample sizes are computed and used to annotate graphs.

---

## 4. Awareness by Lobster Zone (Visualization)

- Computes awareness proportions using zone-normalized denominator.
- Produces a % stacked bar chart:
  - `position = "fill"` for within-zone proportions  
  - X-axis labeled as `Zone (n = count)`  
- Allows rapid comparison of PFAS human-health-risk awareness across lobster zones.

---

## 5. Cross-Tabulation: Awareness vs Reconcat (Point-Source Awareness)

- Free-text “Reconcat” responses listing pollution sources are:
  - split on commas,
  - counted into `source_count`,
  - treated as **0** when the user explicitly answered “No”.
- Builds:
  - a binary variable for “listed any source”
  - a 2×2 table against human-health-risk awareness
- Produces:
  - a heatmap of cell counts with row-wise percentages
  - a contingency table for reporting

This checks internal consistency between two different awareness questions.

---

## 6. Awareness vs Preference for Monitoring Research (McNemar Test)

- Converts missing entries in the “monitor PFAS in lobsters” question to `"No"` (conservative).
- Constructs paired Yes/No variables:
  - “Aware of PFAS human health risks?”
  - “Support monitoring PFAS in lobsters?”
- Runs **McNemar’s test** to detect within-individual discordance.
- Heatmap visualization displays proportion of respondents in each cell of the 2×2 matrix.

---

## 7. Ordinal Scoring for Concern/Intensity Questions

Several Likert-type items are recoded to ordinal numeric scales:

| Response              | Score |
|----------------------|--------|
| Not at all           | 0      |
| Somewhat             | 1      |
| Very                 | 2      |

Applies to questions on:
- concern about PFAS,
- involvement in co-management,
- dependence on the fishery,
- ease of establishing a new territory,
- voicing concerns.

Analysis includes:
- boxplots by age bins,
- jittered raw points,
- Kruskal–Wallis tests,
- Spearman correlations,
- optional ordinal regression.

---

## 8. Age-Based Analyses (Core Section)

For each major outcome (awareness, concern, belief, behavior intention), the script performs:

### **Age (binned, 10-year groups)**
- Boxplots of response by age bin
- Jitter overlays
- Within-bin mean markers
- Kruskal–Wallis for distributional differences across bins

### **Age as continuous**
- Logistic regression for binary outcomes  
- Linear regression for ordinal outcomes  
- Scatter plots with jitter  
- Regression formula, p-value, and R² annotated directly on plots  
- Points colored by age group for readability  

This produces consistent age-based diagnostics across all PFAS-related outcomes.

---

## 9. Behavioral Response Scenarios (PFAS Detected)

For seven behavioral-intention questions (continue fishing, sell, consume, new area, seek compensation, lobby management, leave fishery):

- Recodes Likert values to ordered factors.
- Generates:
  - raw proportion tables,
  - % stacked bar charts,
  - binary-collapsed comparisons,
  - correlation tables,
  - **Sankey diagrams** showing transitions between action pairs.

Sankeys reveal patterns such as:
- Sell vs Consume inconsistencies  
- Compensation vs Lobbying alignments  
- Movement decisions vs ease-of-new-territory beliefs  

Uses fixed `iterations = 0` to guarantee reproducibility.

---

## 10. Community-Resilience Themes (Free-Text)

- Free-text responses are separated into conceptual tags (strength, apathy, resourcefulness, etc.).
- Tags are:
  - comma-separated,
  - cleaned and normalized,
  - expanded to long-format,
  - counted and grouped.
- Visualization:
  - 100% stacked bar charts broken down by PFAS-response category
  - color-coded by conceptual grouping (positive, negative, neutral)
- Provides insight into how respondents’ resilience perceptions relate to PFAS actions.

---

## 11. Conditional Questions (ZBINARY_Y_N + Category Subfields)

For the question:

> “Have you or people you know had to stop lobstering…? If yes, how did you deal with it?”

The script:
- Extracts a Yes/No variable (`ZBINARY_Y_N`)
- Splits multi-tag follow-up categories (e.g., reasons/strategies)
- Builds:
  - tag × Yes/No contingency tables
  - stacked proportion plots
  - overall tag distribution summary

This captures variation in coping strategies among groups experiencing disruptions.

---

## 13. Outputs Produced

The script generates:

- 40–50 boxplots (age × response)
- 15+ stacked bar charts
- Multiple heatmaps and mosaic plots
- 10+ Sankey diagrams (binary and multi-level)
- Regression summaries (logistic + linear)
- Kruskal–Wallis and Spearman tests
- Fisher’s exact and McNemar’s tests
- Tag-based community-resilience charts
- Zone-level awareness visualizations

All outputs support internal consistency checks, trend detection, and PFAS-related attitude/behavior interpretation.

---

## 14. How to Run the Analysis

1. Install required packages:

```r
install.packages(c(
  "dplyr","tidyr","stringr","ggplot2","scales",
  "networkD3","htmlwidgets"
))

2. Place the cleaned CSV (or the provided lobster_clean2.csv) in the project directory.
3. Run the above script. I ran chunk by chunk and then snipped the output plots for some cleanup before presentation.
4. View figures! 


---
