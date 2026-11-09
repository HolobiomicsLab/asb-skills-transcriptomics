---
name: bayesian-confidence-interval-inference
description: Use when smoothed individual cell fate trajectories from time-course
  scRNA-seq are available and the goal is a time-resolved gene-gene network plus identification
  of critical (differentially expressed) genes during differentiation.
when_to_use_negative:
- Input is raw time-course scRNA-seq without prior cell-to-cell lineage alignment
  (cell_name_traj) and smoothing (smooth_cell_expression) — the functional concurrent
  model operates on smoothed individual trajectories, not bulk trajectories.
- The goal is a single bulk population trajectory rather than dynamic pairwise gene-gene
  relationships at the individual cell level; CellST is designed for the individual-cell
  case.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellST
  role: R framework whose functional concurrent model + Bayesian confidence interval
    machinery builds the dynamic gene network from smoothed cell fate trajectories
  repo: github.com/zhanzmr/CellST
- name: R
  role: language environment used to install and run CellST and its dependencies
- name: CombMSC 1.4.2.1
  role: required archived CRAN dependency that must be installed before CellST
  repo: https://cran.r-project.org/src/contrib/Archive/CombMSC/
- name: readr
  role: used to read CSV inputs (e.g., test_data.csv) in the CellST workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ailsci.2023.100068
    title: Elucidating dynamic cell lineages and gene networks in time-course single
      cell differentiation
  source_content_hash: sha256:ed51f1b783dea23e0792543cac1b75fdce92b8b8c941caa86712c5def0963583
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
  - readr
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# bayesian-confidence-interval-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs dynamic gene-gene networks from individual cell fate trajectories by fitting functional concurrent linear models and keeping edges whose time-varying coefficient beta(l,s)(t) has a Bayesian confidence interval excluding zero, as implemented in CellST.

## When to use

Use when smoothed individual cell fate trajectories from time-course scRNA-seq are available and the goal is a time-resolved gene-gene network plus identification of critical (differentially expressed) genes during differentiation.

## When NOT to use

- Input is raw time-course scRNA-seq without prior cell-to-cell lineage alignment (cell_name_traj) and smoothing (smooth_cell_expression) — the functional concurrent model operates on smoothed individual trajectories, not bulk trajectories.
- The goal is a single bulk population trajectory rather than dynamic pairwise gene-gene relationships at the individual cell level; CellST is designed for the individual-cell case.

## Inputs

- smoothed individual cell fate trajectories from CellST smooth_cell_expression() (SSANOVA-based smoothing spline output)
- gene expression across cell lineage trajectories from trajectory_expression() / gene_expression()

## Outputs

- time-resolved weighted adjacency table (dynamic_gene_network.csv) with one row per gene pair, the coefficient beta(l,s)(t), its Bayesian confidence band, and the BY-corrected FMANOVA p-value
- dynamic gene network annotation with critical genes (differentially expressed genes)

## How to apply

Load the smoothed individual cell fate trajectories from the CellST smoothing spline step (SSANOVA-based smooth_cell_expression output). For each gene pair (l,s), fit the functional concurrent linear model in a reproducing kernel Hilbert space by minimizing the penalized least squares criterion (eq. 12), choosing the smoothing parameter lambda via Generalized Cross-Validation (GCV). Compute the Bayesian posterior variance of beta(l,s)(t) (eq. 16) and form the 100(1-alpha)% Bayesian confidence band BCI(l,s) = beta_hat(l,s)(t) +/- z_(alpha/2) * gamma(l,s)(t). A node is a gene; an edge exists for pairs whose non-zero coefficient has a confidence interval excluding zero at each time point. Additionally run the functional ANOVA (FMANOVA) test with Wilks lambda statistics (approximated by the F(l-1)k,(n-l)k distribution, kappa estimated by naive and bias-reduced methods) to flag differentially expressed genes as critical genes, correcting p-values with the Benjamini & Yekutieli method, and write a time-resolved weighted adjacency table with critical-gene annotations.

## Related tools

- **CellST** (R framework whose functional concurrent model + Bayesian confidence interval machinery builds the dynamic gene network from smoothed cell fate trajectories) — github.com/zhanzmr/CellST
- **R** (language environment used to install and run CellST and its dependencies)
- **CombMSC 1.4.2.1** (required archived CRAN dependency that must be installed before CellST) — https://cran.r-project.org/src/contrib/Archive/CombMSC/
- **readr** (used to read CSV inputs (e.g., test_data.csv) in the CellST workflow)

## Examples

```
library(devtools); install_github("zhanzmr/CellST")
```

## Evaluation signals

- dynamic_gene_network.csv exists with one row per gene pair
- output columns include the time-resolved coefficient beta(l,s)(t), its Bayesian confidence band, and the BY-corrected FMANOVA p-value
- edges in the network correspond exactly to gene pairs whose beta(l,s)(t) confidence interval excludes zero at each time point
- the script runs end-to-end on the smoothed trajectories input (SSANOVA smooth_cell_expression output)
- smoothing parameter lambda for the penalized least squares fit was selected by GCV, not fixed arbitrarily

## Limitations

- No software version or dependency information for CellST is provided in the article
- CombMSC is archived on CRAN and must be downloaded manually from the CRAN Archive and installed from the tarball
- FMANOVA critical-gene calls depend on the kappa approximations (naive and bias-reduced); no quantitative performance metrics for the constructed networks are provided in the discussion

## Evidence

- [methods] We use Bayesian confidence intervals to construct the dynamic graph, where a node represents a gene, and an edge between two nodes exists if the two corresponding genes follow the model (11): "We use Bayesian confidence intervals to construct the dynamic graph, where a node represents a gene, and an edge between two nodes exists if the two corresponding genes follow the model (11)"
