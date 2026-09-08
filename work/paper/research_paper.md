---
layout: default
title: Content Opportunity Scoring for Prioritizing Pages for Human Review
---

# Content Opportunity Scoring for Prioritizing Pages for Human Review

## Abstract

This study asks which content pages should be prioritized for human review when observed search and engagement performance deteriorates. Using the FlyRank ML Internship warehouse, page-level performance signals were aggregated into repeated 28-day historical feature windows and compared with subsequent 28-day performance windows. A transparent rule-based baseline was compared with Logistic Regression and Random Forest models using chronological validation. In the reported notebook results, Random Forest achieved the strongest ranking performance, with an Average Precision of 0.690 compared with 0.639 for Logistic Regression, and Precision@20 and Precision@50 of 1.00 on the chronological test set. These results are treated as an exploratory ranking experiment rather than evidence of causal impact, and a target-leakage issue identified in one feature means the reported machine-learning metrics should not be interpreted as a fully leakage-free estimate of real-world performance.

---

# 1. Introduction / Problem Statement

Content teams may have many pages that could potentially benefit from review, but reviewing every page with the same priority is inefficient.

The purpose of this project is to develop a repeatable content-opportunity scoring system that uses observed search and engagement signals to identify pages that should be considered for human review.

The research question is:

> **Which pages should be reviewed first for refresh, expansion, protection, pruning, or monitoring based on observed search and engagement signals?**

The decision supported by this analysis is the prioritization of pages for human review.

The system is not intended to automatically decide whether a page should be changed. Instead, it creates a ranked queue that can help a reviewer decide where to begin.

This capstone follows **Lane 2: Refresh / Content Opportunity Scoring** of the FlyRank ML Internship.

The analysis focuses on observed historical performance and subsequent deterioration. It does not claim that refreshing a page causes future performance improvement, and it does not attempt to establish causal relationships with search-engine ranking algorithms.

---

# 2. Data

## 2.1 Dataset

The analysis uses the **FlyRank ML Internship warehouse** provided as part of the internship dataset release.

The warehouse was queried using DuckDB. The full daily-performance release contains tens of millions of records, so the analytical workflow queries the warehouse directly rather than loading the complete dataset into pandas.

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

The analysis covers the available modeling period from late 2025 through mid-2026.

Cutoff dates are generated at 14-day intervals. The final chronological test observations include cutoff dates through **May 27, 2026**.

Pages are required to have at least 14 active days in both the historical and future windows. This reduces the influence of pages with very sparse observations.

## 2.3 Page-Level Aggregation

Daily records are aggregated by reporting date and content identifier.

| Metric           | Aggregation |
| ---------------- | ----------- |
| Impressions      | Sum         |
| Clicks           | Sum         |
| Average position | Mean        |
| Sessions         | Sum         |
| Engaged sessions | Sum         |
| Engagement time  | Sum         |

This produces a page-level daily modeling table that is then used to construct repeated historical and future windows.

## 2.4 Public-Safe Data Handling

The analysis does not use client names, domains, URLs, private search queries, credentials, or raw client-level exports as predictive information.

Client identifiers and URLs are not used as model features.

The public presentation uses pseudonymized content identifiers and aggregated performance signals.

---

# 3. Methodology

## 3.1 Historical Feature Window

For every page and cutoff date, features are calculated from the preceding 28 days.

The modeled historical signals include:

* historical impressions
* historical clicks
* historical average search position
* historical sessions
* engaged sessions
* engagement time
* historical active days
* click-through rate
* engagement rate
* log-transformed traffic measures
* position risk
* low-CTR signal
* low-engagement signal
* a declining-signal feature present in the reported notebook experiment

These features are intended to represent page visibility, traffic scale, engagement behavior, and potential areas for review.

## 3.2 Future Outcome Window

The following 28 days are used to measure subsequent observed performance.

The future window is used to construct the target label based on changes in impressions, clicks, and sessions.

The intended modeling design is that future performance should be reserved for outcome construction rather than prediction.

However, an important implementation issue was identified in the reported notebook: the `declining_signal` feature was derived from future-window decline information and therefore overlaps with information used to construct the target.

This creates **target leakage**.

Consequently, the reported machine-learning metrics should be interpreted as results from the implemented notebook experiment rather than as a clean leakage-free estimate of future ranking performance.

## 3.3 Label Definition

A page is classified as a **refresh-opportunity page** when at least two of three major performance signals show a decline of 20% or more between the historical and future windows.

The three signals are:

1. impressions
2. clicks
3. sessions

For each signal, relative change is calculated between the historical and future periods.

