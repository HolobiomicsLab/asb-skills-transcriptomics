---
name: annadata-single-cell-data-handling
description: Use when multiplexed imaging data preprocessed with MCMICRO into a single-cell
  marker-intensity table (per cell × antibody channel) must be held in AnnData for
  SpatialCells gating, MPI scoring, or region-based TME measurement.
when_to_use_negative:
- The input is still multi-channel whole-slide image data that has not been converted
  to single-cell marker intensities — MCMICRO preprocessing must run first, since
  'The output of MCMICRO is the input of SpatialCells'.
- The analysis requires measuring spatial regions, neighborhoods, or tumor/immune
  boundaries (assignPointsToRegions, getCommunities, getBoundary, getCellKDE) rather
  than loading and gating per-cell marker intensities.
- The data are bulk or non-imaging expression profiles where per-cell marker-intensity
  gating over CyCIF antibody channels does not apply.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3317
tools:
- name: SpatialCells
  role: Provides spatialcells.prep.setGate/setGates for defining marker-intensity
    gates and spatialcells.msmt.getMPI for assigning per-cell MPI labels {-1, 0, 1}
    on the AnnData object
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing tool that transforms multi-channel whole-slide images
    into the single-cell data that become the AnnData input
- name: Anndata
  role: Container format holding the per-cell marker-intensity matrix and .obs annotations
    that SpatialCells functions read and extend
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:da33e49bf7deb182b064972799b9ab1222b8dbaa2ae105cdfe3a46c2ac54adad
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3317
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# annadata-single-cell-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loads MCMICRO-derived single-cell marker-intensity tables (e.g., the MEL1 CyCIF melanoma sample with 1,110,585 cells across 30 antibody channels) into AnnData objects so SpatialCells functions (setGate/setGates, getMPI) can operate on per-cell annotations. It is the data-ingestion step that converts whole-slide imaging output into a structured, gate-annotated single-cell object.

## When to use

Use when multiplexed imaging data preprocessed with MCMICRO into a single-cell marker-intensity table (per cell × antibody channel) must be held in AnnData for SpatialCells gating, MPI scoring, or region-based TME measurement.

## When NOT to use

- The input is still multi-channel whole-slide image data that has not been converted to single-cell marker intensities — MCMICRO preprocessing must run first, since 'The output of MCMICRO is the input of SpatialCells'.
- The analysis requires measuring spatial regions, neighborhoods, or tumor/immune boundaries (assignPointsToRegions, getCommunities, getBoundary, getCellKDE) rather than loading and gating per-cell marker intensities.
- The data are bulk or non-imaging expression profiles where per-cell marker-intensity gating over CyCIF antibody channels does not apply.

## Inputs

- Single-cell marker-intensity table produced by MCMICRO from multi-channel whole-slide images (e.g., MEL1: 1,110,585 cells, 30 CyCIF antibody channels)
- User-specified intensity gate thresholds for each marker (proliferation markers: PHH3, Ki67, PCNA, MCM2; arrest markers: p21, p27)
- AnnData object holding the intensity matrix and per-cell annotations (.obs), including cell-type labels such as SOX10+ tumor cells

## Outputs

- AnnData object with marker gates applied and per-cell MPI labels in {-1, 0, 1} added to .obs
- Per-cell MPI label CSV export (e.g., mel1_per_cell_mpi.csv)
- Summary statistics such as the fraction of SOX10+ tumor cells with MPI = 1 (7.4% for the MEL1 reference sample)

## How to apply

Take the MCMICRO output — a per-cell marker-intensity table such as MEL1 (1,110,585 cells, 30 CyCIF antibody channels) — and load it into an AnnData object with intensities as the expression matrix and per-cell metadata in .obs. Define user-specified intensity thresholds for the markers of interest via spatialcells.prep.setGate/setGates; for the proliferation-index use case these are mitosis/proliferation markers (Serine-10 phosphorylated histone H3/PHH3, Ki67, PCNA, MCM2) and cell cycle arrest markers (p21, p27). With gates in place, apply spatialcells.msmt.getMPI, which assigns each cell an MPI label in {-1, 0, 1} by a piecewise rule: MPI = 1 when at least one proliferation marker exceeds its gate and no arrest marker does; MPI = -1 when an arrest marker exceeds its gate; MPI = 0 otherwise. Export the per-cell labels (e.g., to mel1_per_cell_mpi.csv) and compute subtype-specific fractions such as the percentage of SOX10+ tumor cells with MPI = 1. The rationale is that SpatialCells requires an AnnData-structured single-cell input — 'The output of MCMICRO is the input of SpatialCells' — and gating thresholds are deliberately user-defined so the same pipeline can be standardized across samples.

## Related tools

- **SpatialCells** (Provides spatialcells.prep.setGate/setGates for defining marker-intensity gates and spatialcells.msmt.getMPI for assigning per-cell MPI labels {-1, 0, 1} on the AnnData object) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing tool that transforms multi-channel whole-slide images into the single-cell data that become the AnnData input)
- **Anndata** (Container format holding the per-cell marker-intensity matrix and .obs annotations that SpatialCells functions read and extend)

## Examples

```
from spatialcells.msmt import getMPI; adata.obs['MPI'] = getMPI(adata, prolif_markers, arrest_markers, gates)
```

## Evaluation signals

- Every cell receives exactly one MPI label and all labels fall in the set {-1, 0, 1}; no cell is left unlabeled.
- Rule consistency: any cell with an arrest marker (p21 or p27) above its gate must have MPI = -1; cells with a proliferation marker (PHH3, Ki67, PCNA, MCM2) above gate and no arrest marker above gate must have MPI = 1; remaining cells are MPI = 0.
- The AnnData cell count matches the expected MCMICRO output (1,110,585 cells for MEL1) and 30 antibody channels are represented.
- The computed fraction of SOX10+ tumor cells with MPI = 1 reproduces the published reference value of 7.4% for the MEL1 sample.
- The exported per-cell label file (e.g., mel1_per_cell_mpi.csv) has one row per cell in the AnnData object with matching indices.

## Limitations

- Gate thresholds for proliferation and arrest markers are user-specified, so results depend on the chosen intensity cutoffs; the article describes 'preprocessing and analyzing data in a standardized manner with user-defined parameters'.
- SpatialCells 'can currently only be installed from source' (pip install -r requirements.txt; pip install ., or conda env create --name spatialcells --file=conda.yaml), and no software version is stated.
- Fragmented areas of SOX10+ cells should be excluded before downstream spatial measurements, as including 'separate fragments of SOX10+ cells along with the intervening empty spaces' would artificially increase distance measurements.

## Evidence

- [results] MCMICRO converts whole-slide images to SpatialCells input: "preprocessed using MCMICRO [7], which transforms multi-channel whole-slide images into single-cell data. The output of MCMICRO is the input of SpatialCells."
- [methods] getMPI is a SpatialCells measurement function: "getMPI
------
.. autofunction:: spatialcells.msmt.getMPI"
- [results] MEL1 sample scale: "publicly available multiplexed imaging data of a cutaneous melanoma sample (MEL1) consisting of 1 110 585 cells"
- [results] Reference MPI validation value for MEL1: "The overall percentage of proliferating tumor cells (MPI = 1) in the MEL1 sample is 7.4%."
- [abstract] SpatialCells processes millions of cells: "SpatialCells efficiently streamlines the automated extraction of features from multiplexed single-cell data and can process samples containing millions of cells."
- [readme] README scope statement: "This package aims to develop computational methods for region-based analyses of multiplexed single-cell data."
