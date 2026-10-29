---
name: permutation-null-count-data-simulation
description: 'Use when benchmarking co-expression estimators or p-value calibration
  on scRNA-seq data: you need null UMI count matrices with realistic depth variation
  but zero true co-expression, e.g. to reproduce bias curves or type-I error Q-Q plots.'
when_to_use_negative:
- You need null data with true co-expressed gene pairs — the Poisson-latent construction
  forces zero true co-expression by design
- Your count model is not UMI-based Poisson-like (e.g. full-length read data); the
  article's measurement model uses only a Poisson distribution, deferring negative
  binomial adaptation to future work
- You are evaluating cell-type-specific clustering or differential co-expression rather
  than estimator bias and p-value calibration on null data
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: CS-CORE
  role: Generative model whose latent-variable/Poisisson formulation defines the null
    simulation scheme; also the reference estimator validated on the null data
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: CSCORE (R package)
  role: Run on each simulated null dataset via CSCORE() on raw UMI counts to obtain
    co-expression estimates and p-values
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: Seurat
  role: Load and handle the source PBMC COVID Seurat object (blish_covid.seu.rds)
    from which simulation parameters are derived
- name: locCSN
  role: Comparison method run on the same permuted null datasets to assess bias and
    type-I error
  repo: https://github.com/xuranw/locCSN
- name: normalisr
  role: Comparison method (Normalisr) run on the same permuted null datasets
  repo: https://github.com/lingfeiwang/normalisr
- name: Noise Regularization
  role: Comparison method run on the same permuted null datasets
  repo: https://github.com/RuoyuZhang/NoiseRegularization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:011a6e5697a4184b8df47a0f7a6f2b86391b87e720aeaedcd8d35eabfff23cae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CSCORE
  - CSCORE (R package)
  - Noise Regularization
  - Seurat
  - locCSN
  - normalisr
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# permutation-null-count-data-simulation

## Summary

Generates permuted null scRNA-seq UMI count datasets under the CS-CORE generative model (latent z_i ~ F_p(mu, Sigma); x_ij | z_ij ~ Poisson(s_i * z_ij)) so that sequencing depth varies across cells but no gene pairs are truly co-expressed, enabling calibration and bias assessment of co-expression estimators.

## When to use

Use when benchmarking co-expression estimators or p-value calibration on scRNA-seq data: you need null UMI count matrices with realistic depth variation but zero true co-expression, e.g. to reproduce bias curves or type-I error Q-Q plots.

## When NOT to use

- You need null data with true co-expressed gene pairs — the Poisson-latent construction forces zero true co-expression by design
- Your count model is not UMI-based Poisson-like (e.g. full-length read data); the article's measurement model uses only a Poisson distribution, deferring negative binomial adaptation to future work
- You are evaluating cell-type-specific clustering or differential co-expression rather than estimator bias and p-value calibration on null data

## Inputs

- Seurat object with raw UMI counts and cell metadata (e.g. blish_covid.seu.rds from the Wilk et al. PBMC COVID atlas)
- Empirical gene mean vector (mu) and covariance structure (Sigma) estimated from the source data
- Cell-specific sequencing depth factors (s_i) from the source data

## Outputs

- Permuted null UMI count matrices per replicate with depth variation but no true co-expressed pairs
- Per-method co-expression estimates and p-values on each null replicate
- Bias curves (mean expression vs. average estimated co-expression) and p-value Q-Q plots vs. Uniform(0,1)

## How to apply

Load a real Seurat object such as the PBMC COVID dataset (blish_covid.seu.rds from Wilk et al.) and derive empirical parameters (gene means mu and covariances Sigma, cell-specific sequencing depths s_i). Simulate latent true expressions z_i from F_p(mu, Sigma) with Sigma constraining pairs to be uncorrelated, then draw observed counts x_ij from Poisson(s_i * z_ij) so depth varies while all gene pairs are true nulls. Run each estimator (e.g. CSCORE() on raw UMI counts, plus comparison methods like locCSN, normalisr, Noise Regularization, Pearson, Spearman, SpQN, propr, rho-sctransform, rho-analytic PR) on each null replicate. Aggregate per-method co-expression estimates over gene pairs binned by mean expression to build depth-dependent bias curves, and aggregate p-values with Benjamini-Hochberg correction for Q-Q plots against Uniform(0,1).

## Related tools

- **CS-CORE** (Generative model whose latent-variable/Poisisson formulation defines the null simulation scheme; also the reference estimator validated on the null data) — https://github.com/ChangSuBiostats/CS-CORE
- **CSCORE (R package)** (Run on each simulated null dataset via CSCORE() on raw UMI counts to obtain co-expression estimates and p-values) — https://github.com/ChangSuBiostats/CS-CORE
- **Seurat** (Load and handle the source PBMC COVID Seurat object (blish_covid.seu.rds) from which simulation parameters are derived)
- **locCSN** (Comparison method run on the same permuted null datasets to assess bias and type-I error) — https://github.com/xuranw/locCSN
- **normalisr** (Comparison method (Normalisr) run on the same permuted null datasets) — https://github.com/lingfeiwang/normalisr
- **Noise Regularization** (Comparison method run on the same permuted null datasets) — https://github.com/RuoyuZhang/NoiseRegularization

## Examples

```
install_github("ChangSuBiostats/CS-CORE"); CSCORE_result <- CSCORE(null_counts, genes = genes_selected)
```

## Evaluation signals

- Average co-expression estimates from CS-CORE on the null data should be unbiased and centered around zero across all bins of mean expression / sequencing depth
- CS-CORE p-values aggregated across replicates and BH-corrected should fall on the identity diagonal in the Q-Q plot against Uniform(0,1)
- Empirical type-I error at the 0.05 threshold should be at or near nominal for a calibrated method; most comparison methods should show inflated false positives
- Null count matrices should exhibit the intended depth variation (Poisson with cell-specific s_i) while preserving no true co-expression between gene pairs

## Limitations

- The measurement model uses only a Poisson distribution; adaptation to other count distributions such as negative binomial is deferred to future work
- Simulation assumes gene expressions from cells follow a single distribution, which may not hold across individuals with different genetic, demographic and clinical characteristics
- Null data are model-based permutations under the CS-CORE generative model, so they favor methods matching that model's assumptions

## Evidence

- [methods] Generative model for null data: [evidence span withheld — anchor status: not_found]
- [results] CS-CORE unbiasedness on null data: [evidence span withheld — anchor status: not_found]
- [methods] Calibration criterion: [evidence span withheld — anchor status: not_found]
- [discussion] Poisson-only measurement model limitation: "It can be useful to adapt CS-CORE to model xij∣zij using other distributions that model nonnegative integers such as the negative binomial distribution and we leave it to future work."
- [readme] Install CS-CORE from GitHub: "CS-CORE` is a R package for cell-type-specific co-expression inference from single cell RNA-sequencing data."
