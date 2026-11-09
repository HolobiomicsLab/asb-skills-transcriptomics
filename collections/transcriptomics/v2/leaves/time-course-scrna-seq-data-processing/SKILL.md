---
name: time-course-scrna-seq-data-processing
description: Use when the input is a time-course scRNA-seq expression matrix (cells
  sacrificed at each time point, no cell-to-cell correspondence) and the goal is per-cell
  lineage tracing, cell fate prediction, or dynamic gene-network/critical-gene discovery.
when_to_use_negative:
- The data are a single time point or lack a temporal design — optimal transport alignment
  between adjacent time points and trajectory construction are not applicable.
- Only a bulk population-average trajectory is needed and per-cell resolution is unnecessary;
  the article notes most existing methods already construct such bulk trajectories.
- Expression data are so low-dimensional or small-sample that classical alignment
  measures (Euclidean distance, Pearson correlation) remain valid, making the OT-based
  alignment unnecessary.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: CellST
  role: 'Core framework: cell_name_traj, trajectory_expression, gene_expression, smooth_cell_expression,
    align_cell_time, plot_linked_traj'
  repo: github.com/zhanzmr/CellST
- name: pot
  role: Python optimal-transport package used under the hood for Earth mover's distance
    cell alignment between adjacent time points
- name: CombMSC 1.4.2.1
  role: Required archived CRAN dependency installed via R CMD INSTALL for the smoothing-spline
    step
  repo: https://cran.r-project.org/src/contrib/Archive/CombMSC/
- name: devtools
  role: R package used to install CellST from GitHub
- name: readr
  role: R package used to load test_data.csv and all_umap.csv into data frames
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.ailsci.2023.100068
    title: Elucidating dynamic cell lineages and gene networks in time-course single
      cell differentiation
  source_content_hash: sha256:4e239361635cf074b66423b85c1c44599c41966755ad7f6b278e38116b1ad2e4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zhangzmr/CellST
  tools:
  - CellST
  - CombMSC
  - CombMSC 1.4.2.1
  - R
  - devtools
  - pot
  - readr
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# time-course-scrna-seq-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end CellST pipeline that turns time-course scRNA-seq expression matrices (e.g. 38,731 cells x 11,588 genes zebrafish embryogenesis Drop-seq data) into individual cell fate trajectories and dynamic gene networks, instead of a single bulk pseudotime trajectory.

## When to use

Use when the input is a time-course scRNA-seq expression matrix (cells sacrificed at each time point, no cell-to-cell correspondence) and the goal is per-cell lineage tracing, cell fate prediction, or dynamic gene-network/critical-gene discovery.

## When NOT to use

- The data are a single time point or lack a temporal design — optimal transport alignment between adjacent time points and trajectory construction are not applicable.
- Only a bulk population-average trajectory is needed and per-cell resolution is unnecessary; the article notes most existing methods already construct such bulk trajectories.
- Expression data are so low-dimensional or small-sample that classical alignment measures (Euclidean distance, Pearson correlation) remain valid, making the OT-based alignment unnecessary.

## Inputs

- time-course scRNA-seq expression matrix (e.g. test_data.csv, cells x genes; zebrafish embryogenesis Drop-seq: 38,731 cells x 11,588 genes)
- UMAP coordinate table for visualization (all_umap.csv)

## Outputs

- unique per-cell trajectories from cell_name_traj()
- per-trajectory gene expression list (traj_express) from trajectory_expression()
- smoothed individual cell fate trajectories (e.g. smoothed_cell_expression.csv) with 300 predicted time points per trajectory
- per-gene expression across trajectories (gene_express)
- linked cell lineage figure from plot_linked_traj()
- normalized developmental timeline (biotime) from align_cell_time()
- dynamic gene network edges / critical gene table (e.g. Ccr7+ migratory DC-like population markers)

## How to apply

Set up two environments: a conda env with Python 3.9 and the 'pot' optimal-transport package, plus R with the archived CombMSC_1.4.2.1 package (installed via R CMD INSTALL from the CRAN Archive link) and CellST installed with devtools::install_github. Load the expression matrix with readr and run cell_name_traj(), which aligns cells between adjacent time points via optimal transport (Earth mover's distance) and returns unique trajectories for individual cells; extract per-trajectory expression with trajectory_expression() and per-gene expression across trajectories with gene_expression(). Smooth the aligned lineages with smooth_cell_expression(npred=300), which fits smoothing-spline SSANOVA mixed-effect models with the smoothing parameter chosen by Generalized Cross-Validation, yielding smoothed individual cell fate trajectories on a 300-point prediction grid. Optionally normalize the developmental timeline with align_cell_time() and visualize linked lineages with plot_linked_traj(ntraj=10, random_seed=2022). Downstream, build dynamic gene networks with functional concurrent linear models where an edge exists when the dynamic coefficient beta(l,s)(t) has a non-zero Bayesian confidence interval, and test differentially expressed genes via functional ANOVA (FMANOVA); critical genes like MYF5, MYOD1, and KRT18 in zebrafish trajectory groups should match network-derived critical genes.

## Related tools

- **CellST** (Core framework: cell_name_traj, trajectory_expression, gene_expression, smooth_cell_expression, align_cell_time, plot_linked_traj) — github.com/zhanzmr/CellST
- **pot** (Python optimal-transport package used under the hood for Earth mover's distance cell alignment between adjacent time points)
- **CombMSC 1.4.2.1** (Required archived CRAN dependency installed via R CMD INSTALL for the smoothing-spline step) — https://cran.r-project.org/src/contrib/Archive/CombMSC/
- **devtools** (R package used to install CellST from GitHub)
- **readr** (R package used to load test_data.csv and all_umap.csv into data frames)

## Examples

```
library(readr); test_data <- read_csv("test_data.csv"); cell_traj <- cell_name_traj(data.frame(test_data, row.names=1)); traj_express <- trajectory_expression(test_data, cell_traj); total_pred <- smooth_cell_expression(traj_express, npred=300)
```

## Evaluation signals

- The conda environment with 'pot' (Python 3.9), CombMSC_1.4.2.1, and devtools-installed CellST installs without error.
- cell_name_traj() returns unique trajectories covering individual cells across the Drop-seq time course (zebrafish dataset: 38,731 cells).
- smooth_cell_expression(npred=300) returns smoothed expression for each trajectory on exactly 300 predicted time points per trajectory.
- Reported dynamic network edges correspond to gene pairs whose dynamic coefficient beta(l,s)(t) has a non-zero Bayesian confidence interval.
- Recovered critical genes match expected biology: MYF5 and MYOD1 significantly higher in trajectory group 5 and KRT18 highly expressed in group 15.

## Limitations

- CombMSC is archived on CRAN and must be manually downloaded from the CRAN Archive link and installed with R CMD INSTALL CombMSC_1.4.2.1.tar.gz.
- A conda environment with Python 3.9 and 'pot' must be set up before using the optimal-transport alignment function.
- Time-course scRNA-seq sacrifices cells at each time point, so no true cell-to-cell correspondence exists; trajectories are inferred by optimal transport rather than observed lineage.
- No software version or dependency information for CellST is provided, and no quantitative performance metrics or benchmark values are given in the discussion.
- Cells within the same time point may be at different developmental stages, so align_cell_time() normalization of the timeline is needed.

## Evidence
