---
name: wald-test-differential-expression
description: Use when you have fitted per-gene Tweedie-family GLMs (CPLM or ZICP)
  to scRNA-seq count data and need coefficient-level significance testing with multiplicity
  adjustment to call differentially expressed genes.
when_to_use_negative:
- Zero-inflation is a property specific to only a subset of genes — Tweedieverse is
  not architected for gene-specific zero-inflation
- Multi-subject experimental designs, individual-level differential expression, or
  differential subpopulation analysis — extension and validation of the method to
  these settings is not yet done
- The data require shrinkage / borrowed-information estimates for improved DE gene
  ranking — the paper does not provide such pooling for the proposed methods
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Tweedieverse
  role: R/Bioconductor package that fits the per-gene Tweedie and zero-inflated Tweedie
    models and computes the Wald test p-values and FDR-adjusted q-values
  repo: https://github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: Evaluates the Tweedie density via the Fourier inversion method used inside
    the model likelihood whose coefficients are Wald-tested
- name: cplm (R package)
  role: Used with the stats base R glm() and optim() functions for the Compound Poisson
    computations underlying the fitted models
- name: statmod (R package)
  role: Supports the above-mentioned computations for the Tweedie density evaluation
    in the fitting procedure
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:de1e1b1c0d2ce8e5f64d8e3e84c6f21dcef4ca3c3ca15880cd3e3aadcdd11be7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - R
  - Tweedieverse
  - cplm (R package)
  - statmod (R package)
  - tweedie (R package)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# wald-test-differential-expression

## Summary

Per-feature Wald testing for differential expression in the Tweedieverse framework: after fitting per-gene Tweedie (CPLM) or zero-inflated Tweedie (ZICP) GLMs, the coefficient for the group indicator is tested against zero using the asymptotic normal distribution of the MLE, yielding p-values that are FDR-adjusted to q-values.

## When to use

Use when you have fitted per-gene Tweedie-family GLMs (CPLM or ZICP) to scRNA-seq count data and need coefficient-level significance testing with multiplicity adjustment to call differentially expressed genes.

## When NOT to use

- Zero-inflation is a property specific to only a subset of genes — Tweedieverse is not architected for gene-specific zero-inflation
- Multi-subject experimental designs, individual-level differential expression, or differential subpopulation analysis — extension and validation of the method to these settings is not yet done
- The data require shrinkage / borrowed-information estimates for improved DE gene ranking — the paper does not provide such pooling for the proposed methods

## Inputs

- Zero-inflated scRNA-seq read count expression matrix (Ygc, genes x cells)
- Two-column design matrix X (intercept plus binary group indicator)
- Cell-specific library sizes S (estimated with SCRAN), used as offset
- Fitted per-feature CPLM or ZICP model objects with asymptotic standard errors

## Outputs

- Per-gene Wald test p-values from a chi-squared distribution with 1 degree of freedom
- FDR-adjusted q-values per gene
- DE calls at alpha = 0.05 (q < 0.05)
- Results table (e.g., zicp_de_results.csv) with coefficient, p-value, q-value, and estimated p-hat columns

## How to apply

For each gene, fit the Tweedie or zero-inflated Tweedie model with log(mu) = X_beta + log(S), where the two-column design matrix X holds an intercept plus a binary group indicator and library sizes S enter as an offset (estimated via SCRAN). Compute the Wald test statistic for H0: beta_g = 0 versus H1: beta_g != 0 using the standard error of beta-hat from its asymptotic normal distribution (the observed Fisher Information matrix decreases in both dispersion phi and index p). Under the null, the statistic follows an asymptotic chi-squared distribution with 1 degree of freedom, from which the per-gene p-value is derived. Apply FDR adjustment to obtain q-values and flag a gene as differentially expressed when the q-value falls below alpha = 0.05. Write coefficients, p-values, q-values, and estimated p-hat per gene to the results table (e.g., zicp_de_results.csv).

## Related tools

- **Tweedieverse** (R/Bioconductor package that fits the per-gene Tweedie and zero-inflated Tweedie models and computes the Wald test p-values and FDR-adjusted q-values) — https://github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (Evaluates the Tweedie density via the Fourier inversion method used inside the model likelihood whose coefficients are Wald-tested)
- **cplm (R package)** (Used with the stats base R glm() and optim() functions for the Compound Poisson computations underlying the fitted models)
- **statmod (R package)** (Supports the above-mentioned computations for the Tweedie density evaluation in the fitting procedure)

## Evaluation signals

- Output table contains coefficient, p-value, q-value, and p-hat columns for every tested gene
- All q-values lie within [0, 1]
- DE calls are defined by q < 0.05 (pre-specified Type 1 error threshold alpha = 0.05)
- Each p-value is consistent with a chi-squared distribution with 1 degree of freedom under the null (asymptotic Wald inference)
- Wald statistics use the standard error of beta-hat from its asymptotic normal distribution, not resampled or heuristic variances

## Limitations

- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes
- No borrowed-information / shrinkage estimates for improved DE gene ranking; benefits of a regularization or pooling strategy for the proposed methods are untested
- Extension to multi-subject designs, individual-level DE, differential subpopulation analysis, and other modalities (metagenomics, spatial transcriptomics) remains unvalidated

## Evidence

- [methods] Wald p-value computation in the paper's method: [evidence span withheld — anchor status: not_found]
- [readme] Benchmark output schema includes adjusted p-values: [evidence span withheld — anchor status: not_found]
