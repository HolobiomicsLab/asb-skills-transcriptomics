---
name: pseudotime-similarity-evaluation
description: 'Use when benchmarking cell lineage construction from time-course scRNA-seq:
  you have cells sampled at adjacent time points and want to quantify whether aligned
  trajectories show higher cell-to-cell similarity (smaller pseudo-time changes) than
  a baseline alignment.'
when_to_use_negative:
- The data lack a time-course structure with adjacent time points, so no cell-to-cell
  alignment between time points can be computed
- You need bulk population-level trajectory inference rather than individual cell
  lineage quality assessment
- The expression matrix has not been aligned into cell trajectories, leaving nothing
  to compute pseudo-time similarity over
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: CellST
  role: Provides cell_name_traj() for optimal-transport cell alignment, trajectory_expression()
    and smooth_cell_expression() (SSANOVA) for trajectory construction, over which
    pseudo-time similarity is computed
  repo: https://github.com/zhanzmr/CellST
- name: readr
  role: Loads the test_data.csv expression matrix into R
- name: pot
  role: Python optimal transport package required in a conda environment for the Earth
    mover's distance alignment step
- name: CombMSC
  role: Archived R dependency required by CellST, installed from the CRAN archive
    tarball
  repo: https://cran.r-project.org/src/contrib/Archive/CombMSC/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ailsci.2023.100068
    title: Elucidating dynamic cell lineages and gene networks in time-course single
      cell differentiation
  source_content_hash: sha256:0fa914b4cc96141ec98f5882de47199d6e90f802ab6172b6f2abadf8917025ed
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zhangzmr/CellST
  repo_url: https://github.com/zhanzmr/CellST
  tools:
  - CellST
  - CombMSC
  - R
  - devtools
  - pot
  - readr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pseudotime-similarity-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates cell lineage quality in time-course scRNA-seq by comparing pseudo-time similarity of connected cells between CellST's optimal-transport alignment and a Pearson-correlation baseline, on datasets like the single-cell mouse hematopoietic system experiment.

## When to use

Use when benchmarking cell lineage construction from time-course scRNA-seq: you have cells sampled at adjacent time points and want to quantify whether aligned trajectories show higher cell-to-cell similarity (smaller pseudo-time changes) than a baseline alignment.

## When NOT to use

- The data lack a time-course structure with adjacent time points, so no cell-to-cell alignment between time points can be computed
- You need bulk population-level trajectory inference rather than individual cell lineage quality assessment
- The expression matrix has not been aligned into cell trajectories, leaving nothing to compute pseudo-time similarity over

## Inputs

- Time-course single-cell RNA-seq expression matrix (CSV, e.g., test_data.csv) with cells sampled at adjacent time points
- Cell alignment trajectories from cell_name_traj() (optimal-transport-based)
- Alternative cell alignment from the Pearson-correlation baseline

## Outputs

- Pseudo-time similarity table (pseudo_time_similarity.csv) with a method column and a similarity column for CellST vs Pearson correlation

## How to apply

Load the time-course scRNA-seq expression matrix (e.g., test_data.csv) with readr in R, then align individual cells between adjacent time points using cell_name_traj(), which solves an optimal transport problem with Earth mover's distance to obtain unique cell-to-cell trajectories. Extract per-trajectory expression with trajectory_expression() and smooth gene expression patterns with smooth_cell_expression() using the smoothing spline (SSANOVA) model. Compute cell pseudo-time along the CellST trajectories and evaluate the pseudo-time similarity of connected cells. Recompute the same pseudo-time similarity metric using a Pearson-correlation baseline alignment for the same adjacent time points — the article notes that high-dimensional, noisy, large-sample data render Pearson correlation invalid, which is the rationale for the comparison. Tabulate similarity values for both methods (e.g., pseudo_time_similarity.csv) with a method column and a similarity column.

## Related tools

- **CellST** (Provides cell_name_traj() for optimal-transport cell alignment, trajectory_expression() and smooth_cell_expression() (SSANOVA) for trajectory construction, over which pseudo-time similarity is computed) — https://github.com/zhanzmr/CellST
- **readr** (Loads the test_data.csv expression matrix into R)
- **pot** (Python optimal transport package required in a conda environment for the Earth mover's distance alignment step)
- **CombMSC** (Archived R dependency required by CellST, installed from the CRAN archive tarball) — https://cran.r-project.org/src/contrib/Archive/CombMSC/

## Examples

```
library(readr); test_data <- data.frame(read_csv("test_data.csv"), row.names = 1); cell_traj <- cell_name_traj(test_data); traj_express <- trajectory_expression(test_data, cell_traj); total_pred <- smooth_cell_expression(traj_express, npred=300)
```

## Evaluation signals

- Output file pseudo_time_similarity.csv exists and parses as CSV containing a method column and a similarity column
- CellST's pseudo-time similarity values are at least as high as the Pearson-correlation baseline's on the same dataset
- Pseudo-time changes of connected cells under CellST concentrate near zero, as reported for the mouse hematopoietic dataset (Fig. 5b and c)
- cell_name_traj() returns unique cell-to-cell trajectories (one-to-one alignment) for each adjacent time-point pair
- smooth_cell_expression() returns smoothed gene expression patterns using the SSANOVA smoothing spline model

## Limitations

- Pearson correlation is invalid as a similarity measure on high-dimensional, noisy, large-sample scRNA-seq data, so the baseline comparison reflects this weakness rather than a fair similarity metric
- No software version or dependency information for CellST is provided in the article
- No quantitative performance metrics or benchmark values are provided in the discussion to support the claimed high accuracy beyond the pseudo-time similarity comparison

## Evidence

- [results] Empirical result: [evidence span withheld — anchor status: not_found]
