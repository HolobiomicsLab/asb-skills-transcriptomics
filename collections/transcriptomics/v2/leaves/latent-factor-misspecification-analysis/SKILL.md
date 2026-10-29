---
name: latent-factor-misspecification-analysis
description: Use when assessing how sensitive a ZINB-WaVE fit is to choosing the wrong
  number of latent factors K or omitting the gene-level intercept in V, e.g. before
  trusting low-dimensional outputs from scRNA-seq counts.
when_to_use_negative:
- The goal is comparing dimensionality-reduction methods (e.g. ZINB-WaVE vs PCA vs
  ZIFA) on real data rather than auditing estimator bias under model misspecification
- The data are already normalized/log-transformed and you do not intend to model raw
  counts with the ZINB likelihood — the skill's simulation refits require count data
  and genewise dispersion
- You lack the fitted-model parameters (true ln(mu), pi) needed to define bias and
  MSE, e.g. when only raw unmodelled data are available
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3572
tools:
- name: zinbwave
  role: Fits the zero-inflated negative binomial model (zinbFit) and simulates replicate
    count datasets from a fitted zinbModel (zinbSim) under misspecified K
  repo: https://bioconductor.org/packages/zinbwave
- name: ZINB-WaVE
  role: The zero-inflated negative binomial modeling framework whose estimator robustness
    to K, V, and dispersion misspecification is being evaluated
  repo: https://github.com/drisso/zinbwave
- name: scRNAseq
  role: Provides the S1/CA1 single-cell RNA-seq count matrix used as the seed dataset
    for fitting and simulation
- name: R (>=3.4)
  role: Runtime environment required for the zinbwave package and the simulation/refit
    loop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:d2587dd041e6dfe7e28226089d88122c96b54d82e54b92e2e186e04d21ede879
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - Bioconductor
  - R
  - R (>=3.4)
  - ZINB-WaVE
  - scRNAseq
  - zinbwave
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# latent-factor-misspecification-analysis

## Summary

A simulation-based robustness assessment for ZINB-WaVE that quantifies bias and MSE of ln(mu) and pi estimates when the number of latent factors K, the gene-level covariate matrix V, or the dispersion specification are misspecified. It grounds the paper's claim that ZINB-WaVE yields robust, unbiased estimators of underlying biological signals.

## When to use

Use when assessing how sensitive a ZINB-WaVE fit is to choosing the wrong number of latent factors K or omitting the gene-level intercept in V, e.g. before trusting low-dimensional outputs from scRNA-seq counts.

## When NOT to use

- The goal is comparing dimensionality-reduction methods (e.g. ZINB-WaVE vs PCA vs ZIFA) on real data rather than auditing estimator bias under model misspecification
- The data are already normalized/log-transformed and you do not intend to model raw counts with the ZINB likelihood — the skill's simulation refits require count data and genewise dispersion
- You lack the fitted-model parameters (true ln(mu), pi) needed to define bias and MSE, e.g. when only raw unmodelled data are available

## Inputs

- S1/CA1 single-cell RNA-seq count matrix (via the scRNAseq Bioconductor package)
- Fitted ZINB-WaVE model coefficients (zinbModel object) used as simulation ground truth
- Simulation design parameters: K=2 true latent factors, X = 1n sample-level intercept, V = 1J gene-level intercept, genewise dispersion

## Outputs

- B=10 simulated count matrices (n=1000 cells x J=1000 genes)
- Per-gene bias and MSE tables for ln(mu) and pi under K = 1, 2, 3, 4 with and without the gene-level intercept
- Boxplots of bias and MSE for ln(mu) and pi across K and intercept settings (bias_mse_boxplots.png)

## How to apply

Fit ZINB-WaVE to the S1/CA1 single-cell RNA-seq count matrix (loaded via the scRNAseq Bioconductor package) with K=2 latent factors, a sample-level intercept design X = 1n, a gene-level covariate matrix V containing a column of ones (1J), and genewise dispersion estimation. From the fitted coefficients, build a zinbModel and use zinbSim to simulate B=10 independent count datasets, each with n=1000 cells and J=1000 genes, so that true ln(mu) and pi parameters are known. Refit ZINB-WaVE to each simulated dataset under misspecified K = 1, 2, 3, 4, both with and without the gene-level intercept in V, again using genewise dispersion. For every gene and setting, compute bias and mean squared error (MSE) of the ln(mu) and pi estimates relative to the true simulation parameters, then summarize with boxplots. The decision point is the gene-level intercept: including the column of ones in V makes the cell-level intercept a global-scaling normalization factor and is what confers robustness to K misspecification.

## Related tools

- **zinbwave** (Fits the zero-inflated negative binomial model (zinbFit) and simulates replicate count datasets from a fitted zinbModel (zinbSim) under misspecified K) — https://bioconductor.org/packages/zinbwave
- **ZINB-WaVE** (The zero-inflated negative binomial modeling framework whose estimator robustness to K, V, and dispersion misspecification is being evaluated) — https://github.com/drisso/zinbwave
- **scRNAseq** (Provides the S1/CA1 single-cell RNA-seq count matrix used as the seed dataset for fitting and simulation)
- **R (>=3.4)** (Runtime environment required for the zinbwave package and the simulation/refit loop)

## Examples

```
library(zinbwave); fit <- zinbFit(counts, X = model.matrix(~1, data = colData), V = model.matrix(~1, data = rowData), K = 2); sim <- zinbSim(fit, ncell = 1000, niter = 10)
```

## Evaluation signals

- Boxplots of bias for ln(mu) and pi are centered at or very near zero for misspecified K (K != 2) when the gene-level intercept (V = 1J) is included
- MSE values remain small across K = 1, 3, 4 settings when the gene-level intercept is included, matching the paper's robustness finding
- Simulated datasets match the specified dimensions: 10 replicates of 1000 cells x 1000 genes generated from the K=2 fitted model
- Refits without the gene-level intercept in V show degraded (larger) bias, confirming the intercept's role as global-scaling normalization factor
- All refits use genewise dispersion, consistent with the fitted ground-truth model

## Limitations

- ZINB-WaVE requires numerical optimization of a non-convex likelihood, which incurs a computational cost; computing time is approximately linear in cells and genes and approximately quadratic in the number of latent factors
- Exact reproduction of the paper's simulations requires version 0.1.1 of the zinbwave package (zinbwave@v0.1.1)
- Robustness of ln(mu) and pi estimates does not guarantee the low-dimensional signal W is biologically relevant; unaccounted unwanted technical variation can be captured by W

## Evidence

- [results] Robustness of estimators to misspecification: [evidence span withheld — anchor status: not_found]
- [discussion] Robust and unbiased estimators: "We have shown with simulated and real data analyses that ZINB-WaVE leads to robust and unbiased estimators of the underlying biological signals."
- [discussion] Global-scaling normalization via gene-level intercept: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [methods] Installation requirements: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [readme] Reproduction dependencies: [evidence span withheld — anchor status: external_doc]
