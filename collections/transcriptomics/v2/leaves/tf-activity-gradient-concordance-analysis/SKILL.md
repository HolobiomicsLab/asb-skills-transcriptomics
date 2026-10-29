---
name: tf-activity-gradient-concordance-analysis
description: Use when you have a multi-sample, multi-condition single-cell atlas with
  a fitted GEDI model and prior TF-gene networks, and need to test whether TF activity
  gradients (e.g., SPI1, CEBPA, SP1) concord with the disease vector field in a defined
  cell type.
when_to_use_negative:
- Input is not a multi-sample, multi-condition count matrix with sample-level covariates
  — GEDI's sample-specific manifold parameters (Qi) require a condition contrast to
  model
- No prior TF-gene network is available for the TFs of interest, since TF activity
  inference depends on DoRothEA high-confidence sets A, B, and C
- The analysis goal is non-linear manifold learning or counterfactual perturbation
  prediction, which GEDI currently does not support
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GEDI
  role: 'Core generative model: infers TF/regulatory network activities from gene-level
    priors, computes the transcriptomic vector field and TF activity gradients, and
    performs cluster-free DGE (getDiffExp.gedi)'
  repo: https://github.com/csglab/GEDI
- name: Harmony
  role: Provides the integrated embedding space for the COVID-19 cohort 1 samples
    that GEDI's sample-specific transformations approximate
- name: R
  role: Analysis environment with scuttle/scran/scater for QC and preprocessing before
    GEDI model fitting
- name: uwot
  role: Dependency used in the notebooks for embedding/visualization of the integrated
    space
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:652314474114971ff36540771f7cf72bef541e270f34f581b2077e65e21da953
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - Harmony
  - R
  - uwot
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tf-activity-gradient-concordance-analysis

## Summary

A GEDI-based analysis that tests whether transcription factor (TF) activity gradients align with the transcriptomic vector field of a condition (e.g., severe COVID-19) within specific cell types such as HLA-DRlo S100Ahi monocytes, using prior knowledge gene regulatory networks (DoRothEA) to infer TF activities.

## When to use

Use when you have a multi-sample, multi-condition single-cell atlas with a fitted GEDI model and prior TF-gene networks, and need to test whether TF activity gradients (e.g., SPI1, CEBPA, SP1) concord with the disease vector field in a defined cell type.

## When NOT to use

- Input is not a multi-sample, multi-condition count matrix with sample-level covariates — GEDI's sample-specific manifold parameters (Qi) require a condition contrast to model
- No prior TF-gene network is available for the TFs of interest, since TF activity inference depends on DoRothEA high-confidence sets A, B, and C
- The analysis goal is non-linear manifold learning or counterfactual perturbation prediction, which GEDI currently does not support

## Inputs

- Harmony-integrated single-cell atlas (e.g., COVID-19 cohort 1) with cell-level raw count matrix and sample-level metadata including disease status (healthy vs severe)
- Sample-level prior matrix H encoding condition contrast (disease severity)
- Prior gene regulatory network (human DoRothEA TF-gene interactions, restricted to high-confidence sets A, B, C)
- Cell type annotations (e.g., HLA-DRlo S100Ahi monocytes, plasmablasts, neutrophils)
- Fitted GEDI model object (pre-computed models available at zenodo.org/record/8222040/ and 8222698/)

## Outputs

- Per-cell transcriptomic vector field in the integrated embedding (svd.vectorField.gedi output rendered via plot_vectorField)
- TF activity gradients per cell type (svd.activityGradient.gedi / svd.joint_vectorField_gradient.gedi)
- TF activity scores per cell (getA.gedi / getADB.gedi)
- TF differential expression test (getDiffExp.gedi, e.g., SPI1 mRNA up-regulation)
- Cell-type ranking by mean vector magnitude (plasmablasts, HLA-DRlo S100Ahi monocytes, neutrophils)

## How to apply

Fit a GEDI model on the Harmony-integrated atlas (e.g., COVID-19 cohort 1) using the sample vector, raw count matrix, and a sample-level prior matrix H encoding disease severity, checking convergence of o, Z, Bi, Qi, oi, si components with model$plotTracking(). Compute the severe-versus-healthy transcriptomic vector field with svd.vectorField.gedi, restrict TF-gene interactions to the high-confidence DoRothEA sets A, B, and C, and retrieve TF activities with getA.gedi/getADB.gedi. Compute the TF (e.g., SPI1) activity gradient in the target cell type (HLA-DRlo S100Ahi monocytes) using svd.activityGradient.gedi and svd.joint_vectorField_gradient.gedi, and test TF mRNA up-regulation with getDiffExp.gedi. Judge success when the gradient direction aligns with the vector field in monocytes, the TF mRNA is significantly up-regulated, and the per-cell-type vector magnitude ranking matches reported orderings (plasmablasts, HLA-DRlo S100Ahi monocytes, neutrophils).

## Related tools

- **GEDI** (Core generative model: infers TF/regulatory network activities from gene-level priors, computes the transcriptomic vector field and TF activity gradients, and performs cluster-free DGE (getDiffExp.gedi)) — https://github.com/csglab/GEDI
- **Harmony** (Provides the integrated embedding space for the COVID-19 cohort 1 samples that GEDI's sample-specific transformations approximate)
- **R** (Analysis environment with scuttle/scran/scater for QC and preprocessing before GEDI model fitting)
- **uwot** (Dependency used in the notebooks for embedding/visualization of the integrated space)

## Examples

```
devtools::install_github("csglab/GEDI"); library(GEDI); A <- getA.gedi(model); grad <- svd.activityGradient.gedi(model, tf = "SPI1")
```

## Evaluation signals

- SPI1 activity gradient direction aligns with the severe-vs-healthy transcriptomic vector field in HLA-DRlo S100Ahi monocytes
- SPI1 mRNA is significantly up-regulated by getDiffExp.gedi in monocytes
- Cell-type ranking by mean vector magnitude reproduces the reported Fig. 3d ordering: plasmablasts first, followed by HLA-DRlo S100Ahi monocytes and neutrophils
- GEDI model components (o, Z, Bi, Qi, oi, si) converge as verified with model$plotTracking()
- TF-gene interactions are restricted to DoRothEA high-confidence sets A, B, and C before activity inference

## Limitations

- GEDI currently supports only linear manifold learning (with optional hyperellipsoid restriction); other parametric manifold approximations are future extensions
- Counterfactual prediction in unobserved cell types (e.g., perturbation response prediction) via GEDI's vector field remains untested
- Whether GEDI's gene-level expression imputations are competitive with existing single-cell imputation methods remains to be tested

## Evidence

- [abstract] GEDI can also incorporate gene-level prior knowledge to infer pathway and regulatory network activities in single cells.: "GEDI can also incorporate gene-level prior knowledge to infer pathway and regulatory network activities in single cells."
- [discussion] Harmony integration for COVID-19 cohort 1: "we used the integrated space identified by Harmony41 for the samples in cohort 1 of the COVID-19 dataset"
- [methods] High-confidence TF-gene interaction restriction: "restricted the TF-gene interactions to the high-confidence sets A, B, and C"
- [readme] Gradient vector field notebook for COVID-19 data: [evidence span withheld — anchor status: external_doc]
