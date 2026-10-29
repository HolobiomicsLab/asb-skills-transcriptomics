---
name: cell-type-balanced-subset-allocation
description: Use when a large scRNA-seq AnnData (e.g. a 428k-cell gut atlas in .h5ad)
  must be subsampled to a fixed sketch size while preserving cell-type diversity and
  rare populations for downstream annotation/ML tasks.
when_to_use_negative:
- The AnnData has no cell type labels and no PCA representation (scValue fits the
  random forest on features from adata.obsm['X_pca'] by default and needs cell_type_key).
- The goal is geometry/distribution-preserving sketching via distance optimization
  (GeoSketch, Sphetcher, Hopper, scSampler maximize distance or structure rather than
  OOB data value) rather than value-based, cell-type-balanced selection.
- The input is a small dataset where subsampling is unnecessary — scValue targets
  datasets scaling from tens of thousands to millions of cells.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scValue
  role: 'Core method: computes per-cell OOB data value via random forest, allocates
    sketch size across cell types, and performs DV-guided cell selection; open-source
    Python package installable via pip.'
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: 'Preprocessing: highly variable gene selection (e.g. 3000 HVGs) and PCA into
    adata.obsm[''X_pca''] that scValue consumes via use_rep=''X_pca''; also used to
    implement Uniform baseline subsampling.'
- name: scikit-learn 1.5.2
  role: Random forest implementation underlying OOB-based per-cell data value computation.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:42234ad79e1fd0ceb86395436aa35e02b61f2aac133f6be3483a1f5f18f97e05
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
  - GeoSketch
  - Hopper
  - Scanpy
  - Seurat
  - Sphetcher
  - scSampler
  - scValue
  - scikit-learn
  - scikit-learn 1.5.2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cell-type-balanced-subset-allocation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Produces cell-type-balanced sketches of large scRNA-seq AnnData objects by scoring each cell with a random-forest out-of-bag (OOB) 'data value' (DV), allocating sketch size across cell types with DV-weighted weighting, and selecting cells via DV-guided binning. Used to make datasets like the 428k-cell human gut reference tractable for downstream ML tasks such as CellTypist label transfer.

## When to use

Use when a large scRNA-seq AnnData (e.g. a 428k-cell gut atlas in .h5ad) must be subsampled to a fixed sketch size while preserving cell-type diversity and rare populations for downstream annotation/ML tasks.

## When NOT to use

- The AnnData has no cell type labels and no PCA representation (scValue fits the random forest on features from adata.obsm['X_pca'] by default and needs cell_type_key).
- The goal is geometry/distribution-preserving sketching via distance optimization (GeoSketch, Sphetcher, Hopper, scSampler maximize distance or structure rather than OOB data value) rather than value-based, cell-type-balanced selection.
- The input is a small dataset where subsampling is unnecessary — scValue targets datasets scaling from tens of thousands to millions of cells.

## Inputs

- Log-normalised scRNA-seq AnnData (.h5ad) with cell type labels in adata.obs and a PCA representation in adata.obsm['X_pca'] (or dense/sparse adata.X via use_rep='X')
- sketch_size: float fraction in (0,1] of cells to keep, or an integer cell count
- cell_type_key: column name in adata.obs holding cell type labels (default 'cell_type')

## Outputs

- Subsampled AnnData sketch (scv.adata_sub) with per-cell data value in adata_sub.obs['dv']
- dv_values.csv (per-cell DV + cell type) and dv_summary.csv (per-type DV summary statistics) when write_dv=True
- scv.dv DataFrame of cell type + DV for the full dataset

## How to apply

Preprocess the AnnData with Scanpy (select top highly variable genes, e.g. 3000 HVGs, then sc.pp.pca) so a PCA representation is stored in adata.obsm['X_pca']. Instantiate SCValue with sketch_size (float fraction in (0,1] or an integer cell count), use_rep='X_pca', cell_type_key (default 'cell_type'), n_trees=100, type_weighting='inv_cv' (weights ∝ mean(DV)*sqrt(N)/std(DV)), strategy='FB', and seed=42, then call scv.value() to obtain the subsampled AnnData with DV stored in adata.obs['dv']. The default FB (full binning) strategy bins DV into 0.1 intervals from 0.0 to 1.0 and selects top-DV cells within each bin, ensuring even low-OOB cells are represented; MTB restricts to bins above the type mean DV, and TP simply selects the highest-DV cells per type — choose FB when preserving overall diversity matters, MTB/TP when favoring informative cells. Sketch size per cell type is determined by both abundance and variability in data values; switch type_weighting to 'proportional' to mirror original cell-type frequencies instead of DV weighting. Validate by checking the requested sketch size was hit exactly, DVs landed in obs['dv'], cell-type balance via the Gini coefficient, and downstream agreement (e.g. CellTypist label-transfer agreement vs the full-reference transfer).

## Related tools

- **scValue** (Core method: computes per-cell OOB data value via random forest, allocates sketch size across cell types, and performs DV-guided cell selection; open-source Python package installable via pip.) — https://github.com/LHBCB/scvalue
- **Scanpy** (Preprocessing: highly variable gene selection (e.g. 3000 HVGs) and PCA into adata.obsm['X_pca'] that scValue consumes via use_rep='X_pca'; also used to implement Uniform baseline subsampling.)
- **scikit-learn 1.5.2** (Random forest implementation underlying OOB-based per-cell data value computation.)

## Examples

```
scv = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', seed=42); adata_sub = scv.value()
```

## Evaluation signals

- adata_sub.obs['dv'] exists for every selected cell and scv.dv covers the full dataset with cell type + DV columns.
- The sketch contains exactly the requested number of cells (integer sketch_size) or fraction (e.g. 0.1), and sketch sizes per type reflect DV-weighted allocation (inv_cv default).
- Cell-type balance in the sketch assessed via the Gini coefficient matches the paper's criterion of well-balanced cell-type representation.
- Downstream check: a CellTypist model trained on the sketch yields transferred annotations closest to the full-reference transfer among subsampling methods (scValue and Uniform agreement, per Figure 4).
- Reproducibility check: seed=42 and n_trees=100 reproduce the paper's scValue runs.

## Limitations

- OOB-based DV estimates depend on random forest hyperparameters; the article notes tuning (increasing n_trees, adjusting maximum depth) may be needed to stabilize OOB estimates in noisy conditions.
- MTB and TP strategies favor higher-DV cells and can exclude low-value cells; only FB bins the full [0,1] DV range so cells with lower OOB values are still included.
- If use_rep='X', sparse matrices may be densified internally, raising memory use; the default expects adata.obsm['X_pca'].
- The scValue-core confidence-score threshold used to filter cells is user-defined with no default value provided.
- No accession numbers or repository identifiers are given for the 16 benchmark datasets in the discussion, hindering direct data reuse.

## Evidence

- [readme] scValue assigns each cell a **data value (DV)** using **out-of-bag (OOB)** estimates from a random forest classifier.: "scValue assigns each cell a **data value (DV)** using **out-of-bag (OOB)** estimates from a random forest classifier."
