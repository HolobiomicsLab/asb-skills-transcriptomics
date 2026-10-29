---
name: estimator-bias-mse-visualization
description: Use when you need to assess robustness of ZINB-WaVE estimates of ln(mu)
  and pi to misspecification of the number of latent covariates K or the gene-level
  covariate matrix V, by comparing refits to simulated ground truth.
when_to_use_negative:
- Input is real observational data without known true ln(mu) and pi parameters — bias
  and MSE cannot be computed against simulation truth
- The goal is low-dimensional representation/t-SNE visualization for downstream clustering
  (ZINB-WaVE's primary use), not estimator validation via refitting
- You are evaluating PCA or ZIFA embeddings, whose parameters (unlike ZINB-WaVE's
  ln(mu) and pi) are not directly comparable to a ZINB simulation truth
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: zinbwave
  role: Fits the ZINB model (zinbFit/zinbModel) and simulates replicate count datasets
    (zinbSim) used as ground truth for bias/MSE computation
  repo: https://github.com/drisso/zinbwave
- name: scRNAseq
  role: Provides the real S1/CA1 single-cell count matrix from which the initial K=2
    ZINB-WaVE fit and simulation parameters are estimated
- name: BiocManager
  role: Installs the zinbwave Bioconductor package required for fitting and simulation
    (R >=3.4, Bioconductor >=3.6)
  repo: https://bioconductor.org/packages/zinbwave
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:f3f66625c5f26fbd0a5aede61c7278c9eb09287bcc62fb1d687c10edc5ff5d50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - BiocManager
  - Bioconductor
  - R
  - scRNAseq
  - zinbwave
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# estimator-bias-mse-visualization

## Summary

Simulation-based validation of ZINB-WaVE parameter estimators: refit the zero-inflated negative binomial model under misspecified numbers of latent factors K and gene-level covariates, then quantify and boxplot bias and MSE of ln(mu) and pi against known simulation truth.

## When to use

Use when you need to assess robustness of ZINB-WaVE estimates of ln(mu) and pi to misspecification of the number of latent covariates K or the gene-level covariate matrix V, by comparing refits to simulated ground truth.

## When NOT to use

- Input is real observational data without known true ln(mu) and pi parameters — bias and MSE cannot be computed against simulation truth
- The goal is low-dimensional representation/t-SNE visualization for downstream clustering (ZINB-WaVE's primary use), not estimator validation via refitting
- You are evaluating PCA or ZIFA embeddings, whose parameters (unlike ZINB-WaVE's ln(mu) and pi) are not directly comparable to a ZINB simulation truth

## Inputs

- S1/CA1 single-cell RNA-seq raw count matrix (loadable via the scRNAseq Bioconductor package)
- Fitted ZINB-WaVE model coefficients (ln(mu), pi, dispersion, W) with K=2, X=1n, V=1J, genewise dispersion
- Simulation settings: B=10 replicates, n=1000 cells, J=1000 genes

## Outputs

- 10 simulated ZINB count datasets per parameter setting (via zinbSim)
- Per-gene bias and MSE tables for ln(mu) and pi across K=1,2,3,4 and intercept-in/out settings
- Boxplot figure of estimator bias and MSE saved as bias_mse_boxplots.png

## How to apply

Load a real scRNA-seq count matrix (e.g., the S1/CA1 data via the scRNAseq Bioconductor package) and fit ZINB-WaVE once with K=2 latent factors, a sample-level intercept design X=1n, a gene-level covariate matrix V containing a column of ones (1J), and genewise dispersion estimation. Use the fitted coefficients with zinbModel/zinbSim from the zinbwave package to simulate B=10 independent count datasets of n=1000 cells and J=1000 genes. Refit ZINB-WaVE to each simulated dataset at K=1, 2, 3, and 4, once with and once without the gene-level intercept in V, using genewise dispersion. For every gene and setting, compute bias and mean squared error (MSE) of the ln(mu) and pi estimates relative to the true simulation parameters, then draw boxplots of bias and MSE across the K and intercept settings and save as bias_mse_boxplots.png. The rationale, grounded in the paper, is that including the gene-level intercept in V makes the cell-level intercept a global-scaling normalization factor, so estimators of ln(mu) and pi should remain unbiased even when K is misspecified.

## Related tools

- **zinbwave** (Fits the ZINB model (zinbFit/zinbModel) and simulates replicate count datasets (zinbSim) used as ground truth for bias/MSE computation) — https://github.com/drisso/zinbwave
- **scRNAseq** (Provides the real S1/CA1 single-cell count matrix from which the initial K=2 ZINB-WaVE fit and simulation parameters are estimated)
- **BiocManager** (Installs the zinbwave Bioconductor package required for fitting and simulation (R >=3.4, Bioconductor >=3.6)) — https://bioconductor.org/packages/zinbwave

## Examples

```
library(zinbwave); fit <- zinbFit(Y, X=matrix(1,nrow(Y),1), V=matrix(1,ncol(Y),1), K=2); sim <- zinbSim(fit, B=10)
```

## Evaluation signals

- Boxplots of bias for ln(mu) and pi are centered at or very near zero for misspecified K (K != 2) when the gene-level intercept V=1J is included, matching the paper's robustness finding
- MSE of ln(mu) and pi estimates is small across all K settings when the gene-level intercept is included
- K=2 refits (the simulation truth) recover the true ln(mu) and pi with near-zero bias, serving as a positive control
- Refits without the gene-level intercept show the expected contrast, confirming the intercept's normalization role is what confers robustness
- Figure file bias_mse_boxplots.png is produced with panels covering both bias and MSE across K=1,2,3,4 and intercept settings over B=10 simulations

## Limitations

- Estimation requires numerically optimizing a non-convex likelihood function, incurring computational cost
- Computing time is approximately linear in the number of cells and genes but approximately quadratic in the number of latent factors
- Exact reproduction of the paper's analyses requires zinbwave version 0.1.1, and the package requires R (>=3.4) and Bioconductor (>=3.6)
- Robustness of ln(mu) and pi does not imply the low-dimensional signal W extracted by ZINB-WaVE is biologically relevant if unwanted technical variation is unaccounted for

## Evidence

- [discussion] robust and unbiased estimators: "We have shown with simulated and real data analyses that ZINB-WaVE leads to robust and unbiased estimators of the underlying biological signals."
- [intro] sample-level and gene-level covariates: "includes a sample-level intercept, which serves as a global-scaling normalization factor, and gives the user the ability to include both gene-level and sample-level covariates"
- [discussion] gene-level intercept and raw counts: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [readme] reproducible zinbwave version: [evidence span withheld — anchor status: external_doc]
- [methods] installation pathway: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
