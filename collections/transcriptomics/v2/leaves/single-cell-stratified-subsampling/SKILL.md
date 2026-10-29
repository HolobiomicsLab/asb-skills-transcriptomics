---
name: single-cell-stratified-subsampling
description: Use when you need to reduce a large scRNA-seq AnnData (e.g. the 89,429-cell
  mTC mouse T cell .h5ad) to a per-cell-type sketch, and you must choose or reproduce
  a DV-guided within-type selection strategy (FB, MTB, or TP) given the number of
  cell types present.
when_to_use_negative:
- The dataset is already a small feature table or a non-single-cell matrix — this
  skill is scoped to scRNA-seq sketching with cell type labels and DV scores
- You need uniform random subsampling rather than value-guided selection — uniform
  subsampling (as in Seurat/Scanpy pipelines) is a separate, simpler baseline
- No cell type labels or DV scores exist and you cannot run the scValue OOB random-forest
  step first — the binning strategies operate on per-cell DV values within each cell
  type
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3173
tools:
- name: scValue
  role: Python package implementing DV computation and the FB/MTB/TP value-guided
    selection strategies; installable via pip
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Loads and preprocesses the input .h5ad (HVG selection, PCA into adata.obsm['X_pca'])
    and visualises the resulting sketch
- name: pandas
  role: Reads dv_values.csv and writes selected index sets such as selected_cells_omega_m.csv
- name: NumPy
  role: Implements the 0.1-width DV binning, per-bin quota allocation, and descending
    DV sorting within bins
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:96176008fb651ae57f5d02bc0ce3afe0f9c7f0b213b8cca50f35e211e19ede73
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
  - NumPy
  - Scanpy
  - pandas
  - scValue
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# single-cell-stratified-subsampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A DV-guided, per-cell-type subsampling step from scValue that selects high-value cells within each cell type using binning strategies (FB: full binning over [0,1] in 0.1-intervals; MTB: mean-threshold binning restricted to bins above the type mean DV) or top-pick (TP), producing a sketch that preserves data diversity while favoring higher-value cells.

## When to use

Use when you need to reduce a large scRNA-seq AnnData (e.g. the 89,429-cell mTC mouse T cell .h5ad) to a per-cell-type sketch, and you must choose or reproduce a DV-guided within-type selection strategy (FB, MTB, or TP) given the number of cell types present.

## When NOT to use

- The dataset is already a small feature table or a non-single-cell matrix — this skill is scoped to scRNA-seq sketching with cell type labels and DV scores
- You need uniform random subsampling rather than value-guided selection — uniform subsampling (as in Seurat/Scanpy pipelines) is a separate, simpler baseline
- No cell type labels or DV scores exist and you cannot run the scValue OOB random-forest step first — the binning strategies operate on per-cell DV values within each cell type

## Inputs

- AnnData .h5ad file (e.g. log-normalised mTC mouse T cell dataset) with cell type labels in adata.obs['cell_type']
- Per-cell data value scores, e.g. adata.obs['dv'] or dv_values.csv written by SCValue with write_dv=True
- Target sketch size S_m per cell type (float fraction like 0.1 or integer count), derived from type weighting ('inv_cv', 'sqrt_n_sd', or 'proportional')

## Outputs

- Selected cell index set Omega_m per cell type and per strategy, each of length S_m (e.g. selected_cells_omega_m.csv)
- Subsampled AnnData sketch (scv.adata_sub) with dv retained in adata_sub.obs['dv']
- Per-cell DV + type table (dv_values.csv) and per-type DV summary statistics (dv_summary.csv) when write_dv=True

## How to apply

After scValue computes per-cell data value (DV) via OOB estimates from a random forest (n_trees=100, seed=42) and allocates sketch sizes across cell types (e.g. 'inv_cv' weighting: mean(DV) * sqrt(N) / std(DV)), restrict to cell type m with N_m DV values and a fixed target size S_m. For Full binning (FB, the default), partition the DV range into 0.1-width bins over [0.0, 1.0] (K=10 bins), allocate per-bin quotas proportional to bin membership summing exactly to S_m, and pick the top-DV cells within each bin — this ensures even cells with lower OOB values are included. For Mean-threshold binning (MTB), use the same bins but consider only bins above the cell-type mean DV, merging/dropping below-mean bins and selecting top-DV cells up to S_m; MTB suits datasets with relatively few cell types and preserves a broad snapshot of variability. TP (top-pick) simply selects the highest-DV cells per type with no binning. Write the selected index set Omega_m (length must equal S_m) per strategy, e.g. to selected_cells_omega_m.csv.

## Related tools

- **scValue** (Python package implementing DV computation and the FB/MTB/TP value-guided selection strategies; installable via pip) — https://github.com/LHBCB/scvalue
- **Scanpy** (Loads and preprocesses the input .h5ad (HVG selection, PCA into adata.obsm['X_pca']) and visualises the resulting sketch)
- **pandas** (Reads dv_values.csv and writes selected index sets such as selected_cells_omega_m.csv)
- **NumPy** (Implements the 0.1-width DV binning, per-bin quota allocation, and descending DV sorting within bins)

## Examples

```
scv = SCValue(adata=adata, sketch_size=0.1, use_rep="X_pca", cell_type_key="cell_type", type_weighting="inv_cv", strategy="FB", seed=42); adata_sub = scv.value()
```

## Evaluation signals

- len(Omega_m) == S_m for each cell type and each strategy (FB, MTB, TP)
- FB selections span all ten 0.1-interval bins over [0.0, 1.0], including low-DV bins
- MTB selections are drawn only from bins above the cell-type mean DV
- Each cell type's quota allocation across bins sums exactly to S_m with no bin under/over-filled relative to its proportional membership
- Sketch cell-type proportions remain balanced and Gini coefficient / Hausdorff distance match values reported on the 16 benchmark datasets

## Limitations

- MTB restricts selection to bins above the type mean DV, so low-DV cells of that type are excluded; FB is the default precisely to ensure even cells with lower OOB values are included
- OOB-based DV estimates can be noisy in noisy conditions; tuning random forest hyperparameters (increasing n_trees, adjusting max depth) may be needed to stabilize them
- use_rep='X' on sparse adata.X may trigger internal densification, raising memory cost on million-cell datasets

## Evidence

- [other] The Full binning strategy selects cells across the entire data value ranges, preserving a broad snapshot of variability within each cell type in the sketch.: [evidence span withheld — anchor status: not_found]
- [readme] "strategy"
    - "FB" (full binning, default): DV is binned into 0.1 intervals from 0.0 to 1.0; selects top-DV cells within each bin.: "- `strategy`
    - `"FB"` (full binning, default): DV is binned into 0.1 intervals from 0.0 to 1.0; selects top-DV cells within each bin."
