# PFAS & Lobster Fishery Perception Survey – Data Wrangling & Ranking Analysis

## 1. Project Overview

This repository contains the full data wrangling and analysis pipeline for a survey of lobster fishery stakeholders’ perceptions of PFAS contamination and other fisheries-related issues.
1. Obtain the raw data in the above repository files,
2. Recreate the cleaned dataset if you want, and
3. Reproduce all analyses and visualizations, including the categorical comparisons, regressions, ranking analysis of perceived concerns, and attempted topic modelling and sentiment analysis. Many more exploratory analyses were conducted and not included for brevity here - most variable correlations were deemed uninteresting given the low power and mixed results of comparisons, and also the domain knowledge that (mostly Phoebe) had about how the questions were answered and what kind of history drove some of the lobstermen's opinions.

To follow:
- Data provenance and cleaning steps
- See fully runnable code (R Markdown and R scripts)
- How missing data and partial rankings are handled
- In theory all modeling decisions and visualizations are reproducible
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

# 3.2 Cross-question subtraction (Q1 − Q2)

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
# 3.3 Merging Fields
```excel
=IF(ISBLANK([@Column1]),[@Column2],CONCAT([@Column2],", ",[@Column1]))
```
Final merged column = Final Q1.


# 4. Pollution Source (Q2) Cleaning
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

# 6. Binary Variables Created
| New variable                      | Original question                                | Excel rule                                                                                                                                            |
| --------------------------------- | ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **PFAS_issue_in_Maine_BINARY**    | Do you know if PFAS is an issue in Maine?        | `=IF([@[Do you know…]]="No","No","Yes")`                                                                                                              |
| **Know_someone_impacted_BINARY**  | Do you know anyone impacted by PFAS?             | `=IF([@[Do you know anyone…]]="No","No","Yes")`                                                                                                       |
| **Willing_to_communicate_BINARY** | Scientist-involvement question                   | `=IF(TEXTBEFORE([@[Would you want to work with scientists…]],",")="Would you want to further communicate with scientists on this topic?","Yes","No")` |
| **Had_to_stop_lobstering_BINARY** | Derived from long-form past-experience responses | Manual                                                                                                                                                |

# 6. Manually Interpreted Variables
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
(scroll further for python topic modelling)
#Survey Cleaning + Analysis Pipeline

