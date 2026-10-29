---
name: empirical-distribution-estimation-scrna-seq
description: Use when time-course scRNA-seq data sacrifice cells at each time point
  (no cell-to-cell correspondence) and you need to set up the discrete OT problem
  for CellST's cell_name_traj lineage alignment.
when_to_use_negative:
- When paired correspondence information already exists between the two time points
  (e.g. longitudinal sampling of the same cells) — the Dirac-mixture OT formulation
  is designed for sacrificed-cell designs with no correspondence
- When the goal is a bulk population trajectory rather than individual cell-to-cell
  lineages — CellST explicitly addresses the case where cells within the same time
  point are at different developmental stages
- When the input is not normalized gene expression per cell (e.g. raw counts with
  unnormalized library sizes), since the squared Euclidean cost assumption requires
  comparable expression scales
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CellST
  role: Framework whose cell_name_traj() uses the estimated empirical distributions
    to align cells between adjacent time points via optimal transport (Earth mover's
    distance)
  repo: github.com/zhanzmr/CellST
- name: pot
  role: Python optimal-transport library used inside the 'ot' conda environment to
    solve the discrete Monge OT problem over the estimated Dirac-mixture distributions
- name: readr
  role: R package used to load test_data.csv into a data.frame of cells x genes from
    which the empirical distributions are built
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ailsci.2023.100068
    title: Elucidating dynamic cell lineages and gene networks in time-course single
      cell differentiation
  source_content_hash: sha256:57f98d939bbe9b6f1ddbd2cfdf689d618c00d96a9c4b1a4af1ae74b01db109f0
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
  - pot
  - readr
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# empirical-distribution-estimation-scrna-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates empirical gene feature distributions (μt, μt+1) as mixtures of Dirac measures from normalized scRNA-seq expression matrices so that a discrete optimal transport map can align cells between adjacent time points.

## When to use

Use when time-course scRNA-seq data sacrifice cells at each time point (no cell-to-cell correspondence) and you need to set up the discrete OT problem for CellST's cell_name_traj lineage alignment.

## When NOT to use

- When paired correspondence information already exists between the two time points (e.g. longitudinal sampling of the same cells) — the Dirac-mixture OT formulation is designed for sacrificed-cell designs with no correspondence
- When the goal is a bulk population trajectory rather than individual cell-to-cell lineages — CellST explicitly addresses the case where cells within the same time point are at different developmental stages
- When the input is not normalized gene expression per cell (e.g. raw counts with unnormalized library sizes), since the squared Euclidean cost assumption requires comparable expression scales

## Inputs

- Normalized gene expression matrix for time point t (CSV, e.g. test_data.csv), cells as rows, genes as d-dimensional expression vectors
- Normalized gene expression matrix for adjacent time point t+1 (CSV)
- conda environment 'ot' with Python 3.9 and the 'pot' package installed

## Outputs

- Empirical distributions μt and μt+1 represented as mixtures of Dirac measures (one per cell)
- Estimated transport map Tt giving the cell-to-cell coupling from Xt to Xt+1
- Aligned lineage assignment per source cell (cell_coupling.csv coupling matrix)

## How to apply

Load the normalized gene expression matrices for the two adjacent time points from test_data.csv with readr (R) or pandas (Python), treating each cell as a d-dimensional gene-expression vector to form the empirical point sets Xt and Xt+1. Formulate the empirical distributions μt and μt+1 as mixtures of Dirac measures, one Dirac delta per cell, as specified in CellST's methods for the discrete optimal transport problem. Set up the conda environment with python=3.9 and the 'pot' package, since CellST delegates the transport solve to POT's Sinkhorn-type solver on these estimated distributions. Choose the squared Euclidean cost (alpha=2) as the transportation cost; with equal-sized time points the discrete Monge problem reduces to a one-to-one assignment represented by a permutation matrix. Apply the resulting transport map Tt to obtain the cell-to-cell coupling, and validate it against known coupled cell pairs in simulated two-time-point data before saving the coupling matrix to cell_coupling.csv.

## Related tools

- **CellST** (Framework whose cell_name_traj() uses the estimated empirical distributions to align cells between adjacent time points via optimal transport (Earth mover's distance)) — github.com/zhanzmr/CellST
- **pot** (Python optimal-transport library used inside the 'ot' conda environment to solve the discrete Monge OT problem over the estimated Dirac-mixture distributions)
- **readr** (R package used to load test_data.csv into a data.frame of cells x genes from which the empirical distributions are built)

## Examples

```
conda create -n ot python=3.9; pip install pot
```

## Evaluation signals

- The conda environment 'ot' (python=3.9) is created and 'pot' is importable before running cell_name_traj, per the CellST methods
- The coupling matrix (cell_coupling.csv) exists after the run and contains one aligned lineage assignment per source cell
- For equal-sized time points, the estimated transport plan is a permutation matrix (one-to-one cell assignment), not a partial or fractional coupling
- Predicted cell-to-cell coupling agrees with known coupled cell pairs in the simulated two-time-point validation data
- Each cell is represented as exactly one Dirac delta in μt (and μt+1), so the discrete OT problem dimension equals the per-time-point cell count

## Limitations

- Classical distances such as Euclidean or Pearson correlation are invalid for large-sample, high-dimensional, noisy scRNA-seq data, which motivates the OT formulation but makes the squared Euclidean cost sensitive to normalization choices
- No quantitative performance metrics or benchmark values are provided in the paper to verify claimed alignment accuracy
- No CellST software version or dependency version information is stated beyond the conda environment setup

## Evidence

- [readme] test_data <- read_csv("test_data.csv")
test_data <- data.frame(test_data, row.names = 1): "test_data <- read_csv("test_data.csv")"
