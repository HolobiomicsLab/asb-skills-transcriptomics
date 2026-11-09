---
name: compound-poisson-glm-fitting
description: Use when fitting per-gene differential expression models to semi-continuous
  count or relative-abundance data (UMI or read counts, microbiome, metabolomics)
  where heavy tails and zero mass demand a three-parameter Tweedie GLM.
when_to_use_negative:
- When zero-inflation is a property specific to only a subset of genes — the paper
  states Tweedieverse is 'not architected for the situation when zero-inflation is
  a property specific only to a subset of the genes'
- When the goal requires multi-subject designs, individual-level DE, or differential
  subpopulation analysis — these extensions are not yet implemented or validated
- When data are already normalized/transformed feature tables that preclude the log(S)
  offset formulation (the method deliberately bypasses ad hoc normalization or transformation)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3577
tools:
- name: Tweedieverse
  role: R/Bioconductor package orchestrating the per-feature Tweedie/ZICP GLM fitting
    and DE testing
  repo: https://github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: Evaluates the Tweedie density via Fourier inversion for 1 < p < 2
- name: cplm (R package)
  role: Fits the initial Compound Poisson GLM with index parameter p = 1.5 by Fisher's
    scoring
- name: statmod (R package)
  role: Supports the numerical Tweedie density computation in the fitting pipeline
- name: R (stats package)
  role: Provides glm() and optim() used for coefficient fitting and profile-likelihood
    optimization of (ϕ, p)
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_004
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:9b2dd84ad854e458725726950f03bf14f5167709b73e4a20432fa35e4054c3fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - R
  - R (stats package)
  - Tweedieverse
  - cplm (R package)
  - statmod (R package)
  - tweedie (R package)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# compound-poisson-glm-fitting

## Summary

Per-gene (or per-feature) generalized linear model fitting with a Tweedie (Compound Poisson, 1 < p < 2) response for differential analysis of sparse, semi-continuous scRNA-seq and other omics data, with automated estimation of the index parameter p and dispersion ϕ by profile likelihood.

## When to use

Use when fitting per-gene differential expression models to semi-continuous count or relative-abundance data (UMI or read counts, microbiome, metabolomics) where heavy tails and zero mass demand a three-parameter Tweedie GLM.

## When NOT to use

- When zero-inflation is a property specific to only a subset of genes — the paper states Tweedieverse is 'not architected for the situation when zero-inflation is a property specific only to a subset of the genes'
- When the goal requires multi-subject designs, individual-level DE, or differential subpopulation analysis — these extensions are not yet implemented or validated
- When data are already normalized/transformed feature tables that preclude the log(S) offset formulation (the method deliberately bypasses ad hoc normalization or transformation)

## Inputs

- Per-gene scRNA-seq expression matrix (UMI or read counts, e.g. Ygc) or feature-by-sample omics abundance matrix
- Design matrix X (e.g. intercept plus binary group indicator) and metadata
- Cell/sample-specific library sizes or normalization factors S (SCRAN-estimated), used as a log-link offset

## Outputs

- Per-feature coefficient estimates, standard errors from the Fisher information matrix (entry (j,l) = (1/ϕ)Σ μ_i^(2−p) X_ij X_il), Wald p-values, and FDR-adjusted q-values
- Estimated per-feature index parameter p̂ and dispersion ϕ̂
- DE calls at α = 0.05 (e.g. de_results.csv / zicp_de_results.csv)

## How to apply

For each gene, set up the log-link model log(μ)=Xβ+log(S), passing the cell-specific library size (SCRAN-normalized) S as an offset. Evaluate the Tweedie density for 1 < p < 2 via Fourier inversion (tweedie R package), since the normalizing constant has no closed form. Initialize β by fitting a Compound Poisson GLM with the index parameter fixed at p = 1.5 (cplm package, Fisher's scoring) and initialize ϕ from the fitted residuals. Optimize θ = (ϕ, p) by profiling β out of the log-likelihood and maximizing the profile likelihood with optim() under the constraints ϕ > 0 and p ∈ (1, 2), alternating with scoring-based β updates until convergence; optionally use the Newton-Raphson zero-inflated (ZICP) variant when excess zeros are present. Refit with the optimized (ϕ̂, p̂), compute Wald tests for H0: β_g = 0 (asymptotic χ² with 1 df), and FDR-adjust p-values, calling features DE at q < 0.05.

## Related tools

- **Tweedieverse** (R/Bioconductor package orchestrating the per-feature Tweedie/ZICP GLM fitting and DE testing) — https://github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (Evaluates the Tweedie density via Fourier inversion for 1 < p < 2)
- **cplm (R package)** (Fits the initial Compound Poisson GLM with index parameter p = 1.5 by Fisher's scoring)
- **statmod (R package)** (Supports the numerical Tweedie density computation in the fitting pipeline)
- **R (stats package)** (Provides glm() and optim() used for coefficient fitting and profile-likelihood optimization of (ϕ, p))

## Examples

```
Tweedieverse(feature_table = Y_gc, metadata = X, output = "de_results", random_effect = FALSE)
```

## Evaluation signals

- Optimization converged and the automatically estimated index parameter p̂ lies within the required interval (1, 2) for every fitted gene
- Estimated dispersion ϕ̂ > 0 for all fitted features
- Output table contains coefficient, p-value, q-value, and index-parameter (p̂) columns for every tested feature
- Wald test statistics yield p-values from an asymptotic χ² distribution with 1 degree of freedom; q-values lie in [0,1] and are FDR-adjusted
- DE calls defined consistently by q < 0.05 threshold

## Limitations

- Not designed for gene-specific zero-inflation affecting only a subset of genes
- No borrowed-information/shrinkage estimates across genes to improve DE gene ranking; benefits of regularization/pooling untested
- Extension to multi-subject designs, individual-level DE, and differential subpopulation analysis not yet done
- Use in other modalities (metagenomics, metatranscriptomics, spatial transcriptomics) remains a conjecture rather than a validated claim

## Evidence

- [methods] when 1 < p < 2, the Tweedie distribution reduces to a semi-continuous distribution with a point mass at zero: "when 1 < p < 2, the Tweedie distribution reduces to a semi-continuous distribution with a point mass at zero"
- [methods] offset formulation: "log(μ)=Xβ+log(S), where S is the library size (or normalization factor) included in the model as an offset"
- [methods] profile-likelihood estimation of θ: "θ = (ϕ, p) can be estimated by profiling out β and maximizing l(θ)"
- [methods] p = 1.5 initialization: "the index parameter is first set to 1.5 and an initial Compound Poisson GLM is fit with p = 1.5"
- [methods] DE calling rule: "infer a gene to be differentially expressed between two experimental conditions if the FDR-adjusted p-value (q-value) falls below a pre-specified Type 1 error threshold (e.g., α = 0.05)"
- [readme] package scope: "Tweedieverse is an R package for differential analysis of omics data implementing a range of statistical methodology based on the Tweedie distribution"
