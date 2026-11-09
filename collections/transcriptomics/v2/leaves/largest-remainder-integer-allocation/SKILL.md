---
name: largest-remainder-integer-allocation
description: Use when fractional allocations (e.g. per-cell-type sketch sizes from
  value-weighted proportions) must be rounded to integer counts while guaranteeing
  the total equals a fixed budget S.
when_to_use_negative:
- The downstream allocation tolerates fractional quotas or the budget itself is approximate
  — rounding to satisfy an exact total is unnecessary
- Cells have not been assigned data values (no DV from OOB random forest estimates),
  so DV-weighted per-type sizes cannot be computed; scValue's 'proportional' fallback
  or uniform subsampling would apply instead
- Sketching is being done by distance- or distribution-based methods (GeoSketch, Sphetcher,
  Hopper, KH, scSampler) that do not allocate a budget per cell type
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scValue
  role: 'Source method: computes per-type weights from cell-type abundance and DV
    variability and applies the largest remainder method so per-type sketch sizes
    sum to the target sketch size S'
  repo: https://github.com/LHBCB/scvalue
- name: pandas
  role: groupby over the cell_type column to compute N_m, mean(DV) and std(DV) per
    type and handle the fractional allocations
- name: NumPy
  role: vectorized flooring, remainder ranking, and verification of the sum_m S_m
    == S invariant
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:6df71561f776021588b2a069150d86441d07a976df4a2956a4de520953814fec
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# largest-remainder-integer-allocation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts fractional per-cell-type sketch allocations into integer counts that sum exactly to the target sketch size S, using the largest remainder (Hare quota) method as implemented in scValue's sketch-size determination step.

## When to use

Use when fractional allocations (e.g. per-cell-type sketch sizes from value-weighted proportions) must be rounded to integer counts while guaranteeing the total equals a fixed budget S.

## When NOT to use

- The downstream allocation tolerates fractional quotas or the budget itself is approximate — rounding to satisfy an exact total is unnecessary
- Cells have not been assigned data values (no DV from OOB random forest estimates), so DV-weighted per-type sizes cannot be computed; scValue's 'proportional' fallback or uniform subsampling would apply instead
- Sketching is being done by distance- or distribution-based methods (GeoSketch, Sphetcher, Hopper, KH, scSampler) that do not allocate a budget per cell type

## Inputs

- Per-cell data values and cell-type labels (dv_values.csv: per-cell DV + type; dv_summary.csv: per-type DV summary statistics)
- Target sketch size S as a float fraction in (0, 1] or an integer cell count
- Weighting scheme choice: 'inv_cv' (default), 'sqrt_n_sd', or 'proportional'

## Outputs

- Integer per-type sketch sizes S_m with the invariant sum_m S_m == S
- Per-type sketch size allocation table (e.g. sketch_allocation.csv)

## How to apply

For each cell type m, compute N_m, mean(DV) and std(DV) via a pandas groupby over the cell_type column of dv_values.csv. Compute raw per-type weights with the chosen scheme — 'inv_cv' (default) w_m ∝ mean(DV) * sqrt(N_m) / std(DV), 'sqrt_n_sd' w_m ∝ sqrt(N_m) * std(DV), or 'proportional' w_m ∝ original cell-type frequencies — then normalize over M types and scale by S (a float fraction in (0,1] times total cells, or an integer count) to get fractional allocations S_m. Floor each S_m, rank the fractional remainders, and assign the leftover cells one-by-one to the types with the largest remainders so that sum_m S_m == S exactly. Verify the invariant with NumPy and write the per-type table (e.g. sketch_allocation.csv) before passing S_m to the FB/MTB/TP within-type selection strategy.

## Related tools

- **scValue** (Source method: computes per-type weights from cell-type abundance and DV variability and applies the largest remainder method so per-type sketch sizes sum to the target sketch size S) — https://github.com/LHBCB/scvalue
- **pandas** (groupby over the cell_type column to compute N_m, mean(DV) and std(DV) per type and handle the fractional allocations)
- **NumPy** (vectorized flooring, remainder ranking, and verification of the sum_m S_m == S invariant)

## Examples

```
import pandas as pd, numpy as np; g = pd.read_csv('dv_values.csv').groupby('cell_type')['DV']; w = g.mean()*np.sqrt(g.size())/g.std(); f = w/w.sum()*S; S_m = np.floor(f).astype(int); S_m[np.argsort(-(f-np.floor(f)))[:int(S-S_m.sum())]] += 1; assert S_m.sum()==S
```

## Evaluation signals

- Exact-budget invariant: NumPy check that sum of integer per-type sizes equals S (integer mode) or S * N_cells (fraction mode) with zero rounding error
- All S_m are non-negative integers, and every cell type with nonzero raw weight receives at least one cell when S >= M
- Fractional allocations before flooring match normalized weights * S; the number of assigned leftover cells equals S minus the sum of floors
- Re-running with the same dv_values.csv and weighting scheme reproduces the identical sketch_allocation.csv
- Setting type_weighting='proportional' reproduces the original cell-type frequencies up to integer-rounding error

## Limitations

- Rounding allocates leftover cells by largest fractional remainder, which can over- or under-assign relative to the exact fractional allocation for individual small cell types
- The 'inv_cv' scheme divides by std(DV), so cell types with near-zero DV variance produce very large or unstable weights
- The scValue paper does not report tool versions for the benchmarked comparison methods, and the confidence-score threshold (if downstream filtering is chained) is user-defined with no default

## Evidence

- [methods] Per-type sketch sizes are computed from cell-type abundance and data-value variability (Equation 2), and the largest remainder method is applied so the total allocation across all cell types sums up exactly to the target sketch size S.: [evidence span withheld — anchor status: not_found]
