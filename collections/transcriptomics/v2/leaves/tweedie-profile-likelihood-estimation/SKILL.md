---
name: tweedie-profile-likelihood-estimation
description: Use when fitting per-gene (or per-feature) GLMs to sparse, semi-continuous
  count or abundance data where the optimal Tweedie index parameter p is unknown and
  must be estimated automatically within the constrained interval (1, 2).
when_to_use_negative:
- Zero-inflation is a property specific only to a subset of the genes — the article
  states Tweedieverse 'is not architected for the situation when zero-inflation is
  a property specific only to a subset of the genes'
- Multi-subject experimental designs, individual-level differential expression, or
  differential subpopulation analysis — extension of Tweedieverse to these designs
  is explicitly listed as not yet done
- The goal requires borrowing information across genes (shrinkage estimates for DE
  ranking) — the article notes this is not implemented and left to further research
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Tweedieverse
  role: R/Bioconductor package implementing the CPLM/ZICP fitting with profile-likelihood
    estimation of (ϕ, p)
  repo: github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: Evaluates the Tweedie density via the Fourier inversion strategy needed since
    the normalizing constant has no closed form
- name: cplm (R package)
  role: Fits the initial Compound Poisson GLM at p = 1.5 by Fisher scoring to initialize
    β and residuals for dispersion
- name: statmod (R package)
  role: Supporting numerical routines for the profile-likelihood computation
- name: R stats (glm(), optim())
  role: Base R functions used to fit the initial GLM and maximize the profile likelihood
    over θ = (ϕ, p)
- name: SCRAN
  role: Estimates cell-specific library sizes incorporated as an offset log(S) in
    the per-gene Tweedie models
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_002
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:5574ec9229cce49fed1f9792d6440a7940933679b916ffc26750d7762fbcaab5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bioconductor
  - R
  - R stats (glm(), optim())
  - SCRAN
  - Tweedieverse
  - cplm (R package)
  - statmod (R package)
  - tweedie (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.96
n_claims: 25
---

# tweedie-profile-likelihood-estimation

## Summary

Fits per-feature Compound Poisson Linear Models (CPLM/ZICP) from the Tweedie family by jointly estimating the dispersion ϕ and index parameter p ∈ (1, 2) via profile likelihood, enabling self-adaptive semi-continuous modeling of scRNA-seq and other omics data.

## When to use

Use when fitting per-gene (or per-feature) GLMs to sparse, semi-continuous count or abundance data where the optimal Tweedie index parameter p is unknown and must be estimated automatically within the constrained interval (1, 2).

## When NOT to use

- Zero-inflation is a property specific only to a subset of the genes — the article states Tweedieverse 'is not architected for the situation when zero-inflation is a property specific only to a subset of the genes'
- Multi-subject experimental designs, individual-level differential expression, or differential subpopulation analysis — extension of Tweedieverse to these designs is explicitly listed as not yet done
- The goal requires borrowing information across genes (shrinkage estimates for DE ranking) — the article notes this is not implemented and left to further research

## Inputs

- Gene-by-cell (or feature-by-sample) count matrix from scRNA-seq (UMI or non-UMI read counts), microbiome taxonomic/functional profiles, or metabolomics compound abundances / MS peak intensities
- Design matrix X with covariates (e.g., intercept + binary group indicator)
- Cell- or sample-specific library sizes (or normalization factors), e.g., SCRAN-estimated, used as offset log(S)

## Outputs

- Per-feature estimated Tweedie parameters: coefficients β̂, dispersion ϕ̂, and index parameter p̂ ∈ (1, 2)
- Standard errors of β̂ from the Fisher information matrix
- Feature-level table of coefficients, Wald-test p-values, FDR-adjusted q-values, and estimated index parameters (e.g., de_results.csv)

## How to apply

Load the feature-by-sample measurement matrix and design matrix X, and incorporate cell/sample-specific library sizes S (e.g., estimated with SCRAN) as an offset under the log link log(μ) = Xβ + log(S). Because the Tweedie normalizing constant for 1 < p < 2 has no closed form, evaluate the density by the Fourier inversion strategy implemented in the R package tweedie. Initialize β by fitting an initial Compound Poisson GLM with the index fixed at p = 1.5 using Fisher's scoring (cplm package), and initialize the dispersion ϕ from the fitted residuals; then optimize θ = (ϕ, p) by profiling β out of the log-likelihood and maximizing the profile likelihood with optim(), subject to ϕ > 0 and p ∈ (1, 2), iterating between profile-likelihood updates of θ and scoring-based updates of β̂ until convergence. If excess zeros are expected (e.g., non-UMI read counts), fit the zero-inflated Compound Poisson (ZICP) variant instead, using Newton-Raphson on the ZICP likelihood. Refit the final CPLM at (ϕ̂, p̂) and obtain standard errors of β̂ from the Fisher information matrix whose (j,l)-th entry is (1/ϕ)Σ μ_i^(2−p) X_ij X_il.

## Related tools

- **Tweedieverse** (R/Bioconductor package implementing the CPLM/ZICP fitting with profile-likelihood estimation of (ϕ, p)) — github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (Evaluates the Tweedie density via the Fourier inversion strategy needed since the normalizing constant has no closed form)
- **cplm (R package)** (Fits the initial Compound Poisson GLM at p = 1.5 by Fisher scoring to initialize β and residuals for dispersion)
- **statmod (R package)** (Supporting numerical routines for the profile-likelihood computation)
- **R stats (glm(), optim())** (Base R functions used to fit the initial GLM and maximize the profile likelihood over θ = (ϕ, p))
- **SCRAN** (Estimates cell-specific library sizes incorporated as an offset log(S) in the per-gene Tweedie models)

## Evaluation signals

- The automatically estimated index parameter p̂ lies within the required interval (1, 2) for every fitted gene, and ϕ̂ > 0 (the stated model constraints)
- The optimization converges (profile-likelihood updates of θ alternating with scoring updates of β̂ terminate per the stated convergence criterion)
- Standard errors computed from the Fisher information matrix are finite and decrease as ϕ or p increase, consistent with the reported monotonicity of the observed Fisher Information in both parameters
- Per-gene p̂ estimates vary significantly across experimental protocols when compared across the 16 public scRNA-seq datasets, all remaining within (1, 2) as in the paper's Fig. 1
- Downstream Wald p-values follow an asymptotic χ² distribution with 1 degree of freedom under H0: beta_g = 0, and q-values are FDR-adjusted with calls thresholded at α = 0.05

## Limitations

- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes
- Extension to multi-subject designs, individual-level DE, and differential subpopulation analysis has not yet been done; extension to other modalities (metagenomics, metatranscriptomics, spatial transcriptomics) remains a conjecture
- The Tweedie density for 1 < p < 2 must be evaluated numerically (Fourier inversion), as the normalizing constant has no closed form, adding computational cost per gene

## Evidence
