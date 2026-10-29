---
name: single-cell-sample-level-prior-modeling
description: Use when you have multi-sample single-cell count data with sample-level
  metadata (e.g., disease status) and need a latent embedding whose sample-specific
  transformations are modeled as a function of condition, enabling downstream vector
  field and TF activity analyses.
when_to_use_negative:
- When the analysis goal is a standard sequential workflow (normalize, integrate,
  cluster, pseudo-bulk DGE) without condition-informed sample modeling — the article
  argues this ignores the interplay between integration and DGE but a simple integration
  may suffice
- When a non-linear manifold approximation is required — 'GEDI currently supports
  linear manifold learning, with the option to further restrict the manifold to a
  hyperellipsoid'
- When the data are not multi-sample or lack sample-level condition labels, since
  the sample-level prior H and sample-specific parameters (Qi) have nothing to encode
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: GEDI
  role: Generative Bayesian model whose sample-specific manifold parameters (Qi) are
    fit against a sample-level prior H encoding condition; optimized via its R implementation
  repo: https://github.com/csglab/GEDI/
- name: Harmony
  role: Provides the integrated space (cohort 1 of the COVID-19 dataset) that GEDI's
    sample-specific transformations are asked to approximate
- name: scuttle
  role: Basic QC steps (log10_total_counts, log10_total_features_by_counts, subsets_Mito_percent
    filters) before running GEDI
- name: scran
  role: Pre-processing dependency for normalization prior to fitting the GEDI model
- name: scater
  role: Pre-processing/scater library used alongside scuttle for QC before running
    GEDI
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:db46838330c6377e4e2ff1ba4776968f2e4039b37a822cb3453ed15cabfe897b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - Harmony
  - R
  - scater
  - scran
  - scuttle
  - uwot
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# single-cell-sample-level-prior-modeling

## Summary

Set up and optimize a GEDI generative model in which sample-specific latent manifold parameters (Qi) are informed by a sample-level prior design matrix H encoding condition contrasts (e.g., healthy vs severe COVID-19), unifying integration, denoising, and condition modeling within one Bayesian framework.

## When to use

Use when you have multi-sample single-cell count data with sample-level metadata (e.g., disease status) and need a latent embedding whose sample-specific transformations are modeled as a function of condition, enabling downstream vector field and TF activity analyses.

## When NOT to use

- When the analysis goal is a standard sequential workflow (normalize, integrate, cluster, pseudo-bulk DGE) without condition-informed sample modeling — the article argues this ignores the interplay between integration and DGE but a simple integration may suffice
- When a non-linear manifold approximation is required — 'GEDI currently supports linear manifold learning, with the option to further restrict the manifold to a hyperellipsoid'
- When the data are not multi-sample or lack sample-level condition labels, since the sample-level prior H and sample-specific parameters (Qi) have nothing to encode

## Inputs

- Cell-level raw count matrix for a multi-sample single-cell atlas
- Sample-level metadata with condition labels (e.g., disease status: healthy vs severe)
- Harmony-integrated low-dimensional embedding of the cells (used as target manifold for cohort 1 of the COVID-19 dataset)
- Sample-level prior design matrix H encoding the condition contrast
- Optional gene-level prior knowledge (e.g., DoRothEA human gene regulatory network for TF activity inference)

## Outputs

- Fitted GEDI model with optimized o, Z, Bi, Qi, oi and si components
- Sample-specific manifold parameters (Qi) informed by disease status
- Convergence tracking plots from model$plotTracking()
- Latent embedding supporting downstream vector field, activity gradient, and differential expression queries

## How to apply

Load the cohort atlas (e.g., COVID-19 cohort 1 in Harmony-integrated space) with its cell-level count matrix and sample-level metadata containing the condition contrast. Construct a sample-level prior matrix H encoding disease severity so that GEDI's sample-specific manifold parameters (Qi) are informed by the condition, and ask GEDI to identify sample-specific transformations approximating Harmony's integrated space. Initialize latent variables and optimize the model, tracking convergence of the o, Z, Bi, Qi, oi and si components with model$plotTracking(). Pre-processing uses scuttle/scran/scater QC, filtering low-quality cells (log10_total_counts > 2.5, log10_total_features_by_counts > 2, subsets_Mito_percent <= 20) and low-expressed genes (genes with sum of counts > 3 in more than 3 cells). The fitted model object (also deposited on Zenodo) then supports downstream svd.vectorField.gedi, svd.activityGradient.gedi, getA.gedi/getADB.gedi, and getDiffExp.gedi calls.

## Related tools

- **GEDI** (Generative Bayesian model whose sample-specific manifold parameters (Qi) are fit against a sample-level prior H encoding condition; optimized via its R implementation) — https://github.com/csglab/GEDI/
- **Harmony** (Provides the integrated space (cohort 1 of the COVID-19 dataset) that GEDI's sample-specific transformations are asked to approximate)
- **scuttle** (Basic QC steps (log10_total_counts, log10_total_features_by_counts, subsets_Mito_percent filters) before running GEDI)
- **scran** (Pre-processing dependency for normalization prior to fitting the GEDI model)
- **scater** (Pre-processing/scater library used alongside scuttle for QC before running GEDI)

## Examples

```
library(GEDI); model <- GEDI(sample_vector, counts, H=disease_severity_prior); model$run(); model$plotTracking()
```

## Evaluation signals

- Convergence of all optimized components (o, Z, Bi, Qi, oi, si) as tracked by model$plotTracking()
- The sample-specific transformations (Qi) approximate Harmony's integrated space for the same samples
- Fitted model reproduces the reported cell-type ordering of transcriptomic vector field magnitudes (plasmablasts, then HLA-DRlo S100Ahi monocytes, then neutrophils, per Fig. 3d)
- Downstream queries (e.g., SPI1 activity gradient in monocytes) align with the severe-vs-healthy vector field and SPI1 mRNA is significantly up-regulated via getDiffExp.gedi
- Reproducibility against the Zenodo-deposited pre-fit GEDI models (e.g., zenodo.org/record/8222040/, zenodo.org/record/8222698/)

## Limitations

- GEDI currently only supports linear manifold learning, with the option to restrict the manifold to a hyperellipsoid; other parametric manifold approximations are future extensions
- It remains untested whether GEDI's gene-level expression imputations are competitive with existing single-cell imputation methods
- Counterfactual prediction in unobserved cell types (e.g., perturbation response prediction) via GEDI's vector field remains untested
- Installation requires RcppEigen and gfortran; code was tested with R 4.0.0 on CentOS Linux 7

## Evidence

- [results] sample-level prior H encoding disease severity informs Qi: [evidence span withheld — anchor status: not_found]
- [discussion] Harmony space as integration target: [evidence span withheld — anchor status: not_found]
- [results] Convergence tracking of model components: [evidence span withheld — anchor status: not_found]
- [discussion] Linear manifold limitation: "GEDI currently supports linear manifold learning, with the option to further restrict the manifold to a hyperellipsoid"
- [methods] QC thresholds before fitting: "filter<- sce$log10_total_counts > 2.5 &
    sce$log10_total_features_by_counts > 2 &
    sce$subsets_Mito_percent <= 20"
- [readme] Installation command from README: [evidence span withheld — anchor status: external_doc]
