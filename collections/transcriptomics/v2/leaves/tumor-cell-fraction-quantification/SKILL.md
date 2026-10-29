---
name: tumor-cell-fraction-quantification
description: Use when a marker-intensity single-cell table from multiplexed imaging
  (e.g. MCMICRO output of a CyCIF slide) must be scored per-cell for proliferation
  status (MPI in {-1, 0, 1}) and the fraction of a tumor-cell type such as SOX10+
  cells with MPI = 1 quantified.
when_to_use_negative:
- Input is raw multi-channel whole-slide images rather than MCMICRO-derived single-cell
  marker-intensity data — SpatialCells expects the output of MCMICRO as its input
- Input is spatial transcriptomics data (e.g. the Visium lymph node sample deconvoluted
  with Cell2location) rather than multiplexed single-cell imaging marker intensities,
  where region-based rather than marker-gated MPI analysis applies
- Gates for the proliferation and arrest markers have not been defined — getMPI requires
  setGate/setGates thresholds to run
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: SpatialCells
  role: Provides spatialcells.prep.setGate/setGates to define marker gates and spatialcells.msmt.getMPI
    to assign per-cell MPI labels used to quantify the proliferating tumor-cell fraction
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide images into
    the single-cell data that serves as SpatialCells' input
- name: AnnData
  role: Container format for loading the single-cell marker-intensity table prior
    to gate definition and MPI computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:507f1dcd7319b9244c588639ccb4f1546c4f5b54f15888c735d500e8565412ce
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tumor-cell-fraction-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the fraction of a marker-defined tumor-cell population (e.g. SOX10+ melanoma cells) in a proliferating state by assigning per-cell multivariate proliferation index (MPI) labels with spatialcells.msmt.getMPI from SpatialCells. MPI integrates mitosis/proliferation markers against cell cycle arrest markers so a single per-cell label in {-1, 0, 1} can be aggregated into population-level proliferation fractions.

## When to use

Use when a marker-intensity single-cell table from multiplexed imaging (e.g. MCMICRO output of a CyCIF slide) must be scored per-cell for proliferation status (MPI in {-1, 0, 1}) and the fraction of a tumor-cell type such as SOX10+ cells with MPI = 1 quantified.

## When NOT to use

- Input is raw multi-channel whole-slide images rather than MCMICRO-derived single-cell marker-intensity data — SpatialCells expects the output of MCMICRO as its input
- Input is spatial transcriptomics data (e.g. the Visium lymph node sample deconvoluted with Cell2location) rather than multiplexed single-cell imaging marker intensities, where region-based rather than marker-gated MPI analysis applies
- Gates for the proliferation and arrest markers have not been defined — getMPI requires setGate/setGates thresholds to run

## Inputs

- Single-cell marker-intensity table produced by MCMICRO from multi-channel whole-slide images (e.g. MEL1 CyCIF, 1,110,585 cells, 30 antibody channels)
- AnnData object holding the single-cell data
- User-specified gate thresholds for proliferation markers (PHH3, Ki67, PCNA, MCM2)
- User-specified gate thresholds for arrest markers (p21, p27)

## Outputs

- Per-cell MPI labels in {-1, 0, 1} (exported as e.g. mel1_per_cell_mpi.csv)
- Fraction of tumor-cell-type cells (e.g. SOX10+) with MPI = 1 (reference: 7.4% for MEL1)

## How to apply

Load the MCMICRO-preprocessed single-cell marker-intensity table (for the MEL1 melanoma sample: 1,110,585 cells across 30 CyCIF antibody channels) into an AnnData object. Define user-specified intensity thresholds with spatialcells.prep.setGate/setGates for two marker lists: proliferation/mitosis markers (Serine-10 phosphorylated histone H3, Ki67, PCNA, MCM2) and cell cycle arrest markers (p21, p27). Apply spatialcells.msmt.getMPI, which uses the piecewise rule MPI = 1 when at least one proliferation marker exceeds its gate and no arrest marker does; MPI = -1 when any arrest marker exceeds its gate; MPI = 0 otherwise. Export per-cell MPI labels (e.g. mel1_per_cell_mpi.csv), then compute the fraction of the tumor-cell type (SOX10+) with MPI = 1. Validate the computed fraction against the published reference value of 7.4% proliferating tumor cells (MPI = 1) for MEL1.

## Related tools

- **SpatialCells** (Provides spatialcells.prep.setGate/setGates to define marker gates and spatialcells.msmt.getMPI to assign per-cell MPI labels used to quantify the proliferating tumor-cell fraction) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide images into the single-cell data that serves as SpatialCells' input)
- **AnnData** (Container format for loading the single-cell marker-intensity table prior to gate definition and MPI computation)

## Examples

```
from spatialcells.prep import setGates; from spatialcells.msmt import getMPI; getMPI(adata, mitosis_markers, arrest_markers)
```

## Evaluation signals

- Every assigned MPI label falls in the set {-1, 0, 1}; any other value indicates a rule violation
- Cells flagged MPI = -1 must all have at least one arrest marker (p21 or p27) exceeding its gate
- Cells flagged MPI = 1 must have at least one proliferation marker (PHH3, Ki67, PCNA, or MCM2) above its gate and no arrest marker above its gate
- The number of exported MPI labels equals the number of input cells (1,110,585 for MEL1)
- Computed fraction of SOX10+ tumor cells with MPI = 1 matches the reported 7.4% for the MEL1 sample

## Limitations

- Marker gate thresholds are user-defined with no automated cutoff selection, so the resulting MPI fraction is sensitive to threshold choices
- SpatialCells can currently only be installed from source, and no software version or changelog is stated
- Fragmented areas of SOX10+ cells should be excluded and analysis focused on the main tumor area; including separate fragments with intervening empty spaces would artificially inflate spatial measurements

## Evidence
