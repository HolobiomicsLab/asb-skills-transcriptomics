---
name: melanoma-tumor-cell-fraction-analysis
description: Use when per-cell MPI labels for MEL1 tumor cells (from a getMPI reconstruction)
  are available and the goal is the overall proliferating (MPI = 1) tumor-cell fraction
  to verify against the reported 7.4%.
when_to_use_negative:
- Input is raw multiplexed image data or MCMICRO output that has not yet been gated/phenotyped
  and passed through getMPI — no per-cell MPI labels exist to count
- The analysis goal is region-level composition (e.g., proportion of MPI = 1 cells
  within immune-rich vs. immune-isolated subregions) rather than the overall sample-level
  fraction
edam_operation: http://edamontology.org/operation_2945
tools:
- name: SpatialCells
  role: Provides the getMPI function (spatialcells.msmt.getMPI) that produces the
    per-cell multivariate proliferation index labels whose MPI = 1 fraction is computed
    here
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms the MEL1 multi-channel whole-slide
    CyCIF images into the single-cell data that is the input to SpatialCells
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:af2beac42f5efe2ec8206775d934b0638466b4c15b899035b23170113b96a1e9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/SemenovLab/SpatialCells
  repo_url: https://github.com/SemenovLab/SpatialCells
  tools:
  - MCMICRO
  - SpatialCells
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# melanoma-tumor-cell-fraction-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the overall percentage of proliferating tumor cells (MPI = 1) among all tumor cells in the MEL1 cutaneous melanoma CyCIF sample using SpatialCells' multivariate proliferation index, yielding the paper's reference value of 7.4%.

## When to use

Use when per-cell MPI labels for MEL1 tumor cells (from a getMPI reconstruction) are available and the goal is the overall proliferating (MPI = 1) tumor-cell fraction to verify against the reported 7.4%.

## When NOT to use

- Input is raw multiplexed image data or MCMICRO output that has not yet been gated/phenotyped and passed through getMPI — no per-cell MPI labels exist to count
- The analysis goal is region-level composition (e.g., proportion of MPI = 1 cells within immune-rich vs. immune-isolated subregions) rather than the overall sample-level fraction

## Inputs

- Per-cell MPI labels for MEL1 tumor cells (output of SpatialCells getMPI reconstruction), loaded into a pandas DataFrame
- MCMICRO-preprocessed single-cell data from the MEL1 CyCIF sample (30 antibody markers, ~1,110,585 cells)

## Outputs

- Single percentage value of MPI = 1 proliferating tumor cells among all tumor cells, written to mpi_fraction_results.csv (expected ≈ 7.4%)

## How to apply

Load the per-cell MPI labels for MEL1 tumor cells — the output of the SpatialCells getMPI reconstruction applied to MCMICRO-preprocessed single-cell data — into a pandas DataFrame. Compute the fraction of cells labeled MPI = 1 (proliferating, Ki67-based multivariate proliferation index) out of all tumor cells, convert to a percentage, and write the single value to mpi_fraction_results.csv. Validate by checking the computed percentage falls within ±0.5 percentage points of the reported 7.4% overall proliferating tumor cell fraction for MEL1. Note this computes the whole-sample (overall ROI) fraction, not region-specific proportions such as the immune-rich or immune-isolated subregions.

## Related tools

- **SpatialCells** (Provides the getMPI function (spatialcells.msmt.getMPI) that produces the per-cell multivariate proliferation index labels whose MPI = 1 fraction is computed here) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms the MEL1 multi-channel whole-slide CyCIF images into the single-cell data that is the input to SpatialCells)

## Examples

```
import pandas as pd; df = pd.read_csv('MEL1_mpi_labels.csv'); pct = (df['MPI'] == 1).mean() * 100; open('mpi_fraction_results.csv', 'w').write(f'{pct:.1f}')
```

## Evaluation signals

- Computed MPI = 1 percentage falls within ±0.5 percentage points of the reported 7.4% for MEL1
- mpi_fraction_results.csv contains exactly one percentage value derived from all tumor cells (denominator = total tumor cells, not a subset)
- MPI labels trace to a getMPI reconstruction on MCMICRO-preprocessed MEL1 single-cell data
- Counting only tumor cells (e.g., SOX10+ tumor cells), not immune or stromal cells, in the denominator

## Limitations

- SpatialCells can currently only be installed from source, and no software version is stated in the paper
- The 7.4% reference applies to the overall ROI of the MEL1 sample; region-specific proportions (e.g., immune-rich regions) differ and are not captured by this whole-sample fraction
- No changelog or version information is available for the repository

## Evidence

- [readme] Installation and documentation: [evidence span withheld — anchor status: external_doc]
