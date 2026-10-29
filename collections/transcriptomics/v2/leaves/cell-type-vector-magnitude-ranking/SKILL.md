---
name: cell-type-vector-magnitude-ranking
description: Use when you have a GEDI model fitted to a multi-sample, condition-contrast
  scRNA-seq atlas (e.g., Harmony-integrated COVID-19 cohort 1) and need a cluster-free,
  cell-type-level ranking of response strength to a condition.
when_to_use_negative:
- The dataset has no sample-level condition contrast (no healthy-vs-severe or equivalent
  prior H), so a condition vector field is undefined
- You need per-gene differential expression rather than a cell-type-level response-magnitude
  summary — use GEDI's cluster-free DGE (getDiffExp.gedi) instead
- The embedding has not been produced by GEDI/Harmony; computing raw expression shifts
  on an unintegrated manifold conflates batch and condition effects, which the article
  explicitly warns against
tools:
- name: GEDI
  role: Provides svd.vectorField.gedi to compute the transcriptomic vector field and
    plot_vectorField for visualization; magnitudes are aggregated per cell type
  repo: https://github.com/csglab/GEDI
- name: Harmony
  role: Supplies the integrated embedding space of COVID-19 cohort 1 samples in which
    the vector field is computed
- name: R
  role: Implementation language for the GEDI workflow and magnitude aggregation/ranking
  repo: https://github.com/csglab/GEDI_manuscript
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:f70c3740ab4aebdcd06eea320ebd667505cb7d0730f3d7663e9eb4564341e28d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - Harmony
  - R
  - uwot
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Cell-type vector magnitude ranking

## Summary

A GEDI-based analysis step that aggregates per-cell transcriptomic vector field magnitudes (severe-vs-healthy COVID-19) by cell type and ranks cell types by mean magnitude to identify which populations carry the strongest disease-associated transcriptional shift.

## When to use

Use when you have a GEDI model fitted to a multi-sample, condition-contrast scRNA-seq atlas (e.g., Harmony-integrated COVID-19 cohort 1) and need a cluster-free, cell-type-level ranking of response strength to a condition.

## When NOT to use

- The dataset has no sample-level condition contrast (no healthy-vs-severe or equivalent prior H), so a condition vector field is undefined
- You need per-gene differential expression rather than a cell-type-level response-magnitude summary — use GEDI's cluster-free DGE (getDiffExp.gedi) instead
- The embedding has not been produced by GEDI/Harmony; computing raw expression shifts on an unintegrated manifold conflates batch and condition effects, which the article explicitly warns against

## Inputs

- Fitted GEDI model object (sample vector, cell-level raw count matrix, sample-level prior H with disease status: healthy vs severe)
- Harmony-integrated embedding of COVID-19 cohort 1 samples
- Per-cell cell-type annotations (e.g., HLA-DRlo S100Ahi monocytes, plasmablasts, neutrophils)

## Outputs

- Per-cell transcriptomic vector field (severe vs healthy) in the integrated embedding
- Cell-type-level table of mean vector magnitudes
- Ranking of cell types by mean vector magnitude (expected top: plasmablasts, HLA-DRlo S100Ahi monocytes, neutrophils)

## How to apply

Fit the GEDI model with the sample vector, raw count matrix, and a sample-level prior H encoding disease severity so that sample-specific manifold parameters (Qi) are informed by the condition contrast, and confirm convergence of o, Z, Bi, Qi, oi and si components with model$plotTracking(). Compute the severe-versus-healthy transcriptomic vector field in the Harmony-integrated embedding with svd.vectorField.gedi and inspect it with plot_vectorField colored by cell type. For each cell, take the vector magnitude (norm of the field vector), aggregate magnitudes per cell type, and rank cell types by mean magnitude. Validate against the reported Fig. 3d ordering for COVID-19 cohort 1: plasmablasts first, followed by HLA-DRlo S100Ahi monocytes and neutrophils. Optionally confirm biological interpretation by checking that TF activity gradients (e.g., SPI1 via svd.activityGradient.gedi and svd.joint_vectorField_gradient.gedi) align with the vector field direction in the top-ranked cell types.

## Related tools

- **GEDI** (Provides svd.vectorField.gedi to compute the transcriptomic vector field and plot_vectorField for visualization; magnitudes are aggregated per cell type) — https://github.com/csglab/GEDI
- **Harmony** (Supplies the integrated embedding space of COVID-19 cohort 1 samples in which the vector field is computed)
- **R** (Implementation language for the GEDI workflow and magnitude aggregation/ranking) — https://github.com/csglab/GEDI_manuscript

## Examples

```
vf <- svd.vectorField.gedi(gedi_model); mag <- sqrt(rowSums(vf^2)); rank(-tapply(mag, cell_types, mean))
```

## Evaluation signals

- model$plotTracking() shows convergence of o, Z, Bi, Qi, oi and si components before vector field computation
- The cell-type ranking reproduces the reported Fig. 3d ordering: plasmablasts first, followed by HLA-DRlo S100Ahi monocytes and neutrophils
- The SPI1 (and CEBPA, SP1) activity gradient direction aligns with the vector field in HLA-DRlo S100Ahi monocytes
- SPI1 mRNA is significantly up-regulated by getDiffExp.gedi in the top-ranked monocyte population
- Per-cell magnitudes are non-negative and finite; aggregation by cell type covers all annotated populations

## Limitations

- Counterfactual prediction in unobserved cell types (e.g., perturbation response prediction) via GEDI's vector field remains untested
- GEDI currently supports only linear manifold learning (with optional hyperellipsoid restriction), which constrains the vector field geometry
- The ranking depends on proper modeling of sample-to-sample variability; the article notes that mis-modeling sample-level variability biases differential analyses

## Evidence

- [discussion] we used the integrated space identified by Harmony41 for the samples in cohort 1 of the COVID-19 dataset: "we used the integrated space identified by Harmony41 for the samples in cohort 1 of the COVID-19 dataset"
- [discussion] calculate the transcriptomic vector field of mild COVID-19 relative to healthy controls: "calculate the transcriptomic vector field of mild COVID-19 relative to healthy controls"
- [intro] GEDI aims to deconvolve sample-to-sample variability and enable cluster-free DGE via the vector field: "uniquely capable of deconvolving the effects of multiple technical and/or biological sources of sample-to-sample variability."
- [readme] Fig. 3: Transcriptomic vector field analysis of COVID-19 data: [evidence span withheld — anchor status: external_doc]
- [discussion] Proper modeling of sample-to-sample variability is key for unbiased differential expression analysis: "Previous work has shown that properly modeling sample-to-sample variability is key for unbiased differential expression analysis"
