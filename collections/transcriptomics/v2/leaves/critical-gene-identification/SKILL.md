---
name: critical-gene-identification
description: Use when analyzing time-course scRNA-seq data (e.g. zebrafish embryogenesis
  Drop-seq with 38,731 cells x 11,588 genes) to discover genes that dynamically regulate
  cells into mature cell types via gene-gene network edges.
when_to_use_negative:
- Data are single-time-point scRNA-seq with no cell-to-cell correspondence between
  time points, since the functional concurrent models require trajectories across
  adjacent time points
- The goal is a bulk population pseudotime trajectory rather than individual cell
  fate trajectories and gene-gene dynamic relationships
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellST
  role: R/Python framework providing cell_name_traj(), trajectory_expression(), smooth_cell_expression()
    and the functional concurrent model machinery used to build dynamic gene networks
    and extract critical genes
  repo: github.com/zhanzmr/CellST
- name: pot
  role: Python optimal-transport package used by cell_name_traj() to align cells between
    adjacent time points via Earth mover's distance
- name: CombMSC
  role: Archived CRAN dependency (CombMSC_1.4.2.1.tar.gz) required by CellST, installed
    via R CMD INSTALL
  repo: https://cran.r-project.org/src/contrib/Archive/CombMSC/
- name: readr
  role: R package used to load the expression matrix CSV before trajectory and network
    construction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ailsci.2023.100068
    title: Elucidating dynamic cell lineages and gene networks in time-course single
      cell differentiation
  source_content_hash: sha256:507909a5a70c32f81d8c488e842c11dbe6361c5708f9fc6535be505afc92c5da
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
  - pot
  - readr
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# critical-gene-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies critical genes regulating cell differentiation by constructing dynamic gene networks from individual cell fate trajectories produced by CellST, using functional concurrent linear models with Bayesian confidence intervals on time-course scRNA-seq data (e.g. zebrafish embryogenesis Drop-seq).

## When to use

Use when analyzing time-course scRNA-seq data (e.g. zebrafish embryogenesis Drop-seq with 38,731 cells x 11,588 genes) to discover genes that dynamically regulate cells into mature cell types via gene-gene network edges.

## When NOT to use

- Data are single-time-point scRNA-seq with no cell-to-cell correspondence between time points, since the functional concurrent models require trajectories across adjacent time points
- The goal is a bulk population pseudotime trajectory rather than individual cell fate trajectories and gene-gene dynamic relationships

## Inputs

- Time-course scRNA-seq expression matrix (e.g. zebrafish embryogenesis Drop-seq: 38,731 cells x 11,588 genes, CSV readable via readr::read_csv)
- Smoothed individual cell fate trajectories from CellST smooth_cell_expression()
- Per-trajectory gene expression from trajectory_expression() and per-gene expression from gene_expression()

## Outputs

- Dynamic gene network where nodes are genes and edges are gene pairs with non-zero beta(l,s)(t) Bayesian confidence intervals
- Table of critical genes potentially regulating cells into mature cell types (e.g. Ccr7+ migratory DC-like markers)

## How to apply

After CellST aligns cells between adjacent time points via optimal transport (Earth mover's distance) with cell_name_traj() and smooths individual cell fate trajectories with smooth_cell_expression(npred=300) using SSANOVA smoothing splines with the smoothing parameter chosen by Generalized Cross-Validation, construct dynamic gene networks by fitting functional concurrent linear models between gene pairs via penalized least squares with the representer theorem. Define a network edge between two genes when the dynamic coefficient beta(l,s)(t) has a Bayesian confidence interval that is non-zero; genes anchoring many such edges are tabulated as critical genes. Optionally normalize the developmental timeline across time points with align_cell_time() before network construction, and validate candidate critical genes by checking differential expression across trajectory groups (e.g. MYF5 and MYOD1 higher in trajectory group 5, KRT18 high in group 15 for Ccr7+ migratory DC-like populations).

## Related tools

- **CellST** (R/Python framework providing cell_name_traj(), trajectory_expression(), smooth_cell_expression() and the functional concurrent model machinery used to build dynamic gene networks and extract critical genes) — github.com/zhanzmr/CellST
- **pot** (Python optimal-transport package used by cell_name_traj() to align cells between adjacent time points via Earth mover's distance)
- **CombMSC** (Archived CRAN dependency (CombMSC_1.4.2.1.tar.gz) required by CellST, installed via R CMD INSTALL) — https://cran.r-project.org/src/contrib/Archive/CombMSC/
- **readr** (R package used to load the expression matrix CSV before trajectory and network construction)

## Examples

```
library(devtools); install_github("zhanzmr/CellST"); test_data <- data.frame(readr::read_csv("test_data.csv"), row.names=1); cell_traj <- cell_name_traj(test_data); traj_express <- trajectory_expression(test_data, cell_traj); total_pred <- smooth_cell_expression(traj_express, npred=300)
```

## Evaluation signals

- Dynamic gene network is produced in which every reported edge corresponds to a gene pair whose dynamic coefficient beta(l,s)(t) has a non-zero Bayesian confidence interval
- Critical gene table is generated and contains known markers of the target mature cell type (e.g. Ccr7+ migratory DC-like population markers)
- Downstream FMANOVA/differential expression checks agree with network calls: MYF5 and MYOD1 significantly higher in trajectory group 5, KRT18 highly expressed in group 15
- Upstream trajectory files exist with unique trajectories for individual cells across the Drop-seq time course and smoothed output uses npred=300

## Limitations

- No quantitative performance metrics or benchmark values are provided to support the claimed high accuracy of lineage and network construction
- No software version or dependency information for CellST is provided; the required CombMSC package is archived on CRAN and must be installed manually
- Time-course scRNA-seq sacrifices cells at each time point, so there is no true cell-to-cell correspondence between time points; alignment relies on optimal transport on high-dimensional, noisy expression data where Euclidean distance or Pearson correlation are invalid

## Evidence
