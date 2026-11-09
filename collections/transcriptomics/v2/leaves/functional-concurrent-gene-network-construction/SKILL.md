---
name: functional-concurrent-gene-network-construction
description: Use when time-course scRNA-seq data have been aligned into individual
  cell fate trajectories (e.g., via CellST smoothing) and the goal is to infer dynamic
  gene-gene regulatory relationships and critical genes along differentiation.
when_to_use_negative:
- Data are a single time point or lack cell-to-cell correspondence between time points,
  so no aligned individual cell fate trajectories exist to fit the functional concurrent
  models
- The goal is a single bulk pseudotime trajectory rather than dynamic pairwise gene-gene
  networks at the individual cell level
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellST
  role: Provides the smoothed individual cell fate trajectories and the functional
    concurrent linear model framework (penalized least squares with representer theorem)
    used to build the dynamic gene networks and identify critical genes
  repo: https://github.com/zhangzmr/CellST
- name: R
  role: Environment for running CellST, installing archived CombMSC_1.4.2.1 via R
    CMD INSTALL, and loading data with readr
- name: readr
  role: Loads the expression matrix (e.g. read_csv of test_data.csv) that feeds trajectory
    extraction prior to network construction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ailsci.2023.100068
    title: Elucidating dynamic cell lineages and gene networks in time-course single
      cell differentiation
  source_content_hash: sha256:a313e444eaec450f4c9284790171d69e7deb4c760835dfdb204fb6a6888be743
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zhangzmr/CellST
  repo_url: https://github.com/zhangzmr/CellST
  tools:
  - CellST
  - CombMSC
  - R
  - devtools
  - pot
  - readr
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# functional-concurrent-gene-network-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs dynamic gene networks from individual cell fate trajectories by fitting functional concurrent linear models between gene pairs, using Bayesian confidence intervals on the dynamic coefficient beta(l,s)(t) to define edges, and thereby identifies critical genes regulating differentiation (e.g., MYF5, MYOD1, KRT18 in zebrafish embryogenesis).

## When to use

Use when time-course scRNA-seq data have been aligned into individual cell fate trajectories (e.g., via CellST smoothing) and the goal is to infer dynamic gene-gene regulatory relationships and critical genes along differentiation.

## When NOT to use

- Data are a single time point or lack cell-to-cell correspondence between time points, so no aligned individual cell fate trajectories exist to fit the functional concurrent models
- The goal is a single bulk pseudotime trajectory rather than dynamic pairwise gene-gene networks at the individual cell level

## Inputs

- Smoothed individual cell fate trajectories from CellST smooth_cell_expression() (SSANOVA smoothing spline output, e.g. npred=300)
- Per-trajectory gene expression from trajectory_expression() applied to a time-course scRNA-seq matrix (e.g. 38,731 cells x 11,588 genes zebrafish Drop-seq CSV via readr)
- Individual gene expression across trajectories from gene_expression()

## Outputs

- Dynamic gene network where nodes are genes and edges are gene pairs with non-zero Bayesian confidence intervals for beta(l,s)(t)
- Critical gene table (e.g. markers of Ccr7+ migratory DC-like populations; MYF5, MYOD1, KRT18)
- Dynamic gene-gene relationship estimates along the differentiation timeline

## How to apply

First obtain smoothed individual cell fate trajectories, e.g. by running cell_name_traj() (optimal transport via Earth mover's distance on the 38,731 cells x 11,588 genes zebrafish Drop-seq matrix), trajectory_expression(), and smooth_cell_expression(npred=300). Then fit functional concurrent linear models between pairwise gene expression patterns along the trajectories via penalized least squares with the representer theorem. Define a network edge between two genes when the Bayesian confidence interval for the dynamic coefficient beta(l,s)(t) is non-zero; a node is a gene, and an edge exists if the gene pair follows the functional concurrent model (11). The resulting dynamic gene network / critical gene table can be tabulated to nominate markers such as Ccr7+ migratory DC-like population genes, and expression patterns (MYF5, MYOD1 in trajectory group 5; KRT18 in group 15) can be cross-checked as validation.

## Related tools

- **CellST** (Provides the smoothed individual cell fate trajectories and the functional concurrent linear model framework (penalized least squares with representer theorem) used to build the dynamic gene networks and identify critical genes) — https://github.com/zhangzmr/CellST
- **R** (Environment for running CellST, installing archived CombMSC_1.4.2.1 via R CMD INSTALL, and loading data with readr)
- **readr** (Loads the expression matrix (e.g. read_csv of test_data.csv) that feeds trajectory extraction prior to network construction)

## Examples

```
total_pred <- smooth_cell_expression(traj_express, npred=300)
```

## Evaluation signals

- Verify that reported network edges correspond exactly to gene pairs whose Bayesian confidence intervals for the dynamic coefficient beta(l,s)(t) are non-zero
- Confirm a dynamic gene network / critical gene table file is produced covering genes across the Drop-seq time course
- Cross-check nominated critical genes against trajectory-group expression: MYF5 and MYOD1 significantly higher in trajectory group 5 and KRT18 highly expressed in group 15
- Confirm the upstream trajectory files contain unique trajectories for individual cells (not a single bulk trajectory) before network fitting

## Limitations

- No quantitative performance metrics or benchmark values are provided to support the claimed high accuracy of the constructed lineages/networks
- No software version or dependency information for CellST is provided; CombMSC must be installed from the CRAN archive because it is archived
- Cell alignment relies on optimal transport between adjacent time points since time-course scRNA-seq sacrifices cells, leaving no true cell-to-cell correspondence

## Evidence
