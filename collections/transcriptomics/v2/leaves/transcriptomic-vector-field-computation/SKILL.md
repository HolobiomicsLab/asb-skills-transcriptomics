---
name: transcriptomic-vector-field-computation
description: Use when you have a fitted GEDI model over a multi-sample, multi-condition
  single-cell dataset (e.g., healthy vs severe COVID-19) and need a cluster-free,
  per-cell-type direction/magnitude field of the condition effect plus TF activity
  gradients.
when_to_use_negative:
- You need perturbation-response prediction in cell types not observed in the data
  — counterfactual prediction via GEDI's vector field is currently untested
- You only need standard pseudo-bulk per-cluster differential expression; a sequential
  workflow (normalize, integrate, cluster, DGE per cell type) may suffice and is simpler
- Your modality is not linear-embeddable — GEDI currently only supports linear manifold
  learning (with optional hyperellipsoid restriction)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GEDI
  role: Fits the multi-sample generative model and provides svd.vectorField.gedi,
    plot_vectorField, getA.gedi, getADB.gedi, svd.activityGradient.gedi, svd.joint_vectorField_gradient.gedi,
    getDiffExp.gedi, and model$plotTracking()
  repo: https://github.com/csglab/GEDI
- name: Harmony
  role: Provides the integrated space for cohort 1 samples that GEDI's sample-specific
    transformations approximate
- name: uwot
  role: Used for embedding/visualization support in the notebook workflow
  repo: https://cran.r-project.org/web/packages/uwot/index.html
- name: R
  role: Implementation environment (scuttle/scran/scater for preprocessing; Rcpp/RcppEigen,
    ggplot2, metR, rsvd, scales as GEDI dependencies)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:9fe6570157b9be258ad45f6fee9b654020426bfd0f059be90ac4fabd10abd104
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - Harmony
  - R
  - uwot
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# transcriptomic-vector-field-computation

## Summary

Compute and visualize the severe-vs-healthy transcriptomic vector field of COVID-19 cohort 1 with GEDI's svd.vectorField.gedi/plot_vectorField in the Harmony-integrated embedding, then rank cell types by mean vector magnitude and test TF activity-gradient concordance (e.g., SPI1 in HLA-DRlo S100Ahi monocytes).

## When to use

Use when you have a fitted GEDI model over a multi-sample, multi-condition single-cell dataset (e.g., healthy vs severe COVID-19) and need a cluster-free, per-cell-type direction/magnitude field of the condition effect plus TF activity gradients.

## When NOT to use

- You need perturbation-response prediction in cell types not observed in the data — counterfactual prediction via GEDI's vector field is currently untested
- You only need standard pseudo-bulk per-cluster differential expression; a sequential workflow (normalize, integrate, cluster, DGE per cell type) may suffice and is simpler
- Your modality is not linear-embeddable — GEDI currently only supports linear manifold learning (with optional hyperellipsoid restriction)

## Inputs

- Harmony-integrated single-cell atlas for COVID-19 cohort 1 (integrated embedding)
- Cell-level raw count matrix
- Sample-level metadata with disease status (healthy vs severe)
- Sample-level prior matrix H encoding disease severity
- Fitted GEDI model object (e.g., from Zenodo record 8222040/8222698)
- TF-gene regulatory network prior (human DoRothEA, high-confidence sets A, B, C)

## Outputs

- Per-cell transcriptomic vector field of severe-vs-healthy COVID-19 in the integrated embedding
- Vector field plot colored by cell type (plot_vectorField)
- Cell-type ranking by mean vector magnitude
- TF activity values (getA.gedi/getADB.gedi) and TF activity gradients (svd.activityGradient.gedi, svd.joint_vectorField_gradient.gedi)
- Cluster-free differential expression results (getDiffExp.gedi) for SPI1 mRNA

## How to apply

Load the COVID-19 cohort 1 atlas (Harmony-integrated space, cell-level counts, sample-level metadata with disease status) and fit a GEDI model with the sample vector, raw count matrix, and a sample-level prior H encoding severity so sample-specific manifold parameters (Qi) are informed by the condition contrast; check convergence of o, Z, Bi, Qi, oi, si with model$plotTracking(). Compute the severe-versus-healthy transcriptomic vector field in the integrated embedding with svd.vectorField.gedi and render it with plot_vectorField colored by cell type. Aggregate per-cell vector magnitudes by cell type and rank types by mean magnitude, comparing against the reported ordering (plasmablasts, HLA-DRlo S100Ahi monocytes, neutrophils). Retrieve TF activities with getA.gedi/getADB.gedi (DoRothEA-derived prior), compute the SPI1 activity gradient with svd.activityGradient.gedi and svd.joint_vectorField_gradient.gedi in monocytes, and confirm SPI1 mRNA up-regulation with getDiffExp.gedi.

## Related tools

- **GEDI** (Fits the multi-sample generative model and provides svd.vectorField.gedi, plot_vectorField, getA.gedi, getADB.gedi, svd.activityGradient.gedi, svd.joint_vectorField_gradient.gedi, getDiffExp.gedi, and model$plotTracking()) — https://github.com/csglab/GEDI
- **Harmony** (Provides the integrated space for cohort 1 samples that GEDI's sample-specific transformations approximate)
- **uwot** (Used for embedding/visualization support in the notebook workflow) — https://cran.r-project.org/web/packages/uwot/index.html
- **R** (Implementation environment (scuttle/scran/scater for preprocessing; Rcpp/RcppEigen, ggplot2, metR, rsvd, scales as GEDI dependencies))

## Examples

```
devtools::install_github("csglab/GEDI"); vf <- svd.vectorField.gedi(model, condition="severe", reference="healthy"); plot_vectorField(vf, embedding, color_by=celltype)
```

## Evaluation signals

- model$plotTracking() shows convergence of the o, Z, Bi, Qi, oi and si components
- Cell-type ranking by mean vector magnitude reproduces the reported Fig. 3d ordering: plasmablasts first, followed by HLA-DRlo S100Ahi monocytes and neutrophils
- SPI1 activity gradient direction aligns with the severe-vs-healthy vector field in HLA-DRlo S100Ahi monocytes
- SPI1 mRNA is significantly up-regulated by getDiffExp.gedi
- High concordance of activity gradients for SPI1, CEBPA, and SP1 with the vector field in monocytes

## Limitations

- Counterfactual/perturbation prediction in unobserved cell types via the vector field remains untested
- GEDI currently supports only linear manifold learning (optionally restricted to a hyperellipsoid); other parametric manifold approximations are future extensions
- It remains untested whether GEDI's gene-level expression imputations are competitive with existing single-cell imputation methods
- Performance depends on factors (number of DE genes per cell, DE magnitude, sample/cell counts, sequencing depth) that need more extensive analysis

## Evidence

- [discussion] Vector field computation via GEDI SVD functions: [evidence span withheld — anchor status: not_found]
- [discussion] Harmony-integrated space for cohort 1: "we used the integrated space identified by Harmony41 for the samples in cohort 1 of the COVID-19 dataset"
- [discussion] Vector field of mild COVID-19 vs healthy: [evidence span withheld — anchor status: not_found]
- [readme] Notebook for Fig. 3 vector field analysis: [evidence span withheld — anchor status: external_doc]
- [readme] GEDI installation: [evidence span withheld — anchor status: external_doc]
