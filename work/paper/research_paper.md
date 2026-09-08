# Content Opportunity Scoring for Prioritizing Pages for Human Review

## Abstract

This study asks which content pages should be prioritized for human review when observed search and engagement performance deteriorates. Using the FlyRank ML Internship warehouse, page-level performance signals were aggregated into repeated 28-day historical feature windows and compared with subsequent 28-day performance windows. A transparent decline-based baseline was compared with Logistic Regression and Random Forest models using chronological validation to reduce temporal leakage. Random Forest achieved the strongest ranking performance, with an Average Precision of 0.690 compared with 0.639 for Logistic Regression, while achieving Precision@20 and Precision@50 of 1.00 on the chronological test set. The resulting system produces a ranked review queue with interpretable reason codes and is intended as directional decision support rather than evidence that refreshing a page will causally improve future performance.

---

# 1. Introduction / Problem Statement

Content teams may have many pages that could potentially benefit from review, but reviewing every page with the same priority is inefficient.

The purpose of this project is to develop a repeatable content-opportunity scoring system that uses observed search and engagement signals to identify pages that should be considered for human review.

The research question is:

> **Which pages should be reviewed first for refresh, expansion, protection, pruning, or monitoring based on observed search and engagement signals?**

The decision supported by this analysis is the prioritization of pages for human review.

The system is not designed to automatically decide whether a page should be changed. Instead, it creates a ranked queue that helps a reviewer decide where to start.

This capstone follows **Lane 2: Refresh / Content Opportunity Scoring** of the FlyRank ML Internship.

The analysis focuses on observed historical performance and subsequent deterioration. It does not claim that a content refresh causes future performance improvement, nor does it attempt to establish causal relationships with search-engine ranking algorithms.

---

# 2. Data

## 2.1 Dataset

The analysis uses the **FlyRank ML Internship warehouse** available through the internship dataset release.

The warehouse was queried directly using DuckDB. The complete daily-performance release was not loaded into pandas because it contains tens of millions of rows.

The analysis uses two primary warehouse components:

* `dim_content.parquet`
* `fact_content_daily_performance/**/*.parquet`

The daily performance data contains page-level search and engagement measurements, including:

* search impressions
* search clicks
* average search position
* sessions
* engaged sessions
* total engagement time
* reporting date
* pseudonymized content identifiers

The analysis uses pseudonymized content identifiers rather than client names, domains, URLs, or private search queries.

## 2.2 Time Windows

The modeling workflow uses repeated historical and future windows.

For each cutoff date:

* **Historical feature window:** previous 28 days
* **Future outcome window:** following 28 days

The daily page-level aggregation covers the available modeling period from October 2025 through June 2026.

Cutoff dates are generated at 14-day intervals. The final chronological test observations include cutoff dates through **May 27, 2026**.

Pages are required to have at least 14 active days in both the historical and future windows. This reduces the effect of pages with very sparse observations.

## 2.3 Page-Level Aggregation

Daily records are aggregated by:

* reporting date
* content identifier

The following aggregation operations are used:

| Metric           | Aggregation |
| ---------------- | ----------- |
| Impressions      | Sum         |
| Clicks           | Sum         |
| Average position | Mean        |
| Sessions         | Sum         |
| Engaged sessions | Sum         |
| Engagement time  | Sum         |

This produces a page-level daily modeling table that can then be used to construct the repeated historical and future windows.

## 2.4 Exclusions and Public-Safe Handling

The analysis does not use client names, domains, URLs, private search queries, credentials, or raw client-level exports as predictive information.

Client identifiers and URLs are not used as model features.

The project is presented using pseudonymized content identifiers and aggregated performance signals to maintain a public-safe framing.

---

# 3. Methodology

## 3.1 Historical Feature Window

For every page and cutoff date, features are calculated using the previous 28 days.

The historical features include:

* historical impressions
* historical clicks
* historical CTR
* average search position
* historical sessions
* engaged sessions
* engagement rate
* engagement time
* number of active days

Additional transformed and indicator features are created to represent traffic scale, visibility risk, CTR opportunity, engagement opportunity, and observed deterioration.

These include:

* log-transformed impressions
* log-transformed clicks
* log-transformed sessions
* position risk
* low CTR signal
* low engagement signal
* declining signal

## 3.2 Future Outcome Window

The following 28 days are used only to measure the subsequent observed outcome.

Future-window metrics are not included as predictive model features.

