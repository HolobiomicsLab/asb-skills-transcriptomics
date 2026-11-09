---
name: cell-type-annotation-model-training
description: Use when benchmarking how subsampling/sketching (e.g. scValue sketches
  at 0.02–0.10 fractions) affects supervised cell-type annotation accuracy on .h5ad
  scRNA-seq datasets with known ground-truth labels.
when_to_use_negative:
- The dataset has no original cell-type annotations, so predicted labels cannot be
  validated against ground truth
- The goal is bulk RNA-seq deconvolution or cross-study label transfer rather than
  supervised annotation accuracy benchmarking
- Input features are raw counts without Scanpy preprocessing (HVG + PCA), which scValue
  and the annotation models expect
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scValue
  role: Generates value-based sketches (OOB random-forest data values, inv_cv weighting,
    FB strategy) on which annotation models are trained and evaluated
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Loads .h5ad files, selects highly variable genes, computes PCA stored in adata.obsm['X_pca']
    as model input features
- name: scANVI
  role: Deep-learning annotation model designated for the PBMC task (10% sketch accuracy
    0.8330 vs 0.8635 full reference)
- name: scPoli
  role: Annotation model designated for the mBrain task
- name: CellTypist
  role: Annotation model designated for the CxG_min task; also assigns confidence
    scores to validate predicted cell types against original annotations
- name: ACTINN
  role: Annotation model designated for the mACA task
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:5b6beecb97bcce899fd72b1c418be16edb75e958f5ec70257f661514a679225d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/LHBCB/scvalue
  repo_url: https://github.com/LHBCB/scvalue
  tools:
  - ACTINN
  - CellTypist
  - GeoSketch
  - Scanpy
  - Sphetcher
  - TreeHopper
  - scANVI
  - scPoli
  - scSampler
  - scValue
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cell-type-annotation-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains/runs dataset-designated automatic cell-type annotation models (scANVI, scPoli, CellTypist, ACTINN) on scRNA-seq references — full data or scValue/baseline sketches at 2%–10% fractions — and scores predicted labels against original annotations to benchmark subsampling impact on downstream ML/DL tasks.

## When to use

Use when benchmarking how subsampling/sketching (e.g. scValue sketches at 0.02–0.10 fractions) affects supervised cell-type annotation accuracy on .h5ad scRNA-seq datasets with known ground-truth labels.

## When NOT to use

- The dataset has no original cell-type annotations, so predicted labels cannot be validated against ground truth
- The goal is bulk RNA-seq deconvolution or cross-study label transfer rather than supervised annotation accuracy benchmarking
- Input features are raw counts without Scanpy preprocessing (HVG + PCA), which scValue and the annotation models expect

## Inputs

- Annotated scRNA-seq datasets as .h5ad files (PBMC, mBrain, CxG_min, mACA) with cell-type labels in adata.obs['cell_type']
- Scanpy-preprocessed features: highly variable gene selection plus PCA stored in adata.obsm['X_pca']
- Subsampled AnnData sketches at fractions 0.02–0.10 from scValue and six baseline methods

## Outputs

- Predicted cell-type labels per cell from scANVI, scPoli, CellTypist, or ACTINN
- Overall and per-cell-type annotation accuracy tables for every method-by-fraction combination
- Per-task boxplots of accuracy vs sketch fraction, with rare cell types flagged

## How to apply

Load each annotated .h5ad dataset (PBMC, mBrain, CxG_min, mACA), preprocess with Scanpy by selecting highly variable genes and running PCA so embeddings are stored in adata.obsm['X_pca'], since scValue defaults to use_rep='X_pca'. Generate sketches with SCValue (sketch_size=0.02–0.10, cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42) plus matched-size baselines (Uniform, GeoSketch, Sphetcher, TreeHopper, KH, scSampler). Train each dataset's designated model on every sketch — scANVI on PBMC, scPoli on mBrain, CellTypist on CxG_min, ACTINN on mACA — and score predicted labels against the original annotations, computing overall and per-cell-type accuracies and flagging rare cell types (assessable via Gini coefficient). Aggregate results into a table and per-task boxplots of accuracy vs sketch fraction; rationale is that sketches preserving decision boundaries should closely match full-data performance (PBMC/scANVI: 0.8330 at 10% sketch vs 0.8635 full reference) and outperform baselines consistently.

## Related tools

- **scValue** (Generates value-based sketches (OOB random-forest data values, inv_cv weighting, FB strategy) on which annotation models are trained and evaluated) — https://github.com/LHBCB/scvalue
- **Scanpy** (Loads .h5ad files, selects highly variable genes, computes PCA stored in adata.obsm['X_pca'] as model input features)
- **scANVI** (Deep-learning annotation model designated for the PBMC task (10% sketch accuracy 0.8330 vs 0.8635 full reference))
- **scPoli** (Annotation model designated for the mBrain task)
- **CellTypist** (Annotation model designated for the CxG_min task; also assigns confidence scores to validate predicted cell types against original annotations)
- **ACTINN** (Annotation model designated for the mACA task)

## Examples

```
adata = sc.read_h5ad('PBMC.h5ad'); sc.pp.highly_variable_genes(adata, n_top_genes=3000); sc.pp.pca(adata, n_comps=50)
```

## Evaluation signals

- Reproduced sketch accuracies closely match full-data performance (e.g. PBMC/scANVI at 10% sketch ≈ 0.8330 vs 0.8635 with full reference)
- scValue-trained model accuracies consistently exceed the six baseline subsampling methods across all four dataset-model pairs and fractions 0.02–0.10
- Per-cell-type accuracy breakdown is reported with rare cell types flagged, not just overall accuracy
- Accuracy-vs-fraction boxplots show less variability for scValue than for counterpart methods
- CellTypist-style validation: predicted cell types checked against original annotations, with confidence-score filtering below a user-defined threshold

## Limitations

- Tool versions for benchmarked annotation software (scANVI, scPoli, CellTypist, ACTINN) were not specified in the article
- No accession numbers or repository identifiers were provided for the benchmark datasets (PBMC, mBrain, CxG_min, mACA)
- Rare cell types are hard to capture; assessing their capture requires the Gini coefficient, and scValue-core's confidence-score filtering threshold is user-defined with no default given
- scValue OOB data-value estimates can be noisy; tuning random forest hyperparameters (e.g. increasing n_trees, adjusting max depth) may be needed for stability

## Evidence

- [discussion] In CTA tasks (PBMC with scANVI, mBrain with scPoli, CxG_min with CellTypist, and mACA with ACTINN): "In CTA tasks (PBMC with scANVI, mBrain with scPoli, CxG_min with CellTypist, and mACA with ACTINN)"
- [abstract] Our method consistently outperformed existing subsampling methods, closely matching full-data performance across all annotation tasks: "Our method consistently outperformed existing subsampling methods, closely matching full-data performance across all annotation tasks"
- [readme] By default, scValue expects features from adata.obsm["X_pca"] (use_rep="X_pca").: "By default, scValue expects features from `adata.obsm["X_pca"]` (`use_rep="X_pca"`)."
- [discussion] each cell is assigned a confidence score using an established CellTypist model, its predicted cell type is validated against the original annotation: "each cell is assigned a confidence score using an established CellTypist model, its predicted cell type is validated against the original annotation"
