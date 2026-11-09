---
name: wald-test-inference
description: 'Use when a fitted Compound Poisson (CPLM) or zero-inflated Compound
  Poisson (ZICP) model needs per-gene or per-feature H0: beta_g = 0 testing with p-values,
  q-values, and DE calls in a two-group design.'
when_to_use_negative:
- When the analysis goal is multi-subject designs, individual-level DE, or differential
  subpopulation analysis — the article states this extension has not yet been validated
- When zero-inflation is a property specific only to a subset of genes — Tweedieverse
  is not architected for that situation
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Tweedieverse
  role: R/Bioconductor package implementing CPLM/ZICP fitting and Wald-test-based
    DE testing with FDR correction
  repo: https://github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: Evaluates the Tweedie density function (Fourier inversion) needed for profile
    likelihood and subsequent Wald inference
- name: cplm (R package)
  role: Fits the Compound Poisson GLM at p = 1.5 used to initialize coefficients before
    Wald testing
- name: statmod (R package)
  role: Supports the numerical computation for Tweedie model fitting underlying the
    Wald test
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:c2edfd9099fbbe6ed89a0eb3037472d526a345f617addc8c0789b9316946a6b2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - Bioconductor
  - R
  - Tweedieverse
  - cplm
  - cplm (R package)
  - statmod
  - statmod (R package)
  - tweedie (R package)
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# Wald test inference

## Summary

Per-feature significance testing of differential expression in Tweedieverse using the Wald test statistic, which follows an asymptotic chi-squared distribution with 1 degree of freedom under the null, followed by FDR adjustment and calling features DE at alpha = 0.05.

## When to use

Use when a fitted Compound Poisson (CPLM) or zero-inflated Compound Poisson (ZICP) model needs per-gene or per-feature H0: beta_g = 0 testing with p-values, q-values, and DE calls in a two-group design.

## When NOT to use

- When the analysis goal is multi-subject designs, individual-level DE, or differential subpopulation analysis — the article states this extension has not yet been validated
- When zero-inflation is a property specific only to a subset of genes — Tweedieverse is not architected for that situation

## Inputs

- Fitted per-feature CPLM or ZICP model objects with coefficient estimates and standard errors (fitted via tweedie, cplm, and statmod R packages)
- Two-group design matrix with an intercept and binary group indicator
- Library size (or normalization factor) per sample, used as log(S) offset

## Outputs

- Per-feature Wald test p-values (asymptotic chi-squared, 1 df)
- FDR-adjusted q-values
- Differential expression calls at alpha = 0.05
- Table of feature-level coefficients, p-values, q-values, and estimated per-feature index parameter (de_results.csv)

## How to apply

After fitting the CPLM or ZICP model per feature via profile likelihood (index parameter initialized at p = 1.5 with a Compound Poisson GLM; ZICP fitted via Newton-Raphson), compute the Wald test statistic for the group coefficient beta_g under H0: beta_g = 0. The statistic follows an asymptotic chi-squared distribution with 1 degree of freedom under the null, from which p-values are derived. Adjust p-values for the False Discovery Rate to obtain q-values, and call a feature (gene or metabolite/taxon) differentially expressed when the q-value falls below the pre-specified Type 1 error threshold alpha = 0.05. The constraint that the index parameter p lies in (1, 2) keeps the Tweedie distribution semi-continuous with a point mass at zero, which is what makes the Wald-based inference valid for sparse count data with library size log(S) entered as an offset via the log link.

## Related tools

- **Tweedieverse** (R/Bioconductor package implementing CPLM/ZICP fitting and Wald-test-based DE testing with FDR correction) — https://github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (Evaluates the Tweedie density function (Fourier inversion) needed for profile likelihood and subsequent Wald inference)
- **cplm (R package)** (Fits the Compound Poisson GLM at p = 1.5 used to initialize coefficients before Wald testing)
- **statmod (R package)** (Supports the numerical computation for Tweedie model fitting underlying the Wald test)

## Examples

```
Tweedieverse(feature_table, metadata, test = "wald", fdr = "BH", cutoff = 0.05)
```

## Evaluation signals

- de_results.csv exists and contains coefficient, p-value, q-value, and index-parameter columns for every feature
- q-values are FDR-adjusted and significant calls are thresholded at alpha = 0.05
- Wald test p-values are consistent with an asymptotic chi-squared distribution with 1 degree of freedom (non-negative, in [0, 1])
- Estimated per-feature index parameters fall within (1, 2) for the semi-continuous Compound Poisson-Gamma regime
- Reproduced power/FDR comparison confirms Tweedieverse controls FDR while producing high F1 and AUC scores

## Limitations

- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes
- No borrowed-information/shrinkage estimates are available for improved DE gene ranking; benefits of regularization/pooling are untested
- Extension to multi-subject experimental designs, individual-level DE, and differential subpopulation analysis has not yet been done
- Extension to other modalities (metagenomics, metatranscriptomics, spatial transcriptomics) remains a conjecture, not validated

## Evidence
