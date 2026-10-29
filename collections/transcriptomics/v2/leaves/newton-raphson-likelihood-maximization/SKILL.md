---
name: newton-raphson-likelihood-maximization
description: Use when fitting a zero-inflated Tweedie (ZICP) per-gene model to scRNA-seq
  read counts with excessive zeros and you need maximum-likelihood estimates of regression
  coefficients, dispersion, and the index parameter p.
when_to_use_negative:
- Zero-inflation is a property specific only to a subset of genes — the paper states
  Tweedieverse is 'not architected for the situation when zero-inflation is a property
  specific only to a subset of the genes'
- The response is a continuous non-count omics measurement better handled by the non-zero-inflated
  Tweedie variant
- Multi-subject designs or individual-level differential expression analysis are needed
  — the paper lists these as extensions not yet done
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: Tweedieverse
  role: R/Bioconductor package implementing the zero-inflated and non-zero-inflated
    Tweedie DE models fitted via Newton-Raphson
  repo: https://github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: Evaluates the Tweedie density via Fourier inversion during likelihood computation
  repo: https://github.com/himelmallick/Tweedieverse
- name: cplm (R package)
  role: Used with glm() and optim() to fit the initial Compound Poisson GLM at p=1.5
    for coefficient initialization
  repo: https://github.com/himelmallick/Tweedieverse
- name: statmod (R package)
  role: Supports the numerical computation of the Tweedie density and likelihood
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:599fdfb5220756c88fd992a51e6960352f794ccd30d5166ec91d61f7a4269004
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
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# newton-raphson-likelihood-maximization

## Summary

Fit zero-inflated Compound Poisson (ZICP/Tweedie) models for scRNA-seq differential expression by maximizing the zero-inflated log-likelihood over Θ=(β,γ,ϕ,p) with a Newton-Raphson algorithm, subject to ϕ>0 and p∈(1,2).

## When to use

Use when fitting a zero-inflated Tweedie (ZICP) per-gene model to scRNA-seq read counts with excessive zeros and you need maximum-likelihood estimates of regression coefficients, dispersion, and the index parameter p.

## When NOT to use

- Zero-inflation is a property specific only to a subset of genes — the paper states Tweedieverse is 'not architected for the situation when zero-inflation is a property specific only to a subset of the genes'
- The response is a continuous non-count omics measurement better handled by the non-zero-inflated Tweedie variant
- Multi-subject designs or individual-level differential expression analysis are needed — the paper lists these as extensions not yet done

## Inputs

- Zero-inflated scRNA-seq read count expression matrix (Ygc) with genes x cells
- Two-column design matrix X (intercept plus binary group indicator)
- Cell-specific library sizes S (e.g., SCRAN-estimated) used as a log-link offset
- Intercept-only zero-inflation design matrix Z

## Outputs

- Per-feature coefficient estimates (β̂) for each gene
- Wald test p-values from a χ²-distribution with 1 degree of freedom
- FDR-adjusted q-values and DE calls at α=0.05 (q<0.05)
- Estimated index parameter p̂ per gene written to zicp_de_results.csv

## How to apply

Set up the ZICP model with logit(q)=Zγ (intercept-only Z) and log(μ)=Xβ+log(S), where S is the cell-specific library size included as an offset. Initialize regression coefficients by fitting a non-zero-inflated Compound Poisson GLM with the index parameter fixed at p=1.5 plus a logistic regression for the zero sub-model. Evaluate the Tweedie density via the Fourier inversion method implemented in the R package tweedie, then maximize l(Θ|y)=Σ log[q_i*1(y_i=0)+(1−q_i)*Tw(y_i,β,ϕ,p)] using a Newton-Raphson algorithm under the constraints ϕ>0 and p∈(1,2). Compute Wald test statistics for H0: β_g=0 from the asymptotic normal distribution of β̂, yielding p-values from a χ²-distribution with 1 degree of freedom, then apply FDR adjustment to obtain q-values.

## Related tools

- **Tweedieverse** (R/Bioconductor package implementing the zero-inflated and non-zero-inflated Tweedie DE models fitted via Newton-Raphson) — https://github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (Evaluates the Tweedie density via Fourier inversion during likelihood computation) — https://github.com/himelmallick/Tweedieverse
- **cplm (R package)** (Used with glm() and optim() to fit the initial Compound Poisson GLM at p=1.5 for coefficient initialization) — https://github.com/himelmallick/Tweedieverse
- **statmod (R package)** (Supports the numerical computation of the Tweedie density and likelihood)

## Examples

```
library(Tweedieverse); fit <- Tweedieverse(feature_table=counts, metadata=meta, test='zicp', libSize=scransizes); write.csv(fit, 'zicp_de_results.csv')
```

## Evaluation signals

- Output table zicp_de_results.csv contains coefficient, p-value, q-value, and p̂ columns for every tested gene
- All q-values fall within [0,1] and DE calls are defined by q<0.05
- Fitted ϕ estimates are strictly positive and p estimates remain in the open interval (1,2)
- Wald p-values are consistent with a χ²-distribution with 1 degree of freedom under the null
- Initialization reproduces the p=1.5 Compound Poisson GLM fit before Newton-Raphson iteration begins

## Limitations

- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes
- No borrowed-information/shrinkage estimates are used for improved DE gene ranking; pooling/regularization benefits are untested
- Extension to multi-subject designs, individual-level DE, and differential subpopulation analysis is not yet done

## Evidence
