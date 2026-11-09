---
name: zero-inflated-count-simulation
description: Use when you need synthetic zero-inflated, over-dispersed scRNA-seq counts
  with known true parameters (ln(mu), pi) to benchmark a model's robustness to misspecification
  of K, V, or dispersion.
when_to_use_negative:
- Input is not zero-inflated count data (e.g., already-normalized or log-transformed
  expression matrices); ZINB-WaVE models raw counts directly
- The goal is differential expression or type I error simulation for bulk-style DE
  analyses — the PlateEffects2016 simulation framework targets that setting instead
- You only need a low-dimensional embedding of real data without a ground-truth simulation/benchmark
  component
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: zinbwave
  role: Provides zinbModel/zinbSim for simulating counts from a fitted ZINB model
    and zinbFit for refitting under misspecified K and V settings
  repo: https://github.com/drisso/zinbwave
- name: scRNAseq
  role: Supplies the real S1/CA1 single-cell count matrix used to fit the generating
    ZINB-WaVE model
- name: BiocManager
  role: Installation channel for zinbwave and other Bioconductor packages
  repo: https://bioconductor.org/packages/zinbwave
- name: ZINB-WaVE
  role: The zero-inflated negative binomial model whose estimators (ln(mu), pi) are
    simulated and benchmarked
  repo: https://github.com/drisso/zinb_analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:a5cb3881ac16aeac18d8e35030d4fea75cb9427e075b270268f5c37e23cc0ae6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - BiocManager
  - Bioconductor
  - R
  - ZINB-WaVE
  - scRNAseq
  - zinbwave
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# zero-inflated-count-simulation

## Summary

Simulates single-cell RNA-seq count matrices from a fitted ZINB-WaVE model using zinbModel/zinbSim, so that estimator bias and MSE can be assessed by refitting under misspecified settings (e.g., wrong number of latent factors K or missing gene-level intercept).

## When to use

Use when you need synthetic zero-inflated, over-dispersed scRNA-seq counts with known true parameters (ln(mu), pi) to benchmark a model's robustness to misspecification of K, V, or dispersion.

## When NOT to use

- Input is not zero-inflated count data (e.g., already-normalized or log-transformed expression matrices); ZINB-WaVE models raw counts directly
- The goal is differential expression or type I error simulation for bulk-style DE analyses — the PlateEffects2016 simulation framework targets that setting instead
- You only need a low-dimensional embedding of real data without a ground-truth simulation/benchmark component

## Inputs

- Single-cell RNA-seq count matrix (e.g., S1/CA1 data from the scRNAseq Bioconductor package)
- Fitted zinbModel object with K=2, X = 1n sample-level intercept, V including a column of ones (1J), genewise dispersion
- Simulation design choices: B=10 replicates, n=1000 cells, J=1000 genes, K in {1,2,3,4}, V with/without intercept

## Outputs

- 10 simulated count matrices (1000 cells x 1000 genes) with known true ln(mu) and pi
- Per-gene bias and MSE tables for ln(mu) and pi under each K and V setting
- Boxplot figure of bias and MSE across K and intercept settings (bias_mse_boxplots.png)

## How to apply

First fit ZINB-WaVE to a real count matrix (e.g., the S1/CA1 data via the scRNAseq package) with K=2 latent factors, a sample-level intercept design X = 1n, a gene-level covariate matrix V containing a column of ones (1J), and genewise dispersion; this supplies the true ln(mu) and pi values. Pass the fitted coefficients into zinbModel and draw B=10 independent simulations with zinbSim, each of n=1000 cells and J=1000 genes. Refit ZINB-WaVE to each simulated dataset under misspecified settings (K = 1, 2, 3, 4; with and without the gene-level intercept in V) and compute gene-wise bias and MSE of ln(mu) and pi estimates relative to the true simulation parameters. Summarize with boxplots; the rationale is that the paper found no or very small bias when K is misspecified provided the gene-level intercept is retained.

## Related tools

- **zinbwave** (Provides zinbModel/zinbSim for simulating counts from a fitted ZINB model and zinbFit for refitting under misspecified K and V settings) — https://github.com/drisso/zinbwave
- **scRNAseq** (Supplies the real S1/CA1 single-cell count matrix used to fit the generating ZINB-WaVE model)
- **BiocManager** (Installation channel for zinbwave and other Bioconductor packages) — https://bioconductor.org/packages/zinbwave
- **ZINB-WaVE** (The zero-inflated negative binomial model whose estimators (ln(mu), pi) are simulated and benchmarked) — https://github.com/drisso/zinb_analysis

## Examples

```
library(zinbwave); fit <- zinbFit(counts, X=matrix(1,n), V=cbind(1,gene.cov), K=2); sim <- zinbSim(zinbModel(..., beta=fit@beta, gamma=fit@gamma, alpha=fit@alpha, W=fit@W, zeta=fit@zeta), ncell=1000, B=10)
```

## Evaluation signals

- For refits with the gene-level intercept (V = 1J) included, boxplots of bias for ln(mu) and pi must be centered at or very near zero for misspecified K (K != 2)
- MSE of ln(mu) and pi estimates must be small across K in {1, 3, 4} when the gene-level intercept is included, matching the paper's robustness finding
- Each simulated dataset must have the specified dimensions n=1000 cells and J=1000 genes, and B=10 independent replicates must be generated
- Refits must use genewise dispersion and a sample-level intercept design X = 1n so estimates are comparable to the true simulation parameters

## Limitations

- zinbwave requires R (>=3.4) and Bioconductor (>=3.6); exact paper reproduction uses zinbwave version 0.1.1 (install_github('drisso/archive-zinbwave@v0.1.1'))
- Estimation requires numerical optimization of a non-convex likelihood, so refitting 10 simulated datasets across 4 K values and 2 V settings is computationally costly; computing time is approximately linear in cells and genes and quadratic in the number of latent factors
- Simulations only test robustness to misspecified K, V, and dispersion; the low-dimensional signal W may capture unwanted technical variation if it is not accounted for in the model

## Evidence

- [intro] The approach is implemented in the open-source R package zinbwave, publicly available through the Bioconductor Project: "The approach is implemented in the open-source R package zinbwave, publicly available through the Bioconductor Project"
- [results] Misspecifying K with gene-level intercept yields no or small bias: [evidence span withheld — anchor status: not_found]
- [discussion] Global-scaling normalization via gene-level column of ones: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [methods] zinbwave version and R/Bioconductor requirements: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [readme] README: zinb_analysis repo reproduces paper simulations: [evidence span withheld — anchor status: external_doc]