A decline signal is counted when the relative change is less than or equal to **-20%**.

The final target is:

> **Refresh Opportunity = 1 when at least two major signals decline by 20% or more; otherwise 0.**

This is a constructed decision-support label rather than a direct measurement of whether a page objectively requires a content refresh.

## 3.4 Feature Engineering

The reported modeling dataset contains **4,228,038 observations and 16 features**.

The feature set used in the notebook experiment includes:

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

The target distribution is:

| Class                  |     Count |
| ---------------------- | --------: |
| No refresh opportunity | 3,868,857 |
| Refresh opportunity    |   359,181 |

The `declining_signal` feature is acknowledged as a leakage issue because it incorporates information related to the future decline used to define the target.

## 3.5 Baseline

A transparent rule-based baseline was used as an interpretable reference point.

The baseline combines observable opportunity indicators such as:

* declining signal
* position risk
* low CTR signal
* low engagement signal

The purpose of the baseline is to provide a simple comparison against supervised ranking models.

Because the declining signal itself is affected by the same future information used to construct the target, the baseline should also be regarded as exploratory rather than a leakage-free benchmark.

## 3.6 Models

Two supervised learning models were evaluated.

### Logistic Regression

The Logistic Regression pipeline uses:

* StandardScaler
* LogisticRegression
* `class_weight="balanced"`
* maximum iterations of 1,000
* fixed random state for reproducibility

### Random Forest

The Random Forest configuration uses:

* 150 trees
* maximum depth of 10
* minimum 20 samples per leaf
* balanced class weighting
* fixed random state
* parallel processing

## 3.7 Chronological Validation

Because the problem is time-dependent, the data was not randomly split.

Instead, unique cutoff dates were ordered chronologically. Earlier cutoff observations were used for training, while later cutoff observations were reserved for testing.

### Training Cutoffs

The training period contains cutoff dates from:

**October 29, 2025 through April 1, 2026.**

### Testing Cutoffs

The chronological test period contains:

* April 15, 2026
* April 29, 2026
* May 13, 2026
* May 27, 2026

This validation structure better reflects the intended use case of using previously observed information to prioritize future review opportunities.

---

# 4. Results

The models were evaluated on the same chronological test period.

The evaluation focuses on ranking quality because the practical objective is to identify a manageable shortlist of pages for human review rather than to treat every observation equally.

The reported metrics include:

* Precision
* Recall
* Average Precision
* Precision@K
* Recall@K

Accuracy is not treated as the primary success criterion because the operational task is prioritization.

## 4.1 Overall Model Performance

| Model                | Precision | Recall | Average Precision |
| -------------------- | --------: | -----: | ----------------: |
| Transparent baseline |         — |      — |                 — |
| Logistic Regression  |     0.472 |  0.960 |         **0.639** |
| Random Forest        |     0.587 |  1.000 |         **0.690** |

In the reported notebook experiment, Random Forest achieved the strongest Average Precision among the evaluated machine-learning models.

Its Average Precision was approximately **0.690**, compared with **0.639** for Logistic Regression.

These values are reported exactly as observed in the existing notebook. Due to the target-leakage issue described in Section 5, they should not be interpreted as unbiased estimates of leakage-free real-world performance.

## 4.2 Ranking Evaluation

Because the practical objective is to prioritize a review queue, top-of-ranking performance was also evaluated.

|   K | Logistic Regression Precision@K | Random Forest Precision@K |
| --: | ------------------------------: | ------------------------: |
|  20 |                            0.85 |                  **1.00** |
|  50 |                            0.92 |                  **1.00** |
| 100 |                            0.95 |                  **0.98** |

The reported Random Forest ranking places a high concentration of observed refresh-opportunity cases near the top of the ranking.

Precision@20 and Precision@50 were both **1.00**, while Precision@100 was **0.98** on the chronological test set.

Recall@K is much smaller because K represents only a tiny fraction of the millions of test observations. Therefore, Average Precision and top-K precision provide more useful measures for the intended prioritization task.

Again, these ranking results are exploratory because of the leakage identified in the feature construction.

## 4.3 Ranking Output

The Random Forest score is used to create a ranked review queue.

Each recommendation can contain:

* rank
* pseudonymized content identifier
* model score
* suggested action
* reason codes

The highest-ranked pages form the initial review queue.

The score determines prioritization, while reason codes provide additional context for human reviewers.

---

# 5. Limitations & Honest Framing

The most important limitation of the reported experiment is **target leakage**.

The notebook includes a `declining_signal` feature that is derived from future-window decline information. The same future-window decline information contributes to the construction of the refresh-opportunity target.

