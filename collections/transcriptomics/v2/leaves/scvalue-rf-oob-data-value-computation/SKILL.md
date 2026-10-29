---
name: scvalue-rf-oob-data-value-computation
description: Use when a large scRNA-seq AnnData (.h5ad) with cell-type labels must
  be subsampled for a downstream ML/DL task (annotation, label transfer, deconvolution)
  and cell-level informativeness should guide which cells enter the sketch.
when_to_use_negative:
- The dataset is small enough that no subsampling is needed, or the analysis goal
  is uniform/distribution-preserving sketching (GeoSketch, Sphetcher, Hopper, scSampler)
  rather than value-based selection for ML/DL tasks
- Input lacks cell-type labels in .obs, since DV computation is supervised (random
  forest OOB estimates per cell type)
- The input is already a reduced feature table or the target task is not a learning
  task (e.g. pure visualization of the full atlas)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scValue
  role: Python package that computes per-cell OOB-based data value and builds the
    DV-guided subsample; installable via pip
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: 'Preprocessing: highly variable gene selection and PCA (sc.pp.highly_variable_genes,
    sc.pp.pca) to produce adata.obsm[''X_pca''] used as the RF feature matrix'
- name: scikit-learn 1.5.2
  role: Random forest implementation underlying the OOB data-value estimates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:6420a6ffb58f0147b3a5fa9b458af7ce72cca3fd88d738fbf0843aeb6d61713e
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# scvalue-rf-oob-data-value-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute a per-cell 'data value' (DV) score for an scRNA-seq dataset using out-of-bag (OOB) estimates from a random forest classifier, via the scValue Python package. Higher-DV cells are more helpful for distinguishing their cell type and are prioritized during subsampling (sketching).

## When to use

Use when a large scRNA-seq AnnData (.h5ad) with cell-type labels must be subsampled for a downstream ML/DL task (annotation, label transfer, deconvolution) and cell-level informativeness should guide which cells enter the sketch.

## When NOT to use

- The dataset is small enough that no subsampling is needed, or the analysis goal is uniform/distribution-preserving sketching (GeoSketch, Sphetcher, Hopper, scSampler) rather than value-based selection for ML/DL tasks
- Input lacks cell-type labels in .obs, since DV computation is supervised (random forest OOB estimates per cell type)
- The input is already a reduced feature table or the target task is not a learning task (e.g. pure visualization of the full atlas)

## Inputs

- Log-normalized scRNA-seq AnnData (.h5ad) with a cell-type label column in .obs (default key 'cell_type')
- PCA representation stored in adata.obsm['X_pca'] computed on top HVGs (e.g. 3,000 HVGs, 50 PCs), or use_rep='X' to use adata.X directly
- sketch_size parameter: fraction of cells in (0, 1] or an integer cell count

## Outputs

- Subsampled AnnData (scv.adata_sub) containing the sketch, with per-cell DV in .obs['dv']
- Per-cell DV stored in the original adata.obs['dv']
- DataFrame of cell type + DV for the full dataset (scv.dv)
- Optional CSV files when write_dv=True: dv_values.csv (per-cell DV + type) and dv_summary.csv (per-type DV summary statistics)

## How to apply

Load the log-normalized .h5ad with Scanpy, select highly variable genes (e.g. n_top_genes=3000) and run sc.pp.pca(n_comps=50) so features live in adata.obsm['X_pca'], which is the default use_rep. Instantiate SCValue with sketch_size (float fraction in (0,1] or integer cell count), cell_type_key, n_trees=100, type_weighting ('inv_cv' default, 'sqrt_n_sd', or 'proportional'), strategy ('FB' full binning default, 'MTB' mean-threshold binning, or 'TP' top-pick), write_dv, and seed=42, then call scv.value(). The random forest yields OOB prediction accuracy per cell as DV; sketch size is then allocated across cell types by the weighting scheme (inv_cv weights ∝ mean(DV)*sqrt(N)/std(DV)), and cells are picked within each type by the DV-guided selection strategy (FB bins DV into 0.1 intervals over [0,1] and selects top-DV cells per bin). Rationale: OOB-based informativeness selects cells that support stable decision boundaries, while FB binning ensures even lower-DV cells are included, preserving data diversity.

## Related tools

- **scValue** (Python package that computes per-cell OOB-based data value and builds the DV-guided subsample; installable via pip) — https://github.com/LHBCB/scvalue
- **Scanpy** (Preprocessing: highly variable gene selection and PCA (sc.pp.highly_variable_genes, sc.pp.pca) to produce adata.obsm['X_pca'] used as the RF feature matrix)
- **scikit-learn 1.5.2** (Random forest implementation underlying the OOB data-value estimates)

## Examples

```
from scvalue import SCValue; scv = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, type_weighting='inv_cv', strategy='FB', write_dv=False, seed=42); adata_sub = scv.value()
```

## Evaluation signals

- DV values are written to adata.obs['dv'] and to adata_sub.obs['dv']; with write_dv=True, dv_values.csv contains per-cell DV + type and dv_summary.csv contains per-type DV summary statistics
- The sketch size matches sketch_size exactly: either the requested fraction of cells (float in (0,1]) or the exact integer cell count
- With strategy='FB' (default), DV is binned into 0.1 intervals from 0.0 to 1.0 and top-DV cells are selected within each bin, so cells with lower OOB values appear in the subsample
- The subsampled AnnData should preserve cell-type diversity and balanced cell-type proportions; in the paper's label-transfer benchmark (CellTypist on the 428k-cell Gut reference), scValue sketches produced transferred annotations closest to the full-reference transfer
- Reproducibility check: re-running with seed=42 and the same parameters (n_trees=100, use_rep='X_pca', type_weighting='inv_cv', strategy='FB') yields the same sketch

## Limitations

- OOB estimates may be unstable in noisy conditions; the article suggests tuning random forest hyperparameters (increasing the number of trees, adjusting maximum depth) to stabilize OOB estimates
- use_rep='X' may trigger internal densification of sparse matrices (memory cost); scValue expects features from adata.obsm['X_pca'] by default
- The scValue-core confidence score threshold for filtering cells is user-defined with no default given; tool versions for benchmarked comparison methods were not specified in the paper
- Data value computation is supervised and depends on the quality of the cell_type_key annotations

## Evidence
