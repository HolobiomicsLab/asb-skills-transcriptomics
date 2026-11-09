---
name: dv-threshold-bin-prioritization
description: Use when implementing or reproducing scValue's step-3 cell selection
  (FB/MTB binning) from per-cell DV scores in adata.obs['dv'] or dv_values.csv, given
  a per-type sketch quota S_m.
when_to_use_negative:
- 'The selection strategy is TP (top-pick): no binning is performed, so bin construction
  is unnecessary — simply select the highest-DV cells per type'
- The goal is sketch-size allocation across cell types (step 2, inv_cv/sqrt_n_sd/proportional
  weighting) rather than within-cell-type selection (step 3)
- DV scores are not yet computed — run the OOB random-forest DV computation (n_trees=100)
  first, since binning operates on adata.obs['dv']
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0203
tools:
- name: scValue
  role: Source of the DV-guided selection logic; its SCValue class exposes strategy='FB'
    (default, bins DV into 0.1 intervals) and strategy='MTB' (bins above the type
    mean DV only)
  repo: https://github.com/LHBCB/scvalue
- name: Scanpy
  role: Loads and preprocesses the input .h5ad (mTC dataset, top 3,000 HVGs, PCA into
    adata.obsm['X_pca']) whose obs['dv'] feeds the binning step
- name: pandas
  role: Reads dv_values.csv / adata.obs['dv'] per-cell DV scores and type labels,
    and writes selected_cells_omega_m.csv
- name: NumPy
  role: Implements the 0.1-width bin partition of [0,1], per-bin quota allocation,
    and descending DV sorts within bins
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:97e6998a18b4fd575ce50d7006f28afd6c7df3e3c587a13de7c10be62c4f3843
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# dv-threshold-bin-prioritization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements scValue's value-guided within-cell-type selection: Full binning (FB) over the [0,1] DV range and Mean-threshold binning (MTB), which restricts selection to bins above the cell-type mean DV, prioritizing high-value cells in a sketch.

## When to use

Use when implementing or reproducing scValue's step-3 cell selection (FB/MTB binning) from per-cell DV scores in adata.obs['dv'] or dv_values.csv, given a per-type sketch quota S_m.

## When NOT to use

- The selection strategy is TP (top-pick): no binning is performed, so bin construction is unnecessary — simply select the highest-DV cells per type
- The goal is sketch-size allocation across cell types (step 2, inv_cv/sqrt_n_sd/proportional weighting) rather than within-cell-type selection (step 3)
- DV scores are not yet computed — run the OOB random-forest DV computation (n_trees=100) first, since binning operates on adata.obs['dv']

## Inputs

- AnnData .h5ad with per-cell data value (DV) scores in adata.obs['dv'] and cell type labels (e.g. mTC mouse T cell dataset, 89,429 cells)
- Per-cell DV values and type labels from dv_values.csv (written by scValue with write_dv=True)
- Per-cell-type sketch size S_m derived upstream (e.g. inv_cv weighting with sketch_size=0.1)

## Outputs

- selected_cells_omega_m.csv — union index set Omega_m of length S_m for each binning strategy (FB and MTB)
- Subsampled cell index sets per cell type, one per strategy, ready for AnnData subsetting

## How to apply

Load the DV values (e.g. from the 89,429-cell mTC mouse T cell .h5ad, read adata.obs['dv'] with pandas) and restrict to the target cell type m with N_m DV values. For FB, partition the DV range [0.0, 1.0] into K=10 equal bins of width 0.1, allocate per-bin quotas proportional to bin membership summing exactly to S_m, then sort cells by DV descending within each bin and pick the top-DV cells per quota — this preserves a broad snapshot of variability including low-DV cells. For MTB, build the same 0.1-width bins but only consider bins lying above the cell-type mean DV, merging/dropping below-mean bins, and select top-DV cells within eligible bins up to S_m — this favors higher-value cells at the cost of low-DV diversity. The choice matters because, as the paper notes, FB/MTB 'can select cells at different value levels, preserving overall data diversity but still favoring higher-value cells'; FB is the scValue default (strategy="FB" in SCValue).

## Related tools

- **scValue** (Source of the DV-guided selection logic; its SCValue class exposes strategy='FB' (default, bins DV into 0.1 intervals) and strategy='MTB' (bins above the type mean DV only)) — https://github.com/LHBCB/scvalue
- **Scanpy** (Loads and preprocesses the input .h5ad (mTC dataset, top 3,000 HVGs, PCA into adata.obsm['X_pca']) whose obs['dv'] feeds the binning step)
- **pandas** (Reads dv_values.csv / adata.obs['dv'] per-cell DV scores and type labels, and writes selected_cells_omega_m.csv)
- **NumPy** (Implements the 0.1-width bin partition of [0,1], per-bin quota allocation, and descending DV sorts within bins)

## Examples

```
scv = SCValue(adata=adata, sketch_size=0.1, cell_type_key='cell_type', type_weighting='inv_cv', strategy='MTB', seed=42); adata_sub = scv.value()
```

## Evaluation signals

- len(Omega_m) == S_m for every cell type and for both strategies (FB and MTB)
- FB selections span all ten 0.1-interval bins over [0.0, 1.0], including low-DV bins
- MTB selections are drawn only from bins above the cell-type mean DV
- Per-bin quotas are proportional to bin membership and sum exactly to S_m (no off-by-one in the integer rounding)
- Selected indices are consistent with descending DV order within each bin (top-DV cells per quota)

## Limitations

- MTB drops all bins below the cell-type mean DV, so low-DV cells are excluded from the sketch when this strategy is chosen; only FB (the default) guarantees inclusion of lower OOB values
- Bin construction assumes DV scores normalized to the [0, 1] range; bin edges of 0.1 intervals from 0.0 to 1.0 are hard-coded in the FB/MTB strategies
- Selection quality depends on stable OOB DV estimates — the paper notes tuning random forest hyperparameters (number of trees, maximum depth) may be needed to stabilize OOB estimates in noisy conditions

## Evidence
