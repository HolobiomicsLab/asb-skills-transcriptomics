---
name: functional-concurrent-linear-model-estimation
description: Use when you have smoothed individual cell fate trajectories from time-course
  scRNA-seq and need to identify dynamic pairwise gene-gene relationships and critical
  genes across differentiation time.
when_to_use_negative:
- Input is raw or unsmoothed scRNA-seq expression — this skill expects smoothed individual
  cell fate trajectories from the CellST smoothing spline step, not bulk or raw count
  matrices
- You need a static (single-time-point) gene co-expression network rather than time-resolved
  dynamic relationships along differentiation
- Your data are bulk cell trajectories (population mean) rather than individual cell
  fate trajectories — the article's motivation is that bulk trajectories miss individual
  differentiation behaviors
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellST
  role: Framework providing smooth_cell_expression() trajectories and the dynamic
    gene network construction via functional concurrent models; installed via devtools
    install_github
  repo: github.com/zhangzmr/CellST
- name: R
  role: Execution environment for the functional concurrent model fitting, Bayesian
    confidence interval computation, and FMANOVA testing
- name: CombMSC 1.4.2.1
  role: Required R dependency (archived on CRAN) installed from the CRAN archive tarball
  repo: https://cran.r-project.org/src/contrib/Archive/CombMSC/
- name: readr
  role: Used in R to read input CSV expression data and write the adjacency output
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ailsci.2023.100068
    title: Elucidating dynamic cell lineages and gene networks in time-course single
      cell differentiation
  source_content_hash: sha256:4740bb56f9cf14e7b64e22496fb4c825499ae717d6ce4266dbfc5c9ac29b7f3d
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
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# functional-concurrent-linear-model-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates time-resolved pairwise gene-gene relationships from individual cell fate trajectories by fitting a functional concurrent linear model in a reproducing kernel Hilbert space, then builds a dynamic gene network using Bayesian confidence intervals and flags critical genes via FMANOVA with Benjamini & Yekutieli correction.

## When to use

Use when you have smoothed individual cell fate trajectories from time-course scRNA-seq and need to identify dynamic pairwise gene-gene relationships and critical genes across differentiation time.

## When NOT to use

- Input is raw or unsmoothed scRNA-seq expression — this skill expects smoothed individual cell fate trajectories from the CellST smoothing spline step, not bulk or raw count matrices
- You need a static (single-time-point) gene co-expression network rather than time-resolved dynamic relationships along differentiation
- Your data are bulk cell trajectories (population mean) rather than individual cell fate trajectories — the article's motivation is that bulk trajectories miss individual differentiation behaviors

## Inputs

- Smoothed individual cell fate trajectories (output of CellST smooth_cell_expression(), SSANOVA-based)
- Time-course scRNA-seq gene expression matrix for each cell lineage trajectory

## Outputs

- dynamic_gene_network.csv: time-resolved weighted adjacency table with one row per gene pair, columns for time-resolved coefficient beta(l,s)(t), its Bayesian confidence band, and BY-corrected FMANOVA p-value
- Critical-gene annotations (differentially expressed genes flagged by FMANOVA)

## How to apply

Load the smoothed individual cell fate trajectories from CellST's smooth_cell_expression() (SSANOVA-based) output. For each gene pair (l,s), fit the functional concurrent linear model by minimizing the penalized least squares criterion (eq. 12) in a reproducing kernel Hilbert space, selecting the smoothing parameter lambda by Generalized Cross-Validation (GCV). Compute the Bayesian posterior variance of beta(l,s)(t) (eq. 16) to derive the 100(1-alpha)% Bayesian confidence band BCI(l,s) = beta_hat(l,s)(t) +/- z_(alpha/2) * gamma(l,s)(t); an edge exists at each time point when this interval excludes zero. Run the functional ANOVA (FMANOVA) test with Wilks lambda statistics, approximated by the F(l-1)k,(n-l)k distribution with kappa estimated by both naive and bias-reduced methods, to flag differentially expressed genes as critical genes. Correct all gene p-values with the Benjamini & Yekutieli method and write the time-resolved weighted adjacency table with critical-gene annotations.

## Related tools

- **CellST** (Framework providing smooth_cell_expression() trajectories and the dynamic gene network construction via functional concurrent models; installed via devtools install_github) — github.com/zhangzmr/CellST
- **R** (Execution environment for the functional concurrent model fitting, Bayesian confidence interval computation, and FMANOVA testing)
- **CombMSC 1.4.2.1** (Required R dependency (archived on CRAN) installed from the CRAN archive tarball) — https://cran.r-project.org/src/contrib/Archive/CombMSC/
- **readr** (Used in R to read input CSV expression data and write the adjacency output)

## Examples

```
library(readr); traj_express <- trajectory_expression(test_data, cell_traj); total_pred <- smooth_cell_expression(traj_express, npred=300)  # then fit the functional concurrent model per gene pair
```

## Evaluation signals

- dynamic_gene_network.csv exists with one row per gene pair and columns for the time-resolved coefficient beta(l,s)(t), its Bayesian confidence band, and the BY-corrected FMANOVA p-value
- For each reported edge, the non-zero coefficient beta(l,s)(t) has a Bayesian confidence interval excluding zero at the corresponding time point(s)
- Smoothing parameter lambda for each gene-pair fit was chosen by Generalized Cross-Validation (GCV) and converges without warnings
- FMANOVA kappa is estimable by both the naive and bias-reduced methods, and p-values are BY-corrected (Benjamini & Yekutieli)
- The script runs end-to-end on the smoothed trajectories input without errors

## Limitations

- No software version or dependency information for CellST is provided beyond the GitHub install and the archived CombMSC requirement
- The method depends on upstream cell lineage alignment (optimal transport) and smoothing spline quality; the article notes cells within the same time point may be at different developmental stages, complicating interpretation
- No quantitative performance metrics or benchmark values are provided in the discussion to support claimed accuracy of the constructed networks

## Evidence
