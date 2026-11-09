---
name: data-value-variability-normalization-across-cell-types
description: Use when a scRNA-seq subsampling pipeline must divide a target sketch
  size S across cell types according to DV abundance/variability, and the per-type
  integer allocations must sum exactly to S.
when_to_use_negative:
- If the goal is proportional subsampling without DV weighting — use scValue's prop_sampling
  option (proportional to original cell-type frequencies) instead of reconstructing
  the weighting scheme
- If cell-type labels or per-cell DV scores are unavailable (only a feature matrix
  like adata.obsm['X_pca']) — the DV computation step must run first
- If the downstream method optimizes distance or distribution (GeoSketch, Sphetcher,
  Hopper, scSampler, KH) rather than DV-weighted per-type allocation
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_2269
tools:
- name: scValue
  role: Source method whose step-2 sketch size allocation (type_weighting='inv_cv'|'sqrt_n_sd'|'proportional')
    is being reconstructed; open-source Python package installable via pip
  repo: https://github.com/LHBCB/scvalue
- name: pandas
  role: Groupby over the cell_type column of dv_values.csv to compute per-type N_m,
    mean(DV) and std(DV); write sketch_allocation.csv
- name: NumPy
  role: Vectorized normalization of weights and verification that sum_m S_m == S holds
    exactly
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:4a8a34f607509e4d6db0bfec021202f71bd694a63fb79d88ee11b08f3959dd0b
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
  - pandas
  - scValue
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# data-value-variability-normalization-across-cell-types

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs scValue's value-weighted sketch size allocation step: per-cell-type sketch sizes are computed from cell-type abundance and data-value (DV) variability, and the largest remainder method guarantees integer allocations S_m that sum exactly to the target sketch size S.

## When to use

Use when a scRNA-seq subsampling pipeline must divide a target sketch size S across cell types according to DV abundance/variability, and the per-type integer allocations must sum exactly to S.

## When NOT to use

- If the goal is proportional subsampling without DV weighting — use scValue's prop_sampling option (proportional to original cell-type frequencies) instead of reconstructing the weighting scheme
- If cell-type labels or per-cell DV scores are unavailable (only a feature matrix like adata.obsm['X_pca']) — the DV computation step must run first
- If the downstream method optimizes distance or distribution (GeoSketch, Sphetcher, Hopper, scSampler, KH) rather than DV-weighted per-type allocation

## Inputs

- dv_values.csv with per-cell data values (DV) and cell-type labels (written by scValue when write_dv=True)
- Target sketch size: a float fraction in (0, 1] or an integer cell count
- Chosen type weighting scheme: 'inv_cv' (default), 'sqrt_n_sd', or 'proportional'

## Outputs

- Integer per-type sketch sizes S_m satisfying sum_m S_m == S exactly
- sketch_allocation.csv table of per-type sketch sizes
- Fractional and floored allocations plus remainder ranks used by the largest remainder method

## How to apply

Load per-cell data values and cell-type labels from dv_values.csv with pandas and read the target sketch size (a float fraction in (0,1] or an integer cell count). For each cell type m, groupby over the cell_type column to obtain N_m, mean(DV) and std(DV), then compute raw weights under the chosen scheme: 'inv_cv' (default) w_m ∝ mean(DV) * sqrt(N_m) / std(DV), 'sqrt_n_sd' w_m ∝ sqrt(N_m) * std(DV), or 'proportional' w_m ∝ original cell-type frequencies. Normalize weights over the M cell types and scale by the total sketch size S to get fractional allocations. Apply the largest remainder method — floor each fractional allocation, then assign leftover cells to the types with the largest fractional remainders — to obtain integer per-type sizes S_m. Verify with NumPy that sum_m S_m == S holds exactly and write the allocation table to sketch_allocation.csv; these S_m values then feed scValue's per-type DV-guided cell selection (FB, MTB, or TP strategy).

## Related tools

- **scValue** (Source method whose step-2 sketch size allocation (type_weighting='inv_cv'|'sqrt_n_sd'|'proportional') is being reconstructed; open-source Python package installable via pip) — https://github.com/LHBCB/scvalue
- **pandas** (Groupby over the cell_type column of dv_values.csv to compute per-type N_m, mean(DV) and std(DV); write sketch_allocation.csv)
- **NumPy** (Vectorized normalization of weights and verification that sum_m S_m == S holds exactly)

## Examples

```
import pandas as pd; g = pd.read_csv('dv_values.csv').groupby('cell_type')['dv']; w = g.mean() * g.size()**0.5 / g.std(); alloc = (w / w.sum() * S); S_m = alloc.astype(int); S_m += (alloc - S_m).rank(ascending=False) <= (S - S_m.sum())
```

## Evaluation signals

- Exact invariant check with NumPy: sum of integer per-type allocations equals the target sketch size S (sum_m S_m == S)
- With type_weighting='proportional', per-type sketch fractions should match original cell-type frequency proportions
- With type_weighting='inv_cv', types with higher mean(DV) and larger sqrt(N_m)/std(DV) receive proportionally larger allocations
- All S_m are non-negative integers after the largest remainder method (floor + leftover assignment to largest fractional remainders)
- sketch_allocation.csv row count equals the number of distinct cell types M in dv_values.csv

## Limitations

- Subsample sizes per type depend on both abundance and variability of DV; 'inv_cv' down-weights types with high DV variability (large std), so rare but highly variable types may get fewer cells than expected
- Requires per-cell DV scores from OOB random forest estimates; noisy OOB estimates may distort allocations unless random forest hyperparameters (n_trees, max depth) are tuned
- If sketch_size is given as a float, cell count depends on N * fraction; the exact integer S must be resolved before the largest remainder method applies

## Evidence

- [other] Per-type sketch sizes are computed from cell-type abundance and data-value variability (Equation 2): [evidence span withheld — anchor status: not_found]
- [other] largest remainder method sums exactly to S: [evidence span withheld — anchor status: not_found]
