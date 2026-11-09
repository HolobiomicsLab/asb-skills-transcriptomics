---
name: proliferation-index-piecewise-scoring
description: Use when single-cell marker-intensity data from multiplexed imaging (e.g.
  CyCIF) must be scored per cell for proliferation vs. cell-cycle arrest, to derive
  the fraction of a cell type (e.g. SOX10+ tumor cells) with MPI = 1.
when_to_use_negative:
- Input is raw multi-channel whole-slide imagery that has not yet been converted to
  single-cell data — run MCMICRO (or equivalent) first, since 'The output of MCMICRO
  is the input of SpatialCells'
- Goal is spatial region assignment (immune-isolated vs. immune-rich subregions, tumor
  isolation index) rather than per-cell proliferation scoring — use assignPointsToRegions/getRegionComposition
  instead
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: SpatialCells
  role: Provides prep.setGate/setGates for marker gating and msmt.getMPI for the piecewise
    MPI scoring rule
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide images into
    the single-cell marker-intensity table consumed by getMPI
- name: AnnData
  role: Container format for the per-cell marker-intensity matrix the gates and MPI
    labels are attached to
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:cfb48752625125d29aff4f58a92fe00d9c4291815ccfe4a11685eeaa3710589e
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
  - AnnData
  - MCMICRO
  - SpatialCells
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# proliferation-index-piecewise-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes a multivariate proliferation index (MPI) per cell with SpatialCells' getMPI, combining gated mitosis/proliferation markers (PHH3, Ki67, PCNA, MCM2) and cell-cycle arrest markers (p21, p27) into a piecewise {-1, 0, 1} label.

## When to use

Use when single-cell marker-intensity data from multiplexed imaging (e.g. CyCIF) must be scored per cell for proliferation vs. cell-cycle arrest, to derive the fraction of a cell type (e.g. SOX10+ tumor cells) with MPI = 1.

## When NOT to use

- Input is raw multi-channel whole-slide imagery that has not yet been converted to single-cell data — run MCMICRO (or equivalent) first, since 'The output of MCMICRO is the input of SpatialCells'
- Goal is spatial region assignment (immune-isolated vs. immune-rich subregions, tumor isolation index) rather than per-cell proliferation scoring — use assignPointsToRegions/getRegionComposition instead

## Inputs

- Single-cell marker-intensity table from MCMICRO output of a multiplexed imaging sample (e.g. MEL1 CyCIF, 30 antibody channels, ~1.1M cells) as an AnnData object
- List of mitosis/proliferation marker channels (e.g. PHH3, Ki67, PCNA, MCM2)
- List of cell-cycle arrest marker channels (e.g. p21, p27)
- User-specified intensity gate thresholds for each marker (via spatialcells.prep.setGate/setGates)

## Outputs

- Per-cell MPI labels in {-1, 0, 1} (exportable as e.g. mel1_per_cell_mpi.csv)
- Summary statistic such as the percentage of a marker-defined cell type (e.g. SOX10+ tumor cells) with MPI = 1

## How to apply

Load the single-cell marker-intensity table (e.g. MEL1: 1,110,585 cells, 30 CyCIF antibody channels preprocessed with MCMICRO) into an AnnData object. Define intensity gates for proliferation markers (Serine-10 phosphorylated histone H3, Ki67, PCNA, MCM2) and arrest markers (p21, p27) with spatialcells.prep.setGate/setGates at user-specified thresholds. Apply spatialcells.msmt.getMPI, which assigns MPI = 1 when at least one proliferation marker exceeds its gate and no arrest marker does, MPI = -1 when an arrest marker exceeds its gate, and MPI = 0 otherwise. Export per-cell labels (e.g. mel1_per_cell_mpi.csv) and summarize, e.g. the fraction of SOX10+ tumor cells with MPI = 1, which for MEL1 is reported as 7.4%.

## Related tools

- **SpatialCells** (Provides prep.setGate/setGates for marker gating and msmt.getMPI for the piecewise MPI scoring rule) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide images into the single-cell marker-intensity table consumed by getMPI)
- **AnnData** (Container format for the per-cell marker-intensity matrix the gates and MPI labels are attached to)

## Examples

```
import spatialcells.prep as prep, spatialcells.msmt as msmt; adata = prep.setGate(adata, mitosisGates, markers=proliferationMarkers); adata.obs['MPI'] = msmt.getMPI(adata, proliferationMarkers, arrestMarkers)
```

## Evaluation signals

- All per-cell MPI values are restricted to {-1, 0, 1} per the piecewise rule
- Cells with any arrest marker above its gate (p21 or p27) carry MPI = -1 and never MPI = 1
- Cells with a proliferation marker above its gate (PHH3, Ki67, PCNA, or MCM2) and no arrest marker above gate carry MPI = 1
- For MEL1, the overall percentage of SOX10+ proliferating tumor cells (MPI = 1) should reproduce the reported reference value of 7.4%
- Exported CSV (e.g. mel1_per_cell_mpi.csv) contains one MPI label per cell matching the 1,110,585-cell input table

## Limitations

- Gate thresholds are user-defined via setGate/setGates, so results depend on analyst-specified intensity cutoffs
- MPI = 0 is a residual class (neither proliferation nor arrest above gate), so it mixes quiescent and uninformative cells
- SpatialCells can currently only be installed from source (pip install -r requirements.txt; pip install .), and no software version or changelog is stated

## Evidence

- [methods] mitosis/proliferation and cell cycle arrest markers: [evidence span withheld — anchor status: not_found]
- [readme] README install steps: [evidence span withheld — anchor status: not_found]
