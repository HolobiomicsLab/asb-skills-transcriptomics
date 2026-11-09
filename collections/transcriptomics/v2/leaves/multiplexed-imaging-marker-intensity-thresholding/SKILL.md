---
name: multiplexed-imaging-marker-intensity-thresholding
description: Use when you have a single-cell marker-intensity table (e.g., MCMICRO
  output from CyCIF imaging, ~1.1M cells x 30 antibody channels) and need to binarize
  marker expression into per-cell proliferation/arrest labels using user-defined thresholds.
when_to_use_negative:
- Input is raw multi-channel whole-slide imagery rather than single-cell intensity
  data — run MCMICRO first, since its output is the input of SpatialCells
- Goal is spatial/region assignment (immune-isolated vs. immune-rich subregions, boundaries,
  communities) rather than per-cell marker-based proliferation classification — use
  spatialcells.spatial functions instead
- Marker channels of interest are not quantitative intensity features in the table,
  so user-defined gates cannot be meaningfully set
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: SpatialCells
  role: Provides setGate/setGates (spatialcells.prep) to set user-defined intensity
    thresholds and getMPI (spatialcells.msmt) to compute the multivariate proliferation
    index per cell
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream tool that transforms multi-channel whole-slide CyCIF images into
    the single-cell marker-intensity data consumed by this skill
- name: Anndata
  role: In-memory container for the per-cell marker-intensity table on which gates
    and MPI labels are computed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:c7b95c102e7cd8123d56495e92676bc715cf3ae41bb7106d111eb77ebaf52004
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
  - Anndata
  - MCMICRO
  - SpatialCells
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# multiplexed-imaging-marker-intensity-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Defines user-specified intensity gates on multiplexed CyCIF single-cell marker channels and applies the SpatialCells multivariate proliferation index (MPI) to classify each cell as proliferating (MPI=1), arrest-marker positive (MPI=-1), or neither (MPI=0).

## When to use

Use when you have a single-cell marker-intensity table (e.g., MCMICRO output from CyCIF imaging, ~1.1M cells x 30 antibody channels) and need to binarize marker expression into per-cell proliferation/arrest labels using user-defined thresholds.

## When NOT to use

- Input is raw multi-channel whole-slide imagery rather than single-cell intensity data — run MCMICRO first, since its output is the input of SpatialCells
- Goal is spatial/region assignment (immune-isolated vs. immune-rich subregions, boundaries, communities) rather than per-cell marker-based proliferation classification — use spatialcells.spatial functions instead
- Marker channels of interest are not quantitative intensity features in the table, so user-defined gates cannot be meaningfully set

## Inputs

- Single-cell marker-intensity table from MCMICRO preprocessing of CyCIF whole-slide images (MEL1: 1,110,585 cells, 30 antibody channels)
- AnnData object holding per-cell marker intensities
- User-specified intensity gate thresholds for proliferation markers (PHH3, Ki67, PCNA, MCM2) and arrest markers (p21, p27)

## Outputs

- Per-cell MPI labels in {-1, 0, 1} (e.g., exported to mel1_per_cell_mpi.csv)
- Fraction of SOX10+ tumor cells with MPI = 1 (reference value: 7.4% for MEL1)

## How to apply

Load the MCMICRO-derived single-cell marker-intensity table (e.g., the MEL1 melanoma sample: 1,110,585 cells, 30 CyCIF antibody channels) into an AnnData object. Define gates for two marker lists with spatialcells.prep.setGate/setGates: (1) mitosis/proliferation markers — Serine-10 phosphorylated histone H3 (PHH3), Ki67, PCNA, MCM2 — and (2) cell cycle arrest markers — p21 and p27; thresholds are user-specified, so inspect per-channel intensity distributions before choosing them. Apply spatialcells.msmt.getMPI, which assigns each cell a label in {-1, 0, 1} via a piecewise rule: MPI = 1 when at least one proliferation marker exceeds its gate and no arrest marker does, MPI = -1 when an arrest marker exceeds its gate, MPI = 0 otherwise. Export the per-cell labels (e.g., to mel1_per_cell_mpi.csv) and validate by computing the fraction of SOX10+ tumor cells with MPI = 1 against the reported reference of 7.4% for MEL1. Standardized, user-defined thresholding is a core design point of the package.

## Related tools

- **SpatialCells** (Provides setGate/setGates (spatialcells.prep) to set user-defined intensity thresholds and getMPI (spatialcells.msmt) to compute the multivariate proliferation index per cell) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream tool that transforms multi-channel whole-slide CyCIF images into the single-cell marker-intensity data consumed by this skill)
- **Anndata** (In-memory container for the per-cell marker-intensity table on which gates and MPI labels are computed)

## Examples

```
from spatialcells.prep import setGates; from spatialcells.msmt import getMPI; setGates(adata, ['PHH3','Ki67','PCNA','MCM2'], gates1, ['p21','p27'], gates2); adata.obs['MPI'] = getMPI(adata, ...)
```

## Evaluation signals

- All MPI labels are restricted to the set {-1, 0, 1}; no other values appear in the output table
- Piecewise rule holds per cell: MPI = -1 whenever any arrest marker (p21/p27) exceeds its gate; MPI = 1 requires at least one proliferation marker (PHH3/Ki67/PCNA/MCM2) above gate AND no arrest marker above gate
- Fraction of SOX10+ tumor cells with MPI = 1 matches the published reference of 7.4% for the MEL1 sample
- Row count of the exported mel1_per_cell_mpi.csv equals the number of cells in the AnnData input (1,110,585 for MEL1)
- Gate thresholds are reproducible/documented, since SpatialCells is designed for standardized analysis with user-defined parameters

## Limitations

- Gate thresholds are user-specified and not learned from data, so results depend on operator choice of cutoffs per channel
- SpatialCells can currently only be installed from source (pip install -r requirements.txt; pip install .), which may complicate reproducible environments
- No software version is stated for SpatialCells, so pinning the exact analyzed version requires care
- MPI reflects only the gated marker set (mitosis/proliferation vs. arrest markers) and does not by itself localize cells spatially

## Evidence
