---
name: zero-inflated-compound-poisson-modeling
description: Use when modeling scRNA-seq read counts (non-UMI, e.g. SMART-Seq2 or
  Fluidigm C1 plate-based protocols) with excess zeros, or when gene-specific Tweedie
  index parameter p must be estimated and compared across experimental platforms.
when_to_use_negative:
- Zero-inflation is a property specific only to a subset of the genes — the method
  is not architected for that situation (per the discussion)
- Data are continuous abundances or UMI counts better handled by the non-zero-inflated
  CPLM variant of the same framework
- Multi-subject designs or individual-level / differential subpopulation DE is required
  — extension and validation of Tweedieverse to these designs is not yet done
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Tweedieverse
  role: Implements the zero-inflated Compound Poisson (ZICP) model for per-gene differential
    expression testing of read counts
  repo: https://github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: Evaluates the Tweedie density function via Fourier inversion for likelihood
    computation
- name: cplm (R package)
  role: Compound Poisson GLM fitting used to initialize coefficients at p = 1.5
- name: statmod (R package)
  role: Supporting numerical computation for Tweedie density evaluation
- name: SCRAN
  role: Estimates cell-specific library sizes incorporated as offsets in the per-gene
    Tweedie models
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:585b5dce8538105a9560f6f6091b9253cee7519092b4b770d0510c3b0a1beb90
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - Bioconductor
  - R
  - SCRAN
  - Tweedieverse
  - cplm (R package)
  - statmod (R package)
  - tweedie (R package)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# zero-inflated-compound-poisson-modeling

## Summary

Fits zero-inflated Compound Poisson (ZICP) models — a Tweedie GLM variant with 1 < p < 2 — to non-UMI (read-count) scRNA-seq data, jointly estimating the index parameter p, dispersion phi, and zero-inflation, to capture heavy tails, sparsity, and point mass at zero.

## When to use

Use when modeling scRNA-seq read counts (non-UMI, e.g. SMART-Seq2 or Fluidigm C1 plate-based protocols) with excess zeros, or when gene-specific Tweedie index parameter p must be estimated and compared across experimental platforms.

## When NOT to use

- Zero-inflation is a property specific only to a subset of the genes — the method is not architected for that situation (per the discussion)
- Data are continuous abundances or UMI counts better handled by the non-zero-inflated CPLM variant of the same framework
- Multi-subject designs or individual-level / differential subpopulation DE is required — extension and validation of Tweedieverse to these designs is not yet done

## Inputs

- Gene-by-cell raw read-count matrix from a non-UMI scRNA-seq protocol (e.g. plate-based full-length read counts)
- Cell-level metadata / design matrix X (e.g. binary condition labels, covariates)
- Cell-specific library sizes or normalization factors S (estimated with SCRAN)
- Optionally filtered gene sets (e.g. genes expressed in at least 10% of cells in negative control experiments)

## Outputs

- Per-gene estimates of the Tweedie index parameter p (constrained to (1, 2)) and dispersion phi
- Per-gene Wald test p-values, FDR-adjusted q-values, and fold changes for each feature
- DE gene calls at alpha = 0.05, plus zero-inflation parameter estimates for the ZICP fit

## How to apply

Estimate cell-specific library sizes with the SCRAN R/Bioconductor package and incorporate them as offsets under the log link log(mu) = X*beta + log(S). For each gene, fit a Zero-inflated Compound Poisson (ZICP) model using Tweedieverse, initializing coefficients with a Compound Poisson GLM at p = 1.5 and evaluating the Tweedie density by Fourier inversion via the tweedie R package. Optimize the index and dispersion parameters (phi > 0, 1 < p < 2) by Newton-Raphson for the zero-inflated model, iterating until convergence; assess differential expression via the Wald test (asymptotic chi-squared, 1 df) with FDR control at alpha = 0.05. The rationale is that for 1 < p < 2 the Tweedie distribution reduces to a semi-continuous distribution with a point mass at zero (Compound Poisson-Gamma), jointly accommodating zero-inflation and heavy tails without ad hoc normalization or transformation.

## Related tools

- **Tweedieverse** (Implements the zero-inflated Compound Poisson (ZICP) model for per-gene differential expression testing of read counts) — https://github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (Evaluates the Tweedie density function via Fourier inversion for likelihood computation)
- **cplm (R package)** (Compound Poisson GLM fitting used to initialize coefficients at p = 1.5)
- **statmod (R package)** (Supporting numerical computation for Tweedie density evaluation)
- **SCRAN** (Estimates cell-specific library sizes incorporated as offsets in the per-gene Tweedie models)

## Examples

```
library(cplm); fit0 <- cpglm(Y ~ condition + offset(log(S)), link = "log", data = dat)  # initialize coefficients at p = 1.5 before ZICP optimization in Tweedieverse
```

## Evaluation signals

- All estimated index parameters p must lie within the constrained range (1, 2) and dispersion phi must be positive
- Wald test statistics under the null follow an asymptotic chi-squared distribution with 1 degree of freedom — q-values should be well-calibrated under negative controls
- In cross-platform comparison, per-gene p estimates should reproduce the significant differences across 16 publicly available scRNA-seq datasets (Fig. 1) reported in the paper
- ZICP on read counts should outperform state-of-the-art DE methods on FDR control and statistical power in benchmarking (e.g. via BenchmarkSingleCell outputs of marginal p-values, adjusted q-values, fold changes)
- Convergence of Newton-Raphson optimization of (phi, p) should be verified before extracting parameter estimates

## Limitations

- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes
- No borrowed-information / shrinkage estimates are available for improved DE gene ranking; benefits of a regularization/pooling strategy are untested
- Extension to multi-subject experimental designs, individual-level differential expression analysis, and differential subpopulation analysis is not yet done
- Applicability to other modalities (metagenomics, metatranscriptomics, spatial transcriptomics) remains a conjecture, not validated

## Evidence

- [readme] Bypass of ad hoc normalization: [evidence span withheld — anchor status: not_found]
