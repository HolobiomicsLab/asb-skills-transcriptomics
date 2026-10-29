---
name: linear-regression-fitting
description: Use when you have DOI scores for cell-type pairs from a CI-graph plus
  Spaco-assigned colors, and need to quantify whether color differences track spatial
  interlacement via OLS regression.
when_to_use_negative:
- You have not yet run Spaco's color assignment on the STARmap data, so no DOI scores
  or assigned colors exist to correlate
- The goal is to perform or tune the colorization itself (e.g., palette selection
  or cluster-color assignment) rather than to validate that the DOI drove the optimization
- The cell-type pairs have no spatial interlacement structure to model (e.g., non-spatial
  or bulk data where the CI-graph/DOI metric is undefined)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
tools:
- name: Spaco
  role: Provides the CI-graph DOI scores and Spaco-assigned colors that are regressed
    against each other
  repo: https://github.com/BrainStOrmics/Spaco
- name: Python
  role: Execution environment for the CIEDE2000 color-difference computation, OLS
    fit, and CSV output
- name: matplotlib
  role: Draws the scatter plot of DOI vs. color difference with the fitted regression
    line
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:7d29c9f6f38571f04d5927d50ac0adb6e4213e920adbf868a0d3ba53f9b1c85b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - matplotlib
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# linear-regression-fitting

## Summary

Fit an ordinary least-squares linear regression between Spaco's Degree of Interlacement (DOI) scores for cell-type pairs and the realized pairwise color differences (CIEDE2000) of Spaco-assigned colors, to validate that Spaco's optimization exploited the CI-graph DOI metric. Applied here to reproduce the STARmap benchmark statistics R-square = 0.45, p = 4e-61.

## When to use

Use when you have DOI scores for cell-type pairs from a CI-graph plus Spaco-assigned colors, and need to quantify whether color differences track spatial interlacement via OLS regression.

## When NOT to use

- You have not yet run Spaco's color assignment on the STARmap data, so no DOI scores or assigned colors exist to correlate
- The goal is to perform or tune the colorization itself (e.g., palette selection or cluster-color assignment) rather than to validate that the DOI drove the optimization
- The cell-type pairs have no spatial interlacement structure to model (e.g., non-spatial or bulk data where the CI-graph/DOI metric is undefined)

## Inputs

- DOI scores for cell-type pairs derived from the Spaco CI-graph (STARmap run, task_002 output)
- Spaco-assigned colors per cell type from the STARmap color-assignment run
- Python environment with scipy>=1.10.0, pyciede2000==0.0.21, matplotlib, Spaco installed from github.com/BrainStOrmics/Spaco

## Outputs

- doi_color_regression.csv containing the regression R-square and p-value
- Optional scatter plot of DOI score vs. realized color difference with fitted regression line (matplotlib)

## How to apply

Load the DOI scores for cell-type pairs from the CI-graph of the STARmap color-assignment run and the Spaco-assigned colors. Compute realized pairwise color differences in a perceptual color space using CIEDE2000 (pyciede2000==0.0.21, matching Spaco's dependency). Pair each realized color difference with the corresponding DOI score for the same cell-type pair, then fit an ordinary least-squares regression of color difference on DOI with scipy.stats (scipy>=1.10.0). Record R-square and p-value into doi_color_regression.csv and optionally draw a scatter plot with the fitted line using matplotlib. The rationale is that Spaco optimizes color assignment based on the CI-graph topology, so a significant positive linear association between DOI and color difference demonstrates the DOI drove the color mapping.

## Related tools

- **Spaco** (Provides the CI-graph DOI scores and Spaco-assigned colors that are regressed against each other) — https://github.com/BrainStOrmics/Spaco
- **Python** (Execution environment for the CIEDE2000 color-difference computation, OLS fit, and CSV output)
- **matplotlib** (Draws the scatter plot of DOI vs. color difference with the fitted regression line)

## Examples

```
from scipy.stats import linregress; res = linregress(doi_scores, ciede2000_diffs); print(res.rvalue**2, res.pvalue)
```

## Evaluation signals

- Reproduced benchmark statistics: R-square = 0.45 (validation window 0.40-0.50) and p = 4e-61 (same order of magnitude)
- Regression slope is positive, consistent with higher interlacement driving larger assigned color differences
- Each realized CIEDE2000 color difference is correctly paired with the DOI score of the same cell-type pair (no index misalignment)
- doi_color_regression.csv is produced and contains both R-square and p-value fields
- Color differences are computed in a perceptual color space with CIEDE2000 consistent with Spaco's pyciede2000==0.0.21 dependency

## Limitations

- An R-square of 0.45 means the linear model explains less than half the variance; DOI and color difference are associated but not deterministically linked
- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco, potentially shifting the DOI/color inputs feeding the regression
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions

## Evidence

- [results] DOI-color regression benchmark statistic: [evidence span withheld — anchor status: not_found]
- [abstract] DOI metric and CI-graph construction: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [methods] Color assignment optimization rationale: "By quantifying the complex topology between cell type clusters, We optimized color assignment of to achieve better visual recognizability."
- [readme] Spaco installation: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