This separation preserves the temporal structure of the problem and prevents future observations from directly entering the feature matrix.

## 3.3 Label Definition

A page is classified as a **refresh-opportunity page** when at least two of three major performance signals show a decline of 20% or more between the historical and future windows.

The three signals are:

1. impressions
2. clicks
3. sessions

For each signal, relative change is calculated between the historical and future periods.

A decline signal is counted when the relative change is less than or equal to -20%.

The final target is:

> **Refresh Opportunity = 1 when at least two major signals decline by 20% or more; otherwise 0.**

This is a constructed decision-support label rather than a direct measurement of whether a page objectively requires a content refresh.

## 3.4 Feature Engineering

The final feature matrix contains 16 features.

These include:

* `hist_impressions`
* `hist_clicks`
* `hist_avg_position`
* `hist_sessions`
* `hist_engaged_sessions`
* `hist_engagement_sec`
* `hist_active_days`
* `hist_ctr`
* `hist_engagement_rate`
* `log_impressions`
* `log_clicks`
* `log_sessions`
* `position_risk`
* `low_ctr_signal`
* `low_engagement_signal`
* `declining_signal`

The final modeling dataset contains **4,228,038 observations and 16 features**.

The target distribution is:

| Class                  |     Count |
| ---------------------- | --------: |
| No refresh opportunity | 3,868,857 |
| Refresh opportunity    |   359,181 |

## 3.5 Baseline

A transparent rule-based baseline is used for comparison.

The baseline score combines four observable signals:

* declining signal
* position risk
* low CTR signal
* low engagement signal

The baseline provides an interpretable reference point against which the machine-learning ranking models can be evaluated.

## 3.6 Models

Two supervised learning models are evaluated.

### Logistic Regression

A Logistic Regression pipeline is used with:

* StandardScaler
* LogisticRegression
* `class_weight="balanced"`
* maximum iterations of 1,000
* fixed random state for reproducibility

### Random Forest

A Random Forest classifier is evaluated using:

* 150 trees
* maximum depth of 10
* minimum 20 samples per leaf
* balanced class weighting
* fixed random state
* parallel processing

## 3.7 Chronological Validation

Because the problem is time-dependent, the data is not randomly split.

Instead, unique cutoff dates are ordered chronologically.

The earlier cutoff observations are used for training and the later observations are reserved for testing.

### Training Cutoffs

The training period contains cutoff dates from:

**October 29, 2025 through April 1, 2026.**

### Testing Cutoffs

The chronological test period contains:

* April 15, 2026
* April 29, 2026
* May 13, 2026
* May 27, 2026

This design helps reduce temporal leakage and better reflects the intended use case of ranking future review opportunities from previously observed information.

## 3.8 Leakage Control

Future-window metrics are used only to construct the target label.

They are not included among model features.

Client identifiers and URLs are also excluded from the predictive feature set.

The model therefore makes its ranking using historical information while the future window is reserved for evaluating the observed outcome.

---

# 4. Results

The models are evaluated on the same chronological test period.

The evaluation focuses on ranking quality because the practical objective is to identify pages for human review rather than classify every observation with equal cost.

The primary metrics are:

* Precision
* Recall
* Average Precision
* Precision@K
* Recall@K

Accuracy is not treated as the main success criterion because the operational task is prioritization.

## 4.1 Overall Model Performance

| Model                | Precision | Recall | Average Precision |
| -------------------- | --------: | -----: | ----------------: |
| Transparent baseline |         — |      — |                 — |
| Logistic Regression  |     0.472 |  0.960 |         **0.639** |
| Random Forest        |     0.587 |  1.000 |         **0.690** |

Random Forest achieves the strongest Average Precision among the evaluated machine-learning models.

Its Average Precision is approximately **0.690**, compared with **0.639** for Logistic Regression.

Random Forest is therefore selected as the final ranking model.

## 4.2 Ranking Evaluation

Because the practical objective is to prioritize a review queue, top-of-ranking performance is also evaluated.

|   K | Logistic Precision@K | Random Forest Precision@K |
| --: | -------------------: | ------------------------: |
|  20 |                 0.85 |                  **1.00** |
|  50 |                 0.92 |                  **1.00** |
| 100 |                 0.95 |                  **0.98** |

The Random Forest identifies a very high concentration of observed refresh-opportunity cases near the top of the ranking.

Precision@20 and Precision@50 are both **1.00**, while Precision@100 remains **0.98** on the chronological test set.

