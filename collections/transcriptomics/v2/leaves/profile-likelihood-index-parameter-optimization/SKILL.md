---
name: profile-likelihood-index-parameter-optimization
description: Use when fitting per-gene Tweedie (CPLM/ZICP) models to UMI or read-count
  scRNA-seq data and the index parameter p must be estimated rather than fixed, e.g.
  to compare gene-specific p across experimental protocols.
when_to_use_negative:
- When zero-inflation is a property specific only to a subset of genes — the article
  notes Tweedieverse 'is not architected for the situation when zero-inflation is
  a property specific only to a subset of the genes'
- When p can be fixed a priori and only regression coefficients are needed — the full
  profile-likelihood optimization is unnecessary overhead
- When the goal is a simple ad-hoc normalization/transformation rather than model-based
  DE testing — the paper argues the offset-based Tweedie model 'bypass[es] the need
  for an adhoc normalization and/or transformation prior to DE testing'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Tweedieverse
  role: R/Bioconductor package implementing the CPLM/ZICP fitting with index and dispersion
    parameter estimation
  repo: https://github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: Evaluates the Tweedie density function by Fourier inversion during likelihood
    computation
- name: cplm (R package)
  role: Compound Poisson linear model machinery used in the profile-likelihood computation
- name: statmod (R package)
  role: Numerical support for the Tweedie density computation
- name: SCRAN
  role: Estimates cell-specific library sizes incorporated as log offsets in the per-gene
    Tweedie models
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:753d85edca717a504fa5848ca961b253cae6ce8cd4679be207f9ecb45304863c
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# profile-likelihood-index-parameter-optimization

## Summary

Optimizes the Tweedie index (power) parameter p and dispersion phi for per-gene Compound Poisson Linear Models fitted to scRNA-seq counts, by profiling out regression coefficients and maximizing the Tweedie log-likelihood subject to phi > 0 and p in (1, 2).

## When to use

Use when fitting per-gene Tweedie (CPLM/ZICP) models to UMI or read-count scRNA-seq data and the index parameter p must be estimated rather than fixed, e.g. to compare gene-specific p across experimental protocols.

## When NOT to use

- When zero-inflation is a property specific only to a subset of genes — the article notes Tweedieverse 'is not architected for the situation when zero-inflation is a property specific only to a subset of the genes'
- When p can be fixed a priori and only regression coefficients are needed — the full profile-likelihood optimization is unnecessary overhead
- When the goal is a simple ad-hoc normalization/transformation rather than model-based DE testing — the paper argues the offset-based Tweedie model 'bypass[es] the need for an adhoc normalization and/or transformation prior to DE testing'

## Inputs

- Gene-by-cell scRNA-seq count matrix (UMI counts, e.g. 10x Genomics PBMC data with 13,713 genes and 2,638 cells)
- Non-UMI read-count scRNA-seq matrices (e.g. GEO accessions GSE67835 and GSE107585)
- Cell-specific library sizes / normalization factors S (SCRAN-estimated), used as log offsets

## Outputs

- Per-gene estimates of the Tweedie index parameter p (all within (1, 2))
- Per-gene dispersion estimates phi (> 0)
- Converged CPLM/ZICP model fits usable for downstream Wald testing (chi-squared, 1 df)

## How to apply

For each gene, fit a Compound Poisson Linear Model (CPLM for UMI counts) or Zero-inflated Compound Poisson Linear Model (ZICP for read counts) with log(mu) = X*beta + log(S), where S is the SCRAN-estimated cell-specific library size used as an offset. Initialize the index parameter at p = 1.5 by fitting an initial Compound Poisson GLM, then estimate theta = (phi, p) by profiling out beta and maximizing the log-likelihood l(theta) (profile likelihood for CPLM; Newton-Raphson for ZICP) using the glm() and optim() functions of base R together with the cplm, statmod, and tweedie packages. Because the Tweedie density has no closed form, evaluate it numerically by Fourier inversion via the tweedie R package. Iterate until convergence, constraining phi > 0 and p within the open interval (1, 2), where the Tweedie distribution is a semi-continuous Compound Poisson-Gamma with a point mass at zero.

## Related tools

- **Tweedieverse** (R/Bioconductor package implementing the CPLM/ZICP fitting with index and dispersion parameter estimation) — https://github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (Evaluates the Tweedie density function by Fourier inversion during likelihood computation)
- **cplm (R package)** (Compound Poisson linear model machinery used in the profile-likelihood computation)
- **statmod (R package)** (Numerical support for the Tweedie density computation)
- **SCRAN** (Estimates cell-specific library sizes incorporated as log offsets in the per-gene Tweedie models)

## Examples

```
fit <- optim(par = c(phi = 1, p = 1.5), fn = profile_loglik, method = "L-BFGS-B", lower = c(1e-8, 1.001), upper = c(Inf, 1.999))
```

## Evaluation signals

- All per-gene p estimates fall within the constrained open interval (1, 2) and phi estimates are strictly positive
- Optimization converges starting from the p = 1.5 Compound Poisson GLM initialization
- Distributions of gene-specific p estimates reproduce the significant cross-protocol separation reported across the 16 scRNA-seq datasets (Fig. 1)
- Downstream Wald test statistics under the null follow an asymptotic chi-squared distribution with 1 degree of freedom
- FDR-controlled DE calls at alpha = 0.05 are consistent with the Tweedieverse benchmark (outperforming >10 DE methods in power and FDR control)

## Limitations

- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes
- The method does not yet borrow information across genes to obtain shrinkage estimates for better DE gene ranking
- Extension to multi-subject designs, individual-level differential expression, and differential subpopulation analysis has not been done

## Evidence
