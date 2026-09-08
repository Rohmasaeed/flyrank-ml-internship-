# Content Opportunity Scoring for Prioritizing Pages for Human Review

## FlyRank ML Internship — Lane 2: Refresh / Content Opportunity Scoring

### Which pages should be reviewed first for content refresh?

---

## Abstract

Content teams often need a repeatable way to identify which pages deserve attention first. This project develops a machine-learning-based content-opportunity scoring approach using historical search and engagement signals to prioritize pages for human review. Using a public-safe FlyRank internship dataset, the workflow aggregates 28-day historical performance windows and evaluates future 28-day performance changes. Random Forest achieved an average precision of **0.690**, compared with **0.639** for Logistic Regression, with strong Precision@K performance in the reported experiments. However, the current notebook contains a target-leakage issue in one feature, so these model metrics are treated as exploratory rather than fully leakage-free evidence.

---

## 1. Research Question

**Which pages should be reviewed first for refresh, expansion, protection, pruning, or monitoring based on observed search and engagement signals?**

The practical decision supported by this project is **prioritization for human review**.

The system is not intended to automatically decide whether a page should be changed. Instead, it provides a repeatable scoring approach that can help content teams focus their attention on pages showing signals associated with future performance decline.

This project does **not** claim to predict Google's ranking algorithm or establish that refreshing a page will cause its performance to improve.

---

## 2. Introduction / Problem

Large content collections can contain thousands or millions of pages, making manual review difficult to prioritize.

A useful content-review workflow therefore needs a way to identify pages that may deserve attention based on measurable historical signals.

This project investigates whether search and engagement performance can be used to build a ranking-oriented content-opportunity score.

The intended workflow is:

**Historical page performance → feature engineering → opportunity scoring → ranked pages → human review**

The output should be treated as **decision support**, not as an automated content-management decision.

---

## 3. Data

The analysis uses the public-safe FlyRank ML internship warehouse containing anonymized content-performance data.

The workflow uses:

* DuckDB for analytical querying
* Daily content-performance records
* Search performance signals
* Engagement signals
* Historical activity windows
* Future performance windows

The main data sources include:

* `dim_content.parquet`
* `fact_content_daily_performance/**/*.parquet`

A **28-day historical window** is used to describe previous page performance, followed by a **28-day future window** used to construct the observed outcome.

Only observations with sufficient activity in the historical and future windows are included.

The resulting modeling dataset contains approximately **4.23 million observations**.

The refresh-opportunity label identifies observations where at least two performance signals show a decline of approximately **20% or more** during the future window.

---

## 4. Methodology

### Historical Features

The model uses page-level historical performance signals including:

* Historical impressions
* Historical clicks
* Average search position
* Sessions
* Engaged sessions
* Engagement time
* Historical active days
* Click-through rate
* Engagement rate
* Log-transformed traffic measures
* Position risk
* Low-CTR signal
* Low-engagement signal

These features are designed to summarize the page's observed performance before the future evaluation window.

### Outcome Definition

The refresh-opportunity label is constructed from future changes in:

* Impressions
* Clicks
* Sessions

A decline signal is recorded when the future value falls by the predefined threshold relative to the historical period.

An observation is labeled as a refresh opportunity when multiple decline signals are present.

### Models

Two classification approaches were evaluated:

1. **Logistic Regression**
2. **Random Forest**

The evaluation also considers ranking-oriented metrics because the practical objective is not simply to classify every page. The more useful question is whether high-scoring pages contain a larger proportion of potential review candidates.

---

## 5. Results

The current notebook reports the following results:

| Model               | Precision | Recall | Average Precision |
| ------------------- | --------: | -----: | ----------------: |
| Logistic Regression |     0.472 |  0.960 |             0.639 |
| Random Forest       |     0.587 |  1.000 |             0.690 |

Random Forest produced the stronger average precision score in the reported experiment.

### Precision@K

|   K | Logistic Regression | Random Forest |
| --: | ------------------: | ------------: |
|  20 |                0.85 |          1.00 |
|  50 |                0.92 |          1.00 |
| 100 |                0.95 |          0.98 |

These results suggest that the model rankings can produce a concentrated shortlist of observations associated with the defined future decline label.

However, these numbers must be interpreted carefully because of the leakage issue described below.

---

## 6. Limitations & Honest Framing

### Target Leakage

The current notebook contains a feature called `declining_signal` that is derived from the future-window decline information used to construct the target.

This means that the feature contains information that would not be available at the time a real prioritization decision is made.

Therefore, the reported model metrics should **not** be presented as fully leakage-free estimates of real-world predictive performance.

The results are retained because this paper mirrors the submitted notebook and its actual outputs. The leakage is explicitly disclosed rather than hidden.

### Observational Data

The analysis is observational.

A relationship between historical performance signals and future decline does not prove that one variable causes another.

### Label Definition

The refresh-opportunity label depends on predefined decline thresholds. Changing these thresholds could change the class distribution and model performance.

### External Factors

Search performance and engagement can be affected by many factors outside the modeled data, including seasonality, competition, technical changes, search behavior, and other external events.

### Not a Google Algorithm

The model does not reproduce or predict Google's ranking algorithm.

It is a prioritization framework based on observed search and engagement signals.

### Human Review Required

A high score should be interpreted as a **reason to investigate a page**, not proof that the page needs to be refreshed, expanded, protected, or removed.

---

## 7. Ranked Recommendations

The scoring approach can support the following review workflow:

### 1. Refresh

Prioritize pages showing meaningful deterioration in observed search or engagement performance.

### 2. Expand

Investigate pages where additional useful content, coverage, or depth may improve their usefulness to users.

### 3. Protect

Monitor valuable pages that show potential performance risk before making major changes.

### 4. Prune

Investigate consistently weak or low-value pages before considering consolidation or removal.

### 5. Monitor

Continue tracking borderline or uncertain pages where the evidence is not strong enough for immediate action.

These recommendations should always be combined with editorial judgment, business context, and additional page-level investigation.

---

## 8. Reproducibility

The complete project is available in the public GitHub repository.

The repository contains the project workflow, capstone notebook, research paper source, supporting scripts, outputs, and documentation.

### Project Repository

https://github.com/Rohmasaeed/flyrank-ml-internship-

### Research Paper Source

`work/paper/research_paper.md`

### Capstone Notebook

`work/notebooks/`

The analysis uses DuckDB for large-scale analytical processing and Python-based machine-learning workflows for model training and evaluation.

The project is structured so that the analysis can be inspected and reproduced from the repository.

---

## 9. Acknowledgments & Data Credit

This project was completed as part of the **FlyRank ML Internship — Lane 2: Refresh / Content Opportunity Scoring**.

The analysis uses the public-safe FlyRank internship dataset and follows the project requirements around reproducibility, public-safe data handling, and honest interpretation of machine-learning results.

The goal of this work is to demonstrate a practical, repeatable approach for prioritizing content-review opportunities while clearly communicating the limitations of the analysis.

---

## Conclusion

This project demonstrates how historical search and engagement signals can be transformed into a repeatable content-opportunity scoring workflow.

The reported experiments show that the Random Forest model produced stronger ranking performance than Logistic Regression in the current notebook, particularly at the top of the ranked list.

At the same time, the identified target-leakage issue means the reported metrics should be treated as **exploratory evidence rather than leakage-free predictive performance**.

The appropriate real-world use of the approach is therefore as a **human-review prioritization tool**, with further validation required before deployment in a production content workflow.

