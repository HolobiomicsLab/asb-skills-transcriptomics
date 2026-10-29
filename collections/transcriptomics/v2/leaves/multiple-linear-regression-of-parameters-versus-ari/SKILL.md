---
name: multiple-linear-regression-of-parameters-versus-ari
description: Use when you have ARI results from clustering methods evaluated across
  a SymSim parameter grid (N, Sigma, alpha, Depth, rare-population proportion) and
  need to attribute performance variation to individual simulation factors.
when_to_use_negative:
- The data are real experimental counts rather than SymSim simulations with known
  ground-truth parameters, so there is no parameter grid to regress against
- You only have ARI for a single parameter setting (no variation in N, Sigma, alpha,
  Depth, or Prop), leaving the regression with no explanatory variation
- The goal is to rank clustering methods rather than attribute ARI variation to simulation
  factors — a direct comparison of ARI curves (Fig. 5b-c) is more appropriate
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: Generates the simulated UMI datasets (SimulateTrueCounts with evf_type='discrete'
    and Phyla5 tree, then True2ObservedCounts with protocol='UMI') across the parameter
    grid whose ARI values feed the regression
  repo: https://github.com/YosefLab/SymSim
- name: RJAGS
  role: R Gibbs-sampler package used in the paper's R-based statistical workflows
    for parameter estimation; regression and coefficient extraction are done in the
    same R environment
- name: ggplot2
  role: Plots the standardized regression coefficients as the Fig. 5a bar plot
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:a493bb84d48a91de48b49707f6436b15b90cd5a2cd67bc9d3b052b7143b34a83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - RJAGS
  - SymSim
  - ggplot2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# Multiple linear regression of simulation parameters versus ARI

## Summary

Fits a multiple linear regression of clustering adjusted Rand index (ARI) on the SymSim simulation parameters (N, Sigma, alpha, Depth, Prop) and extracts standardized coefficients to quantify which factors drive rare-population detection performance, reproducing the Fig. 5a coefficient bar plot of the Zhang et al. 2019 SymSim paper.

## When to use

Use when you have ARI results from clustering methods evaluated across a SymSim parameter grid (N, Sigma, alpha, Depth, rare-population proportion) and need to attribute performance variation to individual simulation factors.

## When NOT to use

- The data are real experimental counts rather than SymSim simulations with known ground-truth parameters, so there is no parameter grid to regress against
- You only have ARI for a single parameter setting (no variation in N, Sigma, alpha, Depth, or Prop), leaving the regression with no explanatory variation
- The goal is to rank clustering methods rather than attribute ARI variation to simulation factors — a direct comparison of ARI curves (Fig. 5b-c) is more appropriate

## Inputs

- ARI results table (one row per SymSim grid combination x clustering method) from cal_ARI computed on simulated UMI datasets
- Simulation parameter grid values: N (total cells), Sigma (within-population variability, e.g. 0.4), alpha (alpha_mean/alpha_sd), depth (depth_mean/depth_sd), and rare-population proportion (Prop, e.g. 5% or 10%) from Supplementary Table 6

## Outputs

- Regression coefficients (standardized) for each simulation parameter per clustering method
- Fig. 5a-style bar plot of standardized regression coefficients
- Fitted model object with effect sizes and significance of each parameter on ARI

## How to apply

First assemble the ARI results table with one row per grid combination and clustering method, where each row carries the simulation parameters used by SymSim: number of cells (N), within-population variability (Sigma, e.g. 0.4), mRNA capture efficiency (alpha_mean/alpha_sd), sequencing depth (depth_mean/depth_sd), and rare-population proportion (Prop). Fit a multiple linear regression with ARI as the response and N, Sigma, alpha, Depth, and Prop as predictors (R formula: ARI ~ N + Sigma + alpha + Depth + Prop), optionally per clustering method (scVI+k-means, PCA+k-means, Seurat, SIMLR). Extract the standardized coefficients so effect sizes are comparable across predictors with different scales, and visualize them as a bar plot to reproduce Fig. 5a. Interpret signs against the paper's reported trends: ARI increases with number of cells and decreases with smaller rare-population proportion. The regression is run in R alongside the ggplot2 plotting and can be coupled to RJAGS-based workflows used elsewhere in the paper's parameter estimation.

## Related tools

- **SymSim** (Generates the simulated UMI datasets (SimulateTrueCounts with evf_type='discrete' and Phyla5 tree, then True2ObservedCounts with protocol='UMI') across the parameter grid whose ARI values feed the regression) — https://github.com/YosefLab/SymSim
- **RJAGS** (R Gibbs-sampler package used in the paper's R-based statistical workflows for parameter estimation; regression and coefficient extraction are done in the same R environment)
- **ggplot2** (Plots the standardized regression coefficients as the Fig. 5a bar plot)

## Examples

```
fit <- lm(ARI ~ N + Sigma + alpha + Depth + Prop, data=ari_results); coef_df <- data.frame(param=names(coef(fit)), beta=coef(fit)); ggplot(coef_df, aes(param, beta)) + geom_col()
```

## Evaluation signals

- The ARI input table contains exactly one row per grid combination and clustering method before fitting
- Standardized coefficients are scale-free and comparable across predictors; N should show a positive coefficient and rare-population proportion (Prop) effects should match the reported direction (ARI decreases with smaller Prop)
- The resulting coefficient bar plot qualitatively matches the trends reported in Fig. 5a (e.g., number of cells positively associated with ARI)
- Residual diagnostics / model fit confirm a linear relationship is tenable; coefficients are stable across clustering methods where the paper reports consistent method rankings (scVI+k-means highest ARI in most settings)

## Limitations

- Conclusions are conditional on the SymSim simulation model, which omits several steps including reverse transcription and library cleanup, so coefficient magnitudes may not transfer directly to real data
- The parameter grid must span the Supplementary Table 6 ranges (Sigma around 0.4, varied alpha and depth, Prop at 5% and 10%) for the regression to identify effects; sparse grids give unstable coefficients
- Linear terms may miss interactions (e.g., Prop x N effects on rare-population detection that Fig. 5b-c curves suggest)

## Evidence

- [other] Fit a multiple linear regression of ARI on the simulation parameters (N, Sigma, alpha, Depth, Prop) and extract the standardized coefficients to reproduce the Fig. 5a bar plot: [evidence span withheld — anchor status: not_found]
- [discussion] We apply three different dimensionality reduction methods to cluster cells simulated from multiple discrete populations: "We apply three different dimensionality reduction methods to cluster cells simulated from multiple discrete populations: PCA, scVI, and SIMLR."
- [abstract] SymSim can be used to evaluate the number of cells required to detect a rare population under various scenarios: "We also show how SymSim can be used to evaluate the number of cells required to detect a rare population under various scenarios."
- [methods] SymSimTrueCounts and True2ObservedCounts are the two major functions: "**SimulateTrueCounts( )** and **True2ObservedCounts( )** are the two major functions to generate datasets."
- [readme] SymSim install instruction from README: "devtools::install_github("YosefLab/SymSim")"
