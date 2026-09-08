# Content Opportunity Scoring

## FlyRank ML Internship — Lane 2

### Which pages should be reviewed first for content refresh?

This research project develops a repeatable **content-opportunity scoring approach** to help prioritize pages for human review based on observed search and engagement signals.

The goal is not to predict Google's algorithm or claim causality. Instead, the approach uses historical performance signals to identify pages that may deserve attention for **refresh, expansion, protection, pruning, or monitoring**.

---

## Research Question

**Which pages should be reviewed first for refresh, expansion, protection, pruning, or monitoring based on observed search and engagement signals?**

---

## Introduction / Problem

Content teams need a repeatable way to decide which pages deserve attention first. This project develops a scoring approach that uses historical search and engagement signals to prioritize pages for human review.

The output is intended as decision support rather than an automated content decision system.

---

## Data

The analysis uses the public-safe FlyRank internship warehouse containing anonymized content-performance data.

The workflow uses:

* DuckDB for analytical querying
* Daily content-performance records
* Search performance signals
* Engagement signals
* Historical activity windows
* Future performance windows

The analysis uses a 28-day historical window and a 28-day future window.

---

## Methodology

Historical page performance is aggregated over a 28-day window.

The model uses historical-only features including:

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

A refresh-opportunity label is created from observed future declines across multiple performance signals.

Outcome-derived features are excluded from the final model to prevent target leakage.

---

## Results

The final model results will be reported using the leakage-safe model after the corrected notebook has been rerun.

Evaluation focuses on ranking-oriented measures such as **Precision@K** and average precision because the practical goal is to identify a manageable shortlist of pages for human review.

Results are interpreted as observed associations and prioritization signals, not causal effects or predictions of Google's ranking algorithm.

---

## Limitations & Honest Framing

This analysis is observational and does not establish causality.

Search and engagement performance can be affected by many external factors. The refresh-opportunity label depends on predefined decline thresholds, and a high model score does not guarantee that a page needs updating.

The results should therefore be treated as **decision support for human review**.

---

## Ranked Recommendations

The resulting scores can support the following review priorities:

1. **Refresh** — investigate pages showing meaningful performance deterioration.
2. **Expand** — investigate opportunities to improve useful content coverage.
3. **Protect** — monitor valuable pages showing potential risk.
4. **Prune** — investigate consistently weak or low-value content.
5. **Monitor** — continue tracking uncertain or borderline pages.

These recommendations should be combined with editorial judgment and business context.

---

## Reproducibility

The complete project is available in the GitHub repository, including the capstone notebook, research paper, data-processing workflow, and project documentation.

**Project Repository:**
https://github.com/Rohmasaeed/flyrank-ml-internship-

**Research Paper Source:**
work/paper/research_paper.md

---

## Acknowledgments & Data Credit

Built on the **FlyRank ML Internship dataset**.

This project was completed as part of the FlyRank ML Internship, Lane 2: Refresh / Content Opportunity Scoring.

The analysis follows the project requirements around public-safe data handling, reproducibility, and honest interpretation.

**Internship lane:** Lane 2 — Refresh / Content Opportunity Scoring