Recall@K is very small because K represents only a tiny fraction of the millions of test observations. Therefore, Recall@K is not used as the primary operational interpretation of the ranking.

Average Precision provides a more useful overall measure of ranking quality across the complete test set.

## 4.3 Final Ranking

The Random Forest score is used to rank the chronological test observations.

Each recommendation contains:

* rank
* pseudonymized content identifier
* model score
* suggested action
* reason codes

The highest-ranked pages form the initial review queue.

The system is designed so that the score determines priority, while the reason codes provide additional context for the reviewer.

---

# 5. Limitations & Honest Framing

This analysis identifies pages whose observed historical signals are associated with future deterioration.

It does **not** demonstrate that refreshing a page will cause its performance to improve.

The target is a constructed decision-support label based on observed changes in impressions, clicks, and sessions. It is therefore not a direct measurement of whether a page actually needed a content refresh.

The dataset contains aggregated search and engagement signals rather than detailed editorial information. Factors such as:

* content quality
* search intent
* competition
* seasonality
* technical issues
* external events

may influence observed performance.

The model should therefore be interpreted as a **prioritization system rather than an automated decision maker**.

The ranked output is best used as a queue for human review.

The analysis also does not prove anything about Google's ranking algorithm or establish causal relationships between content changes and search performance.

---

# 6. Ranked Recommendations

The final recommendation system uses the Random Forest model because it achieved the strongest Average Precision among the evaluated models.

The ranked queue is intended to help a reviewer decide where to begin investigation.

## Refresh / Review

Pages showing multiple declining performance signals receive the highest review priority.

These pages may be investigated for potential content refresh or broader editorial review.

## CTR Review

Pages with observed visibility but relatively weak click capture can be reviewed for potential title and metadata opportunities.

## Visibility Review

Pages with weaker observed average position can be investigated for:

* content depth
* relevance
* internal linking
* competitive changes

## Engagement Review

Pages with relatively low engagement signals can be reviewed for:

* content usefulness
* structure
* intent alignment
* user experience

## Monitor

Pages without strong deterioration signals can remain in the monitoring queue rather than being automatically changed.

These actions are recommendations for **human review**, not automated editorial decisions.

---

# 7. Reproducibility

The analysis was developed in Python using DuckDB and scikit-learn.

The warehouse was queried directly from the FlyRank internship dataset using DuckDB rather than loading the full daily-performance release into memory.

The modeling workflow consists of:

1. Aggregating daily page-level performance.
2. Creating repeated 28-day historical feature windows.
3. Creating subsequent 28-day outcome windows.
4. Constructing the refresh-opportunity label from observed deterioration.
5. Performing chronological train/test validation.
6. Comparing a transparent rule-based baseline with Logistic Regression and Random Forest.
7. Selecting the strongest ranking model using Average Precision.
8. Generating a ranked review queue with reason codes and action recommendations.

## Project Artifacts

* Capstone notebook: `work/notebooks/capstone.ipynb`
* Recommendation artifact: `top20_content_recommendations.csv`
* Model comparison: `model_results.csv`
* Repository: the accompanying GitHub repository containing the complete project history and notebooks.

The notebook provides the executable record of the analysis and documents the methodology and evaluation workflow.

---

# 8. Artifacts and Visual Evidence

The analysis produces several artifacts that support the reported results.

### Model Comparison

A model comparison chart shows Average Precision for the evaluated approaches, with Random Forest achieving the strongest result among the machine-learning models.

### Score Distribution

The score distribution shows how the final Random Forest ranking scores are distributed across the chronological test observations.

### Top 20 Review Priorities

The top-20 visualization shows the highest-ranked pseudonymized content identifiers and their model scores.

### Action Distribution

The action-distribution chart summarizes the suggested review categories across the ranked test observations.

These artifacts provide visual support for the evaluation and final recommendation workflow.

---

# 9. Acknowledgments & Data Credit

This project was developed as part of the **FlyRank ML Internship capstone**.

The analysis uses the **FlyRank internship dataset** and follows the public-safe requirements of the assignment.

The project uses pseudonymized identifiers and aggregated performance signals. It does not expose client names, domains, URLs, private queries, credentials, or raw client exports.

**Data source:** FlyRank

The work is intended as an independent analytical and modeling exercise using the provided internship warehouse. The results represent observed patterns in the available data and should not be interpreted as causal evidence about search-engine algorithms or the effect of content changes.
