---
name: cell-type-stratified-accuracy-comparison
description: Use when you have an integration-adjusted expression matrix plus raw
  logcounts, consensus cell-type labels and per-sample severity metadata, and need
  to quantify whether integration improves cell-type-stratified prediction accuracy.
when_to_use_negative:
- Your integration method returns only a low-dimensional embedding with no adjusted
  expression matrix — the article notes such embeddings restrict sample-level analysis
  and interpretable downstream features
- You want cell-type annotation or clustering quality rather than sample-level prediction
  performance from pseudo-bulk signatures
- Data lack per-sample condition/severity labels, so pseudo-replication and class-prediction
  evaluation cannot be performed
tools:
- name: scMerge2
  role: Produces the adjusted expression matrix and consensus cell-type map from the
    multi-sample multi-condition COVID-19 collection, used to build pseudo-bulk signatures
    for the adjusted arm of the comparison
  repo: https://github.com/SydneyBioX/scMerge
- name: Seurat
  role: Used for single-cell data handling and per-cell-type/per-sample pseudo-bulk
    aggregation of the adjusted and raw matrices
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:22470ddc612127a8c27d49740d2c2844888ce3d0b8b0fc1873a8340cef35523d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - Seurat
  - scMerge2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cell-type-stratified-accuracy-comparison

## Summary

Evaluates whether scMerge2-adjusted single-cell expression improves downstream prediction compared with raw log-normalised data, by building per-cell-type pseudo-bulk signatures, training RBF-kernel SVMs to predict COVID-19 disease severity (Healthy / Mild-Moderate / Severe-Critical), and comparing per-cell-type cross-validation accuracies.

## When to use

Use when you have an integration-adjusted expression matrix plus raw logcounts, consensus cell-type labels and per-sample severity metadata, and need to quantify whether integration improves cell-type-stratified prediction accuracy.

## When NOT to use

- Your integration method returns only a low-dimensional embedding with no adjusted expression matrix — the article notes such embeddings restrict sample-level analysis and interpretable downstream features
- You want cell-type annotation or clustering quality rather than sample-level prediction performance from pseudo-bulk signatures
- Data lack per-sample condition/severity labels, so pseudo-replication and class-prediction evaluation cannot be performed

## Inputs

- scMerge2-adjusted COVID-19 cell-type-specific expression matrix
- Raw logcounts expression matrix (unadjusted baseline)
- Consensus cell-type labels per cell
- Per-sample disease severity metadata (Healthy / Mild-Moderate / Severe-Critical)

## Outputs

- Per-cell-type accuracy table for adjusted vs raw logcounts
- Comparison boxplot of accuracies per cell type
- Average accuracy delta between adjusted and raw baselines (e.g. 3.2% mean gain)

## How to apply

Load the scMerge2-adjusted COVID-19 expression matrix together with consensus cell-type labels and per-sample severity metadata. Aggregate expression within each cell type and sample to build sample-level pseudo-bulk signatures from the adjusted matrix, then train an RBF-kernel SVM to predict the three severity classes (Healthy / Mild-Moderate / Severe-Critical) with repeated fivefold cross-validation, recording accuracy per cell type. Repeat the identical pseudo-bulk aggregation and SVM cross-validation on the raw logcounts matrix to obtain the unadjusted baseline. Rationale: the paper argues that integration improves discrimination of distinct cell states in COVID-19 patients of varying severity, but only methods returning adjusted expression matrices enable this sample-level downstream analysis. Tabulate adjusted vs raw accuracies per cell type and render a comparison boxplot.

## Related tools

- **scMerge2** (Produces the adjusted expression matrix and consensus cell-type map from the multi-sample multi-condition COVID-19 collection, used to build pseudo-bulk signatures for the adjusted arm of the comparison) — https://github.com/SydneyBioX/scMerge
- **Seurat** (Used for single-cell data handling and per-cell-type/per-sample pseudo-bulk aggregation of the adjusted and raw matrices)

## Examples

```
BiocManager::install("scMerge"); library(scMerge); data(segList)
```

## Evaluation signals

- Adjusted pseudo-bulk SVM accuracies exceed raw logcounts baselines for all cell types above 1% abundance
- Average accuracy increase of ~3.2% across cell types relative to raw log-normalised data
- CD14 Monocyte accuracy rises from 81.3% (raw) to 83.6% (scMerge2-adjusted)
- Identical repeated fivefold cross-validation protocol applied to both adjusted and raw arms so the comparison is controlled
- Three severity classes (Healthy / Mild-Moderate / Severe-Critical) are consistently the prediction targets in both arms

## Limitations

- Only cell types above 1% abundance are evaluated; rare cell types are excluded from the accuracy comparison
- The evaluation depends on the integration returning an adjusted expression matrix — embedding-only integration outputs diminish downstream sample-level utility
- Multi-modal integration was only performed within each modality in the scMerge2 paper; joint multi-modal signal use is deferred to future work

## Evidence

- [abstract] Signatures derived from cell-type expression are more accurate in discriminating disease progression: "reveals signatures derived from cell-type expression that are more accurate in discriminating disease progression"
- [intro] Integration improves discrimination of severity states: "the integration using scMerge2 improves the performance of discriminating distinct cell states in COVID-19 patients with varying degrees of severity"
- [discussion] Pseudo-bulk building is one of the three key innovations: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [intro] Lack of adjusted expression matrices restricts sample-level analysis: "the lack of adjusted expression matrices restricts the utilisation of such integrative results and diminishes their potency and generalizability"
- [discussion] Embedding-only outputs hamper interpretable downstream features: "an embedding output by itself increases interpretability challenges since a low-dimensional representation does not naturally lend itself to the development of interpretable features"
- [readme] scMerge is installed from Bioconductor: [evidence span withheld — anchor status: not_found]