Below is a detailed, step-by-step documentation of the full data-cleaning and statistical analysis pipeline, including most of the R-generated visuals in the project presentation given.
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
- Most Zone-level analyses were not used in the report due to low survey counts in many regions (almost all were in low/mid-coast zones and early visuals did not find any significant differences between the two/three).
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
  - counted into `source_count`-> ordinal/numeric count of named sources (reference is all are sources, so really a question of how many sources people were aware of - naming all/some is meaningful and not-open ended. Essentially a test of knowledge.
  - for later by-age Awareness comparisons and other variables, these 0-6 counts were kept as well in addition to the binarized version.
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

Several Likert-type items are produced from named/selected categories to ordinal numeric scales:

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

### 12. Exploratory Topic Modelling (attempt, haha).

## 1. Data Source for Text Analysis

Topic modeling focuses on long-form, open-ended responses, in particular:
from the supplementary interview transcriptions/assistant notes from some of the interviews,
text notes from general/any "Longform Notes" taken down by Phoebe or assistant and "Daily Observations" and "Methodological Reflections" and "Personal Reactions" and "Emerging Themes" and "Ethical Considerations"
were copy-pasted using the clipboard tab in excel, through the formula box shortcut to allow for single-cell pasting of many multiline items. The resulting header-added csv is a very standard format of column variables and row observations with cell contents text-only.

All of the following is implemented in Python (`pandas`) inside `tm_pfas1.ipynb` in local VSCODE and a second "long" version (see below).
---

### 2. Topic Modeling Pipeline (BERTopic) - tm_pfas1.ipynb

We use a standard BERTopic pipeline (Bobak sample, parameter-tweaked):

1. **Vectorization**

   ```python
   from sklearn.feature_extraction.text import CountVectorizer

   vectorizer_model = CountVectorizer(
       ngram_range=(1, 3),
       stop_words="english",
       min_df=1
   )
2. Dimensionality reduction & clustering

UMAP is used to embed the document representations into a low-dimensional space.

HDBSCAN clusters the UMAP embeddings into topics (with one cluster -1 reserved for outliers).

UMAP/HDBSCAN hyperparameters follow Dr. Bobak’s example code with minor tuning (e.g., neighborhood size and minimum cluster size) to produce a small number of interpretable clusters; exact values are recorded in the notebook.

3. BERTopic Model:
 ```excel
from umap import UMAP
from hdbscan import HDBSCAN
from bertopic import BERTopic
from bertopic.representation import KeyBERTInspired

rep_model = KeyBERTInspired()
topic_model = BERTopic(
    embedding_model="all-mpnet-base-v2",
    umap_model=umap_model,
    hdbscan_model=hdbscan_model,
    vectorizer_model=vectorizer_model,
    calculate_probabilities=True,
    representation_model=rep_model,
    verbose=True,
)

topics, probs = topic_model.fit_transform(documents)
```
embedding_model="all-mpnet-base-v2" provides sentence embeddings.

KeyBERTInspired creates human-readable topic labels/keywords.
4. model saving and outputs:
```excel
save_dir = r"...\tmres"
model_path = os.path.join(save_dir, "topic_model_python")
topic_model.save(model_path)
```
Main derived outputs:
Per-topic keyword table: 
```excel
topic_info = topic_model.get_topic_info()
valid_topic_ids = topic_info[topic_info.Topic != -1].Topic.tolist()

topics_data = []
for topic_id in valid_topic_ids:
    topic = topic_model.get_topic(topic_id)
    if topic is None:
        continue
    for word, weight in topic:
        topics_data.append((topic_id, word, weight))

topics_df = pd.DataFrame(topics_data, columns=["Topic", "Word", "Weight"])
csv_path = os.path.join(save_dir, "topic_keywords_2.csv")
topics_df.to_csv(csv_path, index=False)

```
This CSV is what we use for reporting example keywords per topic.
Also,
Per-document topic assignments:
```excel
df_text = df_text.reset_index(drop=True)
df_topics_docs = df_text.copy()
df_topics_docs["Topic"] = topics
```
These assignments were used qualitatively (e.g., to inspect which responses fall into each topic), but not as primary quantitative variables in the main models.

5. Visualizations (exploratory)

topic_model.visualize_topics() – 2D topic layout.

topic_model.visualize_hierarchy() – hierarchical topic tree.

topic_model.hierarchical_topics(documents) + get_topic_tree() – printed hierarchical structure.

These plots were used for interpretation only and are not required to reproduce the cleaned dataset.

### 3. Sentiment Analysis (VADER, Exploratory)

Sentiment analysis was run on selected open-ended questions (e.g., experiences shaping trust in scientists, reflections on regulations, and community responses). Code again follows Dr. Bobak’s sample, adapted to our column names.

Scoring text
Use NLTK’s SentimentIntensityAnalyzer (VADER).
For each response, compute the compound sentiment score in [-1, 1].
Store this score as a new column (e.g., sentiment_compound).

Use of sentiment variables
Sentiment scores were explored descriptively (e.g., distributions, boxplots across groups).
They were not used as primary predictors/outcomes in the main inferential models due to small sample size and interpretive complexity.

All sentiment code and column names are documented in the corresponding Python script/notebook; anyone with the raw CSV and this README can re-run it.


### Additional topic and sentiment analysis (tm_pfas_long.ipynb)
The notebook `tm_pfas_long.ipynb` extends the exploratory work described above, again using code adapted from Dr. Bobak’s sample notebooks and refit for this dataset. This section documents only the **additional** steps that are not already covered in the previous topic-model/sentiment chunk.

---

### 1. Longform Data Source

- Input file: `longform_conv1.csv`
- Key columns used:
  - `Response` – longform text (e.g., notes from conversations/interviews)
  - `Age` – respondent age (numeric)
- Pre-processing:
  - Drop rows with missing `Response`.
  - Strip whitespace and cast to `str`.
  - Store cleaned responses in a list `documents` for BERTopic.

All BERTopic settings (vectorizer, UMAP, HDBSCAN, embedding model, representation model) follow the same structure as in `tm_pfas1.ipynb`, with minor hyperparameter tweaks (e.g., `ngram_range=(1, 2)` and `n_neighbors=10`).

---

### 2. Topics Over Age (Using `topics_over_time`)

To explore how themes vary with age, the `Age` variable is used as a pseudo-“time” axis:

```python
df_text["Age"] = pd.to_numeric(df_text["Age"], errors="coerce")
df_age = df_text.dropna(subset=["Age"]).reset_index(drop=True)

documents_age = df_age["Response"].astype(str).tolist()
ages = df_age["Age"].astype(float).tolist()

updated_topics, _ = topic_model.transform(documents_age)

topics_over_time = topic_model.topics_over_time(
    documents_age,
    ages,
    updated_topics
)

fig = topic_model.visualize_topics_over_time(
    topics_over_time,
    title="Longform Topics Across Age"
)
```
This produces an exploratory figure of topic prevalence across age, not used in inferential models.

## 3. Static Intertopic Distance Map (PCA)

In addition to BERTopic’s interactive visualizations, a static intertopic distance map is created using PCA on the internal topic embeddings:
```python
from sklearn.decomposition import PCA

topic_info = topic_model.get_topic_info()
valid_topic_ids = topic_info[topic_info.Topic != -1].Topic.tolist()

emb = topic_model.topic_embeddings_[valid_topic_ids]
pca = PCA(n_components=2, random_state=42)
coords = pca.fit_transform(emb)

plt.figure(figsize=(6, 6))
plt.scatter(coords[:, 0], coords[:, 1])

for (x, y, tid) in zip(coords[:, 0], coords[:, 1], valid_topic_ids):
    plt.text(x, y, f"{tid}", fontsize=9, ha="center", va="center")

plt.xlabel("PCA 1")
plt.ylabel("PCA 2")
plt.title("Intertopic Distance Map (PCA, static)")
plt.gca().set_aspect("equal", "box")
plt.grid(True)
plt.show()
```
^This produces a paper-friendly static figure showing relative distances between topics.
## 4. Removing “Lobster” Words and Refitting Topics

To reduce trivial clustering driven by repeated domain words, a second round of topic modeling is run on text with lobster-related terms removed:
```python
import re

df["ResponseNoLobster"] = (
    df["Response"]
      .fillna("")
      .str.replace(r"\b(lobster|lobsters|lobstermen|lobstering)\b", "",
                   flags=re.IGNORECASE, regex=True)
      .str.replace(r"\s{2,}", " ", regex=True)
      .str.strip()
)

TEXT_COL = "ResponseNoLobster"
df_text = df.dropna(subset=[TEXT_COL]).copy()
df_text[TEXT_COL] = df_text[TEXT_COL].astype(str).str.strip()

documents = df_text[TEXT_COL].tolist()

# BERTopic refit with same architecture
topic_model = BERTopic(
    embedding_model="all-mpnet-base-v2",
    umap_model=umap_model,
    hdbscan_model=hdbscan_model,
    vectorizer_model=vectorizer_model,
    calculate_probabilities=True,
    representation_model=KeyBERTInspired(),
    verbose=True,
)

topics, probs = topic_model.fit_transform(documents)
```
Outputs (saved in tmres_longform1/):
topic_model_longform1 – model with lobster terms removed.
topic_keywords_longform1.csv – topic–keyword–weight table.
longform_with_topics.csv – per-response topic assignments.

## 5. Sentiment vs Age (VADER)

Using VADER sentiment on the same longform responses, the notebook explores how sentiment changes with age.
```python
from nltk.sentiment import SentimentIntensityAnalyzer
sia = SentimentIntensityAnalyzer()

df_text["Age"] = pd.to_numeric(df_text["Age"], errors="coerce")
df_sent = df_text.dropna(subset=["Age", TEXT_COL]).copy()

df_sent["sentiment"] = df_sent[TEXT_COL].apply(
    lambda x: sia.polarity_scores(str(x))["compound"]
)
```
Exploratory analyses:
Summary + correlation:
```python
print(df_sent["sentiment"].describe())
corr = df_sent[["Age", "sentiment"]].corr().loc["Age", "sentiment"]

```
Scatter + regression line:
```python
m, b = np.polyfit(df_sent["Age"], df_sent["sentiment"], 1)
# scatter of sentiment vs Age with fitted line
#no relationship at all. totally random.
```
Age-group barplot: 
```python
bins = [0, 29, 39, 49, 59, 69, 79, 120]
labels = ["<30", "30–39", "40–49", "50–59", "60–69", "70–79", "80+"]

df_sent["AgeGroup"] = pd.cut(df_sent["Age"], bins=bins, labels=labels, right=True)
sent_by_group = df_sent.groupby("AgeGroup")["sentiment"].mean().reset_index()
# bar plot of mean sentiment by AgeGroup
#also not very meaningful. i guess thats meaning in itself tho.
```
These are descriptive only and not used in the main inferential analysis.
## 6. Sentiment by Topic (ResponseNoLobster)
Finally, sentiment is examined across BERTopic topics using the lobster-removed text:
```python
TEXT_COL = "ResponseNoLobster"

df_text = df.dropna(subset=[TEXT_COL]).copy()
df_text[TEXT_COL] = df_text[TEXT_COL].astype(str).str.strip()
df_text = df_text.reset_index(drop=True)

df_topics_docs = df_text.copy()
df_topics_docs["Topic"] = topics  # from lobster-removed BERTopic model

# VADER sentiment
df_topics_docs["sentiment"] = df_topics_docs[TEXT_COL].apply(
    lambda x: sia.polarity_scores(str(x))["compound"]
)

valid = df_topics_docs[df_topics_docs["Topic"] != -1].copy()
topic_sent_summary = (
    valid.groupby("Topic")["sentiment"]
         .agg(["count", "mean", "std", "median", "min", "max"])
         .reset_index()
)
```
Visualization and test:
Boxplot of sentiment by topic with a horizontal line at 0 (neutral).
Kruskal–Wallis test to assess whether sentiment distributions differ across topics.
Again, these sentiment–topic comparisons are exploratory and serve as qualitative context rather than primary quantitative findings.


### 4. Role of Topic & Sentiment Analyses

Both the BERTopic topics and VADER sentiment scores and visuals are used:
as exploratory tools to better understand themes and tones in qualitative responses, and
as context for interpreting the quantitative survey findings.

They do not drive the core inferential results (regression models, ranking analyses), but the full pipeline is included here and in the notebook to support transparency and reproducibility. A few of the output plots were shown in the presentation for proof of attempt and showing that while technically separable into categories, the topics and other modelling were unsuccessful at capturing meaningful clusters (they all had the same words/topics even after filtering out filler words and variations of the word Lobster.)


## 13. Outputs Produced

The script(s) generates:

- 40–50 boxplots (age × response)
- 15+ stacked bar charts
- Multiple heatmaps and mosaic plots
- 10+ Sankey diagrams (binary and multi-level)
- Regression summaries (logistic + linear)
- Kruskal–Wallis and Spearman tests
- Fisher’s exact and McNemar’s tests
- Tag-based community-resilience charts
- Zone-level awareness visualizations
- Exploratory Topic hierarchies, PCA visuals, and boxplots/scatters for initial (failed) interesting topic visuals.

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

5. for topic modelling, use the longform_conv1.csv file (interview docs likely won't be released for privacy reasons, but this should contain all needed info).


---
