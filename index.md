---

layout: default
title: Content Opportunity Scoring
----------------------------------

# Content Opportunity Scoring

## FlyRank ML Internship — Lane 2

### Which pages should be reviewed first for content refresh?

This research project develops a repeatable **content-opportunity scoring approach** to help prioritize pages for human review based on observed search and engagement signals.

The goal is not to predict Google's algorithm or claim causality. Instead, the approach uses historical performance signals to identify pages that may deserve attention for **refresh, expansion, protection, pruning, or monitoring**.

---

## Research Question

**Which pages should be reviewed first for refresh, expansion, protection, pruning, or monitoring based on observed search and engagement signals?**

---

## Approach

The project uses historical content-performance data and a 28-day historical window to construct page-level features. A future 28-day window is used to define the observed outcome.

The modeling workflow compares machine-learning approaches for identifying pages associated with future performance declines and evaluates their usefulness for prioritization.

### Key principles

* Historical information is used to construct model features.
* Future performance is used only to define the evaluation outcome.
* Target leakage is avoided by excluding outcome-derived features.
* Evaluation focuses on ranking and prioritization, not only overall classification accuracy.
* Results are presented as decision support for human review rather than automated SEO decisions.

---

## Dataset

The analysis uses the public-safe FlyRank internship warehouse containing anonymized content-performance data.

The workflow uses:

* DuckDB for analytical querying
* Daily content-performance records
* Search performance signals
* Engagement signals
* Historical activity windows
* Future performance windows

The dataset and processing workflow are documented in the project repository.

---

## Methodology

The analysis aggregates historical page performance over a 28-day window and creates features such as:

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

A binary refresh-opportunity label is created from observed future declines across multiple performance signals.

The final model uses **historical-only features** so that information from the future outcome does not leak into the prediction process.

---

## Results

The final quantitative results are reported in the research paper after the leakage-safe model has been rerun.

The analysis evaluates models using ranking-oriented measures such as **Precision@K** and average precision, because the practical goal is to identify a manageable shortlist of pages for human review.

> **Important:** Results should be interpreted as observed associations and prioritization signals, not causal effects or predictions of Google's ranking algorithm.

---

## Limitations

This project has several important limitations:

1. The dataset is observational and does not establish causality.
2. Search and engagement performance can be affected by many external factors.
3. The refresh-opportunity label is based on predefined decline thresholds.
4. A model score does not guarantee that a page needs updating.
5. Human review remains necessary before taking content actions.
6. Results may vary across clients, content types, and time periods.

---

## Recommendations

The resulting scores are intended to support a practical review queue.

Pages with stronger opportunity signals can be prioritized for:

1. **Refresh** — review declining or weakening content.
2. **Expansion** — investigate opportunities to improve coverage.
3. **Protection** — monitor valuable pages showing risk signals.
4. **Pruning** — investigate consistently weak or low-value content.
5. **Monitoring** — continue tracking uncertain or borderline pages.

These recommendations should be combined with editorial judgment and business context.

---

## Reproducibility

The complete project is available in the GitHub repository, including:

* Capstone notebook
* Research paper
* Data-processing workflow
* Project documentation
* Output artifacts
* Reproducibility information

### Project Repository

[View the FlyRank ML Internship repository →](https://github.com/Rohmasaeed/flyrank-ml-internship-)

### Research Paper Source

[View the research paper source →](work/paper/research_paper)

---

## Acknowledgments & Data Credit

This project was completed as part of the **FlyRank ML Internship**, Lane 2: Refresh / Content Opportunity Scoring.

The analysis uses the public-safe anonymized FlyRank internship dataset and follows the provided project constraints around data handling, reproducibility, and honest interpretation.

---

### About the Author

**Rohma Saeed**
BS Data Science Student

This project demonstrates practical work in:

**Python · DuckDB · Pandas · Machine Learning · Data Analysis · Content Analytics · Research Communication**

---

**Research focus:** Content Opportunity Scoring for Human Review
**Internship lane:** Lane 2 — Refresh / Content Opportunity Scoring
