---
name: optimal-transport-cell-lineage-alignment
description: Use when a time-course scRNA-seq matrix sacrifices cells at each time
  point (no cell-to-cell correspondence) and individual cell lineage paths between
  adjacent time points are needed before smoothing, fate prediction, or dynamic gene
  network construction.
when_to_use_negative:
- Only a single-time-point scRNA-seq dataset is available — adjacent time points are
  required for the optimal transport coupling
- A bulk population-average trajectory across time is sufficient for the analysis,
  e.g. bulk pseudotime ordering without per-cell lineage resolution
- Ground-truth cell-to-cell correspondence already exists between time points, making
  transport-based estimation unnecessary
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellST
  role: Provides cell_name_traj(), the cell lineage tracing function that aligns cells
    between adjacent time points via optimal transport
  repo: github.com/zhanzmr/CellST
- name: pot
  role: Python optimal transport package required in the 'ot' conda environment; supplies
    the Earth mover's distance solver behind the alignment
- name: readr
  role: R package used to load the CSV expression matrix (read_csv) before calling
    cell_name_traj()
- name: devtools
  role: Used to install CellST from GitHub with install_github()
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.ailsci.2023.100068
    title: Elucidating dynamic cell lineages and gene networks in time-course single
      cell differentiation
  source_content_hash: sha256:b5e0e374e7b395605acc37d93d815382be0aede5c51ca6d91846002b510ac8a8
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
  - R
  - devtools
  - fdANOVA
  - pot
  - readr
  - tradeseq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# optimal-transport-cell-lineage-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns individual cells between adjacent time points of a time-course scRNA-seq experiment by solving a discrete Monge optimal transport problem (Earth mover's distance, squared Euclidean cost, alpha=2), yielding unique cell-to-cell trajectories that no bulk pseudotime method provides.

## When to use

Use when a time-course scRNA-seq matrix sacrifices cells at each time point (no cell-to-cell correspondence) and individual cell lineage paths between adjacent time points are needed before smoothing, fate prediction, or dynamic gene network construction.

## When NOT to use

- Only a single-time-point scRNA-seq dataset is available — adjacent time points are required for the optimal transport coupling
- A bulk population-average trajectory across time is sufficient for the analysis, e.g. bulk pseudotime ordering without per-cell lineage resolution
- Ground-truth cell-to-cell correspondence already exists between time points, making transport-based estimation unnecessary

## Inputs

- Normalized time-course scRNA-seq expression matrix (CSV, e.g. test_data.csv), rows/cells x columns/genes, with adjacent-time-point cells to be aligned
- Python 3.9 conda environment with the 'pot' optimal transport package

## Outputs

- Cell-to-cell coupling / aligned lineage assignment for each source cell (permutation-style unique trajectories across adjacent time points)
- cell_traj object consumable by trajectory_expression() and plot_linked_traj()

## How to apply

Load the normalized gene expression matrix (e.g. test_data.csv via readr) and treat each cell as a d-dimensional gene-expression vector; estimate empirical gene feature distributions mu_t and mu_t+1 as mixtures of Dirac measures, one per cell. Run CellST's cell_name_traj(), which uses Earth mover's distance: a Python 3.9 conda environment with the 'pot' package must exist, and the solver minimizes the quadratic transportation cost ||x_tj - x_t+1,i||^2 between the two time points. For equal-sized time points this reduces to a one-to-one assignment represented by a permutation matrix; apply the estimated transport map Tt to obtain the cell-to-cell coupling and record one aligned lineage assignment per source cell. The rationale is that high-dimensional, noisy, large-sample expression data render classical alignment measures such as Euclidean distance or Pearson correlation invalid, whereas the estimated empirical transportation cost selects the pairs with smallest cost. The resulting coupling feeds downstream trajectory_expression() and smooth_cell_expression() steps of the CellST pipeline.

## Related tools

- **CellST** (Provides cell_name_traj(), the cell lineage tracing function that aligns cells between adjacent time points via optimal transport) — github.com/zhanzmr/CellST
- **pot** (Python optimal transport package required in the 'ot' conda environment; supplies the Earth mover's distance solver behind the alignment)
- **readr** (R package used to load the CSV expression matrix (read_csv) before calling cell_name_traj())
- **devtools** (Used to install CellST from GitHub with install_github())

## Examples

```
library(readr); test_data <- data.frame(read_csv("test_data.csv"), row.names = 1); cell_traj <- cell_name_traj(test_data)
```

## Evaluation signals

- Each source cell at time t receives exactly one unique aligned target cell at t+1 (one-to-one permutation-style coupling)
- On simulated two-time-point data with known ground-truth correspondences (200–600 cells, 100–500 genes), CellST's coupling attains higher aligning accuracy than Euclidean-distance and Pearson-correlation baselines
- On the mouse hematopoietic dataset, pseudo-time changes of connected cells concentrate near zero and CellST's pseudo-time similarity is at least as high as Pearson's correlation baseline (Fig. 5b and c)
- Output file (e.g. cell_coupling.csv) exists, is parseable, and contains one aligned lineage assignment per source cell
- Environment check: the 'ot' conda environment (python=3.9) with 'pot' installed resolves the transport problem without error

## Limitations

- Requires a conda environment with the 'pot' package installed before the function can be used
- Cells within the same time point may be at different developmental stages, which motivated the method but also means alignment quality depends on smoothing downstream to reduce cell-cell variation
- Classical similarity measures (Euclidean distance, Pearson correlation) are invalid at this scale, so alternative ad-hoc couplings should not be substituted casually
- No quantitative performance metrics or benchmark values are provided in the article's discussion to support the claimed high accuracy beyond the reported comparisons

## Evidence
