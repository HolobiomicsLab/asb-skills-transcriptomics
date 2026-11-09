---
name: single-cell-count-modeling
description: Use when modeling scRNA-seq gene-level count distributions (UMI or non-UMI)
  with heavy tails and point mass at zero, and you need platform-adaptive fitting
  with automatically estimated gene-specific index parameter p.
when_to_use_negative:
- Zero-inflation is a property specific to only a subset of genes — Tweedieverse is
  not architected for gene-specific zero-inflation affecting only a subset of genes
- Multi-subject experimental designs requiring individual-level or differential subpopulation
  analysis — this extension is not yet implemented or validated
- When standard preprocessing (normalization/transformation) is preferred over offset-based
  handling of cellular sequencing depth, or the response is already a normalized/transformed
  feature table rather than raw counts
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: Tweedieverse
  role: R/Bioconductor package implementing the Tweedie GLM fitting and DE testing
    pipeline
  repo: https://github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: Evaluates the Tweedie density for 1 < p < 2 via Fourier inversion
- name: cplm (R package)
  role: Fits the initial Compound Poisson GLM at p = 1.5 via Fisher's scoring
- name: statmod (R package)
  role: Supports the Tweedie density computation
- name: SCRAN
  role: Estimates cell-specific library sizes incorporated as offsets in per-gene
    Tweedie models
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:017ef7c79cfc6ad347c907bdd18fb56fc8e3aa67f54d093e298cd11056bbaacf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - R
  - SCRAN
  - Tweedieverse
  - cplm (R package)
  - statmod (R package)
  - tweedie (R package)
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# single-cell-count-modeling

## Summary

Model per-gene scRNA-seq counts with a three-parameter Tweedie (Compound Poisson-Gamma) GLM — mean (μ), dispersion (ϕ), and index/power parameter p ∈ (1, 2) — fitted via profile likelihood to capture heavy tails, sparsity, and zero-inflation across UMI and non-UMI platforms.

## When to use

Use when modeling scRNA-seq gene-level count distributions (UMI or non-UMI) with heavy tails and point mass at zero, and you need platform-adaptive fitting with automatically estimated gene-specific index parameter p.

## When NOT to use

- Zero-inflation is a property specific to only a subset of genes — Tweedieverse is not architected for gene-specific zero-inflation affecting only a subset of genes
- Multi-subject experimental designs requiring individual-level or differential subpopulation analysis — this extension is not yet implemented or validated
- When standard preprocessing (normalization/transformation) is preferred over offset-based handling of cellular sequencing depth, or the response is already a normalized/transformed feature table rather than raw counts

## Inputs

- Per-gene scRNA-seq count matrix (UMI or non-UMI/read counts)
- Design matrix X of covariates (e.g., condition labels)
- Cell-specific library sizes / normalization factors (SCRAN-estimated) used as offsets

## Outputs

- Per-gene estimates of mean μ, dispersion ϕ̂, and index parameter p̂ ∈ (1, 2)
- GLM coefficient estimates β̂ with standard errors from the Fisher information matrix
- Marginal p-values (Wald, asymptotic χ² with 1 df) and FDR-adjusted q-values per gene

## How to apply

For each gene, fit log(μ)=Xβ+log(S), passing library size (SCRAN-estimated) S as an offset. Evaluate the Tweedie density for 1 < p < 2 via the Fourier inversion strategy in the R package tweedie, since the normalizing constant has no closed form. Initialize β by fitting a Compound Poisson GLM with p fixed at 1.5 using Fisher's scoring (cplm), and initialize ϕ from the fitted residuals. Optimize θ=(ϕ, p) by profiling β out of the log-likelihood and maximizing the profile likelihood with optim(), subject to ϕ > 0 and p ∈ (1, 2), alternating profile-likelihood updates of θ with scoring updates of β̂ until convergence; optionally estimate zero-inflated (ZICP) parameters via Newton-Raphson. Refit the final Compound Poisson GLM at (ϕ̂, p̂) and compute Wald test p-values from the Fisher information matrix (with (j,l)-entry (1/ϕ)Σ μ_i^(2−p) X_ij X_il), FDR-adjust q-values, and call a gene DE when q < α = 0.05.

## Related tools

- **Tweedieverse** (R/Bioconductor package implementing the Tweedie GLM fitting and DE testing pipeline) — https://github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (Evaluates the Tweedie density for 1 < p < 2 via Fourier inversion)
- **cplm (R package)** (Fits the initial Compound Poisson GLM at p = 1.5 via Fisher's scoring)
- **statmod (R package)** (Supports the Tweedie density computation)
- **SCRAN** (Estimates cell-specific library sizes incorporated as offsets in per-gene Tweedie models)

## Examples

```
library(Tweedieverse); res <- Tweedieverse::Tweedieverse(counts, metadata, priors = c(1, 1, 1), libSize = scran::computeSumFactors(counts))
```

## Evaluation signals

- Optimization converged for each fitted gene
- Automatically estimated index parameter p̂ lies within the required interval (1, 2) for each gene
- Dispersion estimate satisfies ϕ̂ > 0
- Wald test statistic under the null follows an asymptotic χ²-distribution with 1 degree of freedom
- FDR-adjusted q-values control false discovery at the pre-specified threshold α = 0.05

## Limitations

- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes
- Extension to multi-subject experimental designs, individual-level differential expression analysis, and differential subpopulation analysis is not yet done
- No borrowed-information/shrinkage estimates for improved DE gene ranking; benefits of a regularization/pooling strategy are untested

## Evidence
