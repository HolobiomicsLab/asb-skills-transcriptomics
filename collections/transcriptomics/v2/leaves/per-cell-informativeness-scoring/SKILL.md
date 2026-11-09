---
name: per-cell-informativeness-scoring
description: Use when a cell-typed .h5ad single-cell dataset is too large for ML/DL
  training and you need a per-cell importance score before sketching, or want to rank
  ambiguous/atypical cells by OOB accuracy.
when_to_use_negative:
- Cells lack cell-type labels — the OOB accuracy score requires supervised labels
  Y
- The goal is distance- or distribution-preserving subsampling without labels (e.g.
  GeoSketch, Sphetcher, Hopper, scSampler designs) — DV ranking is supervised and
  OOB-based
- You already have a feature/importance table and need only type-level size allocation
  or within-type selection (steps 2–3 of the scValue pipeline), not per-cell scoring
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scValue
  role: Python package whose SCValue class computes per-cell DV via OOB estimates
    from a random forest (n_trees=100, seed=42) and exposes them in adata.obs['dv']
  repo: https://github.com/LHBCB/scvalue
- name: scikit-learn 1.5.2
  role: RandomForestClassifier used to fit the B-tree forest on (X, Y) and produce
    OOB prediction correctness per cell
- name: Scanpy
  role: Reads the .h5ad input, selects the top 3,000 highly variable genes, computes
    PCA into adata.obsm['X_pca'] used as the forest feature matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:4244df6f9c1e597508b649b4734649095d0f2bc26ff78cc4731c40c76ef7a7e7
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# per-cell-informativeness-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores every cell in a labeled scRNA-seq dataset with a 'data value' (DV) equal to its out-of-bag (OOB) prediction accuracy from a random forest, quantifying per-cell informativeness for downstream subsampling or filtering.

## When to use

Use when a cell-typed .h5ad single-cell dataset is too large for ML/DL training and you need a per-cell importance score before sketching, or want to rank ambiguous/atypical cells by OOB accuracy.

## When NOT to use

- Cells lack cell-type labels — the OOB accuracy score requires supervised labels Y
- The goal is distance- or distribution-preserving subsampling without labels (e.g. GeoSketch, Sphetcher, Hopper, scSampler designs) — DV ranking is supervised and OOB-based
- You already have a feature/importance table and need only type-level size allocation or within-type selection (steps 2–3 of the scValue pipeline), not per-cell scoring

## Inputs

- Cell-typed scRNA-seq dataset in .h5ad format (AnnData, log-normalized)
- Feature matrix X: adata.obsm['X_pca'] from PCA on the top 3,000 highly variable genes (or adata.X if use_rep='X')
- Cell-type labels Y: adata.obs['cell_type']

## Outputs

- Per-cell data value (DV) scores v_i in [0,1], stored in adata.obs['dv']
- dv_values.csv: per-cell DV scores with cell-type labels
- dv_summary.csv: per-type DV summary statistics (when write_dv=True)

## How to apply

Load the cell-typed dataset with Scanpy, compute PCA on the top 3,000 highly variable genes, and take adata.obsm['X_pca'] as the feature matrix X with adata.obs['cell_type'] as labels Y. Fit a scikit-learn RandomForestClassifier with n_estimators=100 trees (scValue default n_trees=100) and seed=42 on (X, Y). Per Equation (1), score each cell's DV v_i in [0,1] as the average OOB prediction correctness of the ensemble of trees that did not train on that cell; ambiguous or atypical cells receive lower values. Assemble per-cell DV scores with cell-type labels into a DataFrame (e.g. dv_values.csv); increasing trees or adjusting maximum depth can stabilize OOB estimates in noisy conditions.

## Related tools

- **scValue** (Python package whose SCValue class computes per-cell DV via OOB estimates from a random forest (n_trees=100, seed=42) and exposes them in adata.obs['dv']) — https://github.com/LHBCB/scvalue
- **scikit-learn 1.5.2** (RandomForestClassifier used to fit the B-tree forest on (X, Y) and produce OOB prediction correctness per cell)
- **Scanpy** (Reads the .h5ad input, selects the top 3,000 highly variable genes, computes PCA into adata.obsm['X_pca'] used as the forest feature matrix)

## Examples

```
from scvalue import SCValue; scv = SCValue(adata=adata, sketch_size=0.1, use_rep="X_pca", cell_type_key="cell_type", n_trees=100, seed=42, write_dv=True); scv.value()
```

## Evaluation signals

- Every DV score lies in [0,1] since v_i is averaged OOB prediction correctness per cell
- Length of the DV vector equals the number of cells in adata; dv_values.csv rows pair each cell with a cell-type label
- Re-running with the same seed=42 and n_trees=100 reproduces the same DV values (fixed random seed)
- Ambiguous or atypical cells should receive systematically lower DVs than cells that support stable decision boundaries
- Per-type DV summary (dv_summary.csv) shows type-level mean/std consistent with the full-dataset dv DataFrame

## Limitations

- OOB estimates are stochastic; in noisy conditions, tuning random forest hyperparameters (increasing number of trees, adjusting maximum depth) may be needed to stabilize the DV scores
- DV depends on the chosen feature representation — scValue expects adata.obsm['X_pca'] (use_rep='X_pca') by default, and use_rep='X' may densify sparse matrices internally
- Cells with low DV are not necessarily uninformative — FB binning exists specifically to ensure even cells with lower OOB values are included in the final subsample

## Evidence

- [methods] Each cell's data value is its out-of-bag (OOB) accuracy: how accurately the ensemble of random forest trees, trained on all other (in-bag) cells, predicts that cell's type (Equation 1).: [evidence span withheld — anchor status: not_found]
