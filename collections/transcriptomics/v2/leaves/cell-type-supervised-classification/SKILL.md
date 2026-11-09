---
name: cell-type-supervised-classification
description: Use when a labeled .h5ad scRNA-seq dataset needs per-cell informativeness
  scores (data values) for value-based subsampling, quality assessment, or identifying
  ambiguous/atypical cells relative to their annotated cell types.
when_to_use_negative:
- The dataset has no cell-type labels — the random forest is supervised by adata.obs
  cell-type annotations and cannot compute OOB type-prediction accuracy without them
- The goal is distance- or distribution-preserving sketching (e.g. GeoSketch, Sphetcher,
  Hopper, scSampler, KH) rather than OOB data-value-guided subsampling
- The input is raw counts without cell-type annotation or a computed feature representation
  like X_pca
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scValue
  role: 'Implements the DV computation pipeline step 1: fits the random forest and
    derives per-cell OOB data values; available as an open-source Python package installable
    via pip'
  repo: https://github.com/LHBCB/scvalue
- name: scikit-learn 1.5.2
  role: Provides the RandomForestClassifier (n_estimators=100, seed=42) whose out-of-bag
    predictions yield per-cell data values
- name: Scanpy
  role: Loads the .h5ad AnnData, computes HVGs (n_top_genes=3000) and PCA stored in
    obsm['X_pca'] as classifier features
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:8c1f435e2118274490c3b9ae2307b5b2ff122f858296e2268b04be9888cf560f
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
  - Scanpy
  - scValue
  - scikit-learn
  - scikit-learn 1.5.2
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cell-type-supervised-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a supervised random-forest classifier on cell-type-labeled scRNA-seq features (PCA on top 3,000 HVGs) and scores each cell's 'data value' as its out-of-bag prediction accuracy, quantifying how well the cell's type is distinguished by the rest of the data.

## When to use

Use when a labeled .h5ad scRNA-seq dataset needs per-cell informativeness scores (data values) for value-based subsampling, quality assessment, or identifying ambiguous/atypical cells relative to their annotated cell types.

## When NOT to use

- The dataset has no cell-type labels — the random forest is supervised by adata.obs cell-type annotations and cannot compute OOB type-prediction accuracy without them
- The goal is distance- or distribution-preserving sketching (e.g. GeoSketch, Sphetcher, Hopper, scSampler, KH) rather than OOB data-value-guided subsampling
- The input is raw counts without cell-type annotation or a computed feature representation like X_pca

## Inputs

- Cell-typed scRNA-seq dataset in .h5ad format (AnnData object)
- adata.obsm['X_pca'] PCA representation computed on top 3,000 HVGs (or adata.X with use_rep='X')
- adata.obs['cell_type'] cell-type labels (column configurable via cell_type_key)

## Outputs

- Per-cell data value (DV) score v_i in [0,1] stored in adata.obs['dv']
- dv_values.csv: per-cell DV scores with cell-type labels
- dv_summary.csv: per-cell-type DV summary statistics

## How to apply

Load the cell-typed .h5ad with Scanpy, compute PCA on the top 3,000 highly variable genes, and take adata.obsm['X_pca'] as the feature matrix X with adata.obs['cell_type'] as labels Y. Fit a scikit-learn RandomForestClassifier with n_estimators=100 trees (scValue default n_trees=100) and a fixed seed (seed=42) on (X, Y). For each cell, extract the out-of-bag (OOB) decision function and, per Equation (1), score the cell's data value v_i as its OOB cell-type prediction accuracy, yielding v_i in [0,1]. Assemble per-cell DV scores with cell-type labels into a DataFrame and optionally write dv_values.csv (per-cell) and dv_summary.csv (per-type summary); noisy conditions may require tuning RF hyperparameters (more trees, adjusted max_depth) to stabilize OOB estimates.

## Related tools

- **scValue** (Implements the DV computation pipeline step 1: fits the random forest and derives per-cell OOB data values; available as an open-source Python package installable via pip) — https://github.com/LHBCB/scvalue
- **scikit-learn 1.5.2** (Provides the RandomForestClassifier (n_estimators=100, seed=42) whose out-of-bag predictions yield per-cell data values)
- **Scanpy** (Loads the .h5ad AnnData, computes HVGs (n_top_genes=3000) and PCA stored in obsm['X_pca'] as classifier features)

## Examples

```
import scanpy as sc; from scvalue import SCValue; adata = sc.read_h5ad('mTC.h5ad'); sc.pp.highly_variable_genes(adata, n_top_genes=3000); sc.pp.pca(adata, n_comps=50); scv = SCValue(adata=adata, sketch_size=0.1, n_trees=100, seed=42, write_dv=True); adata_sub = scv.value()
```

## Evaluation signals

- All v_i values fall in [0,1], being averaged OOB prediction correctness per cell
- Ambiguous or atypical cells receive lower DV values than cells whose types are well distinguished by the ensemble
- With write_dv=True, dv_values.csv contains one row per cell with cell type + DV, and dv_summary.csv contains per-type summary statistics
- Fitting uses B=100 trees (scValue default n_trees=100) and seed=42 for reproducibility
- Feature matrix X corresponds to adata.obsm['X_pca'] (PCA on top 3,000 HVGs), matching the paper's setup

## Limitations

- In noisy conditions, OOB estimates may be unstable; tuning random forest hyperparameters (increasing the number of trees, adjusting maximum depth) may be needed to stabilize them
- With use_rep='X', sparse matrices may be densified internally, which can be memory-heavy
- OOB accuracy reflects how well a cell's type is distinguished given current labels and representation; mislabeled cells may receive misleadingly low values

## Evidence

- [other] Each cell's data value is its out-of-bag (OOB) accuracy: how accurately the ensemble of random forest trees, trained on all other (in-bag) cells, predicts that cell's type (Equation 1).: [evidence span withheld — anchor status: not_found]