This means that the model has access to information that would not be available at the time a real-world ranking decision is made.

Therefore, the reported Average Precision and Precision@K values should **not** be interpreted as reliable estimates of production performance.

This limitation does not invalidate the broader analytical idea of using historical search and engagement signals for content prioritization, but it means that the reported model metrics are best treated as an exploratory result from the implemented experiment.

A leakage-free version would remove the outcome-derived feature and rerun the complete training and evaluation workflow before the model could be considered ready for production use.

Other limitations also apply.

This analysis is observational and does not establish that refreshing a page will cause its performance to improve.

The target is a constructed decision-support label based on observed changes in impressions, clicks, and sessions. It is therefore not a direct measurement of whether a page actually needed a content refresh.

The dataset contains aggregated search and engagement signals rather than detailed editorial information. Factors such as:

* content quality
* search intent
* competition
* seasonality
* technical issues
* external events

may influence observed performance.

The model should therefore be interpreted as a **prioritization experiment rather than an automated decision maker**.

The analysis also does not prove anything about Google's ranking algorithm and does not establish causal relationships between content changes and search performance.

---

# 6. Ranked Recommendations

The ranking system is intended to help a reviewer decide where to begin investigation.

The following action categories provide a practical framework for interpreting high-priority pages.

## Refresh / Review

Pages showing meaningful deterioration across multiple performance signals can receive higher review priority.

A reviewer can investigate whether content freshness, relevance, structure, or intent alignment may warrant attention.

## CTR Review

Pages receiving search visibility but comparatively weak click capture can be investigated for potential title, snippet, metadata, or search-intent opportunities.

## Visibility Review

Pages with weaker observed average search position can be investigated for:

* content depth
* relevance
* internal linking
* competitive changes

## Engagement Review

Pages with relatively weak engagement signals can be reviewed for:

* content usefulness
* structure
* intent alignment
* user experience

## Monitor

Pages without strong deterioration signals can remain in a monitoring queue rather than being automatically changed.

These categories are **human-review recommendations**, not automated editorial decisions.

---

# 7. Reproducibility

The analysis was developed in Python using DuckDB and scikit-learn.

The warehouse was queried directly using DuckDB rather than loading the full daily-performance release into memory.

The modeling workflow consists of:

1. Aggregating daily page-level performance.
2. Creating repeated 28-day historical feature windows.
3. Creating subsequent 28-day outcome windows.
4. Constructing the refresh-opportunity label from observed deterioration.
5. Performing chronological train/test validation.
6. Comparing a transparent rule-based approach with Logistic Regression and Random Forest.
7. Evaluating ranking quality using Average Precision and Precision@K.
8. Generating a ranked review queue with reason codes and action recommendations.

## Project Artifacts

* **Capstone notebook:** `work/notebooks/capstone.ipynb`
* **Research paper:** `work/paper/research_paper.md`
* **Recommendation artifact:** `top20_content_recommendations.csv`
* **Model comparison:** `model_results.csv`

The notebook provides the executable record of the analysis and documents the data-processing, modeling, and evaluation workflow.

---

# 8. Artifacts and Visual Evidence

The analysis produces artifacts that support the modeling and recommendation workflow.

### Model Comparison

A model comparison visualization summarizes the ranking performance of the evaluated approaches. In the reported experiment, Random Forest achieved the highest Average Precision among the machine-learning models.

### Score Distribution

The score distribution illustrates how model ranking scores are distributed across the chronological test observations.

### Top 20 Review Priorities

The top-20 recommendation artifact identifies the highest-ranked pseudonymized content observations and their associated model scores.

### Action Distribution

The action distribution summarizes the suggested review categories generated by the recommendation workflow.

These artifacts provide visual and tabular evidence for the exploratory evaluation and demonstrate how model scores can be translated into a human-review queue.

---

# 9. Acknowledgments & Data Credit

This project was developed as part of the **FlyRank ML Internship capstone**, Lane 2: **Refresh / Content Opportunity Scoring**.

The analysis uses the **FlyRank internship dataset** and follows the public-safe requirements of the assignment.

The project uses pseudonymized identifiers and aggregated performance signals. It does not expose client names, domains, URLs, private queries, credentials, or raw client exports.

**Data source:** FlyRank

The work is intended as an independent analytical and modeling exercise using the provided internship warehouse.

The reported results represent the behavior of the implemented notebook experiment. Because a target-leakage issue was identified in the current feature set, the machine-learning metrics should be treated as exploratory and should not be interpreted as production-ready estimates.

The project does not claim causal evidence about search-engine algorithms or the effect of content changes.
