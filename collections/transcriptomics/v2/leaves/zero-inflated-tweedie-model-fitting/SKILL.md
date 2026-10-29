---
name: zero-inflated-tweedie-model-fitting
description: Use when input is a raw scRNA-seq read-count matrix (e.g. Ygc) with excessive
  zero counts from a plate-based protocol, and a per-gene differential expression
  test between two groups is needed.
when_to_use_negative:
- The data are already normalized or transformed continuous abundances rather than
  raw read counts, since the model relies on log(μ)=Xβ+log(S) with library size as
  an offset on counts
- Zero-inflation is suspected to be a property specific only to a subset of genes,
  which the paper explicitly states Tweedieverse is not architected for
- UMI-based droplet data with modest sparsity where a non-zero-inflated Tweedie (CPLM)
  fit suffices and zero-inflation modeling adds unnecessary complexity
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Tweedieverse
  role: R/Bioconductor package implementing the ZICP per-feature fit for differential
    expression of zero-inflated scRNA-seq counts
  repo: https://github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: evaluates the Tweedie density function via the Fourier inversion method needed
    for the ZICP likelihood
- name: cplm (R package)
  role: used with stats base R glm()/optim() to carry out the Compound Poisson computations
    and coefficient initialization
- name: statmod (R package)
  role: supports the Tweedie density/likelihood computation for model fitting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:7b8745d5a87f2a23ff8d8a031683d5e1f1ad5f62cc28e7d168a31e9e3a1c6670
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# zero-inflated-tweedie-model-fitting

## Summary

Fit a zero-inflated Tweedie (ZICP) model per feature to zero-inflated scRNA-seq read-count data, extending the Tweedie family so the zero probability mass can exceed that of a traditional Tweedie (Compound Poisson-Gamma, 1 < p < 2) distribution.

## When to use

Use when input is a raw scRNA-seq read-count matrix (e.g. Ygc) with excessive zero counts from a plate-based protocol, and a per-gene differential expression test between two groups is needed.

## When NOT to use

- The data are already normalized or transformed continuous abundances rather than raw read counts, since the model relies on log(μ)=Xβ+log(S) with library size as an offset on counts
- Zero-inflation is suspected to be a property specific only to a subset of genes, which the paper explicitly states Tweedieverse is not architected for
- UMI-based droplet data with modest sparsity where a non-zero-inflated Tweedie (CPLM) fit suffices and zero-inflation modeling adds unnecessary complexity

## Inputs

- zero-inflated scRNA-seq read count expression matrix (Ygc), genes x cells
- two-column design matrix X (intercept plus binary group indicator)
- cell-specific library sizes S (SCRAN-estimated), used as log-link offset

## Outputs

- per-gene coefficient estimates (β̂) from the Tweedie sub-model
- per-gene Wald-test p-values (χ², 1 df)
- FDR-adjusted q-values with DE calls at q < 0.05
- estimated zero-inflation probability (p̂) per gene, written to zicp_de_results.csv

## How to apply

Set up the model with logit(q)=Zγ (intercept-only Z) for the zero sub-model and log(μ)=Xβ+log(S) for the Tweedie sub-model, where S is the library size (SCRAN-estimated) included as an offset. Initialize coefficients by fitting a non-zero-inflated Compound Poisson GLM with the index parameter set to p=1.5 plus a logistic regression for the zero sub-model. Evaluate the Tweedie density via the Fourier inversion method implemented in the R package tweedie, then maximize the zero-inflated log-likelihood l(Θ|y)=Σ log[q_i*1(y_i=0)+(1−q_i)*Tw(y_i,β,ϕ,p)] with respect to Θ=(β,γ,ϕ,p) using a Newton-Raphson algorithm under the constraints ϕ>0 and p∈(1,2). For each gene, compute the Wald test statistic for H0: β_g=0, obtaining a p-value from an asymptotic χ²-distribution with 1 degree of freedom, then apply FDR adjustment to flag genes as differentially expressed at α=0.05.

## Related tools

- **Tweedieverse** (R/Bioconductor package implementing the ZICP per-feature fit for differential expression of zero-inflated scRNA-seq counts) — https://github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (evaluates the Tweedie density function via the Fourier inversion method needed for the ZICP likelihood)
- **cplm (R package)** (used with stats base R glm()/optim() to carry out the Compound Poisson computations and coefficient initialization)
- **statmod (R package)** (supports the Tweedie density/likelihood computation for model fitting)

## Examples

```
Tweedieverse(feature_table = Ygc, response = X, test = "zicp")
```

## Evaluation signals

- Output table zicp_de_results.csv contains coefficient, p-value, q-value, and p̂ columns for every tested gene
- All q-values lie within [0,1] and DE calls are defined by q < 0.05 (α = 0.05 threshold)
- Constrained optimization converged with ϕ > 0 and p ∈ (1, 2) for each gene's fit
- Per-gene Wald p-values are consistent with an asymptotic χ²-distribution with 1 degree of freedom
- Initialization matches the profile-likelihood approach: index parameter first set to 1.5 with an initial Compound Poisson GLM fit

## Limitations

- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes
- Extension to multi-subject experimental designs, individual-level differential expression analysis, and differential subpopulation analysis is not yet done
- No borrowed-information/shrinkage estimates are available for improved DE gene ranking; benefits of a regularization/pooling strategy remain untested

## Evidence
