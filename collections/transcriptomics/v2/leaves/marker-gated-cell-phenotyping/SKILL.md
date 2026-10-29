---
name: marker-gated-cell-phenotyping
description: Use when the input is per-cell x/y coordinates plus multi-marker intensity
  values (e.g., MCMICRO output from a CyCIF whole-slide image in an AnnData object)
  and the analysis goal is to label each cell by marker-defined type (e.g., SOX10+
  tumor cells) before spatial/region analyses.
when_to_use_negative:
- Input is spot-level or bulk data rather than per-cell multiplexed imaging intensities
  — e.g., the Visium lymph node dataset, which the paper deconvolutes with Cell2location
  before SpatialCells analysis rather than gating single-cell markers
- The task is spatial region delineation or per-cell region assignment, not phenotype
  calling — those are handled by getBoundary and assignPointsToRegions, which require
  already-gated cell-type labels
- No marker-intensity data are available (e.g., only pre-computed cell labels or morphology-only
  segmentation), since gating requires raw per-cell intensity values
tools:
- name: SpatialCells
  role: Provides setGates/setGate (spatialcells.prep) for thresholding marker intensities
    and defining cell type labels; open-source package for region-based exploratory
    analysis of TMEs
  repo: https://semenovlab.github.io/SpatialCells
- name: MCMICRO
  role: 'Upstream preprocessing: transforms multi-channel whole-slide images into
    single-cell data; its output is the input to SpatialCells'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:48a1bb0bd3ef01e52dd3b58f9fb5723ee689d3295ca1823e68258ecaf2538838
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/SemenovLab/SpatialCells
  tools:
  - MCMICRO
  - SpatialCells
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# marker-gated-cell-phenotyping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Defines per-cell phenotype labels on multiplexed single-cell data by applying user-specified marker-intensity gates with SpatialCells' setGates/setGate (e.g., calling SOX10+ tumor cells and Ki67+/SOX10+ proliferating cells in 30-marker CyCIF data), giving downstream region-based TME analyses a marker-defined cell-type basis.

## When to use

Use when the input is per-cell x/y coordinates plus multi-marker intensity values (e.g., MCMICRO output from a CyCIF whole-slide image in an AnnData object) and the analysis goal is to label each cell by marker-defined type (e.g., SOX10+ tumor cells) before spatial/region analyses.

## When NOT to use

- Input is spot-level or bulk data rather than per-cell multiplexed imaging intensities — e.g., the Visium lymph node dataset, which the paper deconvolutes with Cell2location before SpatialCells analysis rather than gating single-cell markers
- The task is spatial region delineation or per-cell region assignment, not phenotype calling — those are handled by getBoundary and assignPointsToRegions, which require already-gated cell-type labels
- No marker-intensity data are available (e.g., only pre-computed cell labels or morphology-only segmentation), since gating requires raw per-cell intensity values

## Inputs

- AnnData object with per-cell x/y spatial coordinates and multi-channel marker intensity columns (MCMICRO output; MEL1 example: 1,110,585 cells, 30 antibody markers, CyCIF imaging)
- User-defined gate thresholds per marker (e.g., intensity cutoff distinguishing SOX10+ vs SOX10- cells)

## Outputs

- Per-cell cell-type labels stored in the AnnData object (e.g., SOX10+ tumor cells, Ki67+/SOX10+ proliferating cells)
- Marker-gated cell populations used as input to spatial.community/boundary construction (getCommunities, getBoundary, assignPointsToRegions)

## How to apply

Load the single-cell data produced by MCMICRO (which transforms multi-channel whole-slide images into single-cell data) into an AnnData object; for the MEL1 cutaneous melanoma sample this is ~1,110,585 cells with 30 antibody markers from CyCIF imaging. Call spatialcells.prep.setGates/setGates to threshold marker intensities with user-defined gates and convert them into cell type labels such as SOX10+ tumor cells or Ki67+/SOX10+ proliferating tumor cells. Rationale: SpatialCells is designed to preprocess and analyze data in a standardized manner with user-defined parameters, so gates are set explicitly by the analyst rather than learned automatically. Apply the gates uniformly across the whole ROI before downstream spatial steps, since region construction (getCommunities/getBoundary) and region assignment (assignPointsToRegions) operate on these marker-defined cell populations. Keep the gate thresholds recorded with the analysis so the labeling step is reproducible and consistent between marker-gating and region-annotation outputs.

## Related tools

- **SpatialCells** (Provides setGates/setGate (spatialcells.prep) for thresholding marker intensities and defining cell type labels; open-source package for region-based exploratory analysis of TMEs) — https://semenovlab.github.io/SpatialCells
- **MCMICRO** (Upstream preprocessing: transforms multi-channel whole-slide images into single-cell data; its output is the input to SpatialCells)

## Examples

```
from spatialcells.prep import setGates; setGates(adata)
```

## Evaluation signals

- Every cell in the AnnData object receives exactly one marker-derived cell-type label after setGates is run (no unclassified cells in the gated columns)
- Gated populations reproduce known marker relationships in the sample — e.g., in MEL1, Ki67+/SOX10+ (proliferating tumor, MPI = 1) cells were 7.4% overall and enriched along the main tumor periphery and invasive front
- Gate thresholds are documented and applied identically to all cells in the ROI, consistent with SpatialCells' standardized, user-defined-parameter preprocessing
- The same gated populations feed downstream region construction (getCommunities, getBoundary) and region assignment (assignPointsToRegions), so labels are consistent between the cell-type annotation and the region annotation outputs

## Limitations

- SpatialCells can currently only be installed from source (no packaged release); installation requires cloning the repository and installing via pip/conda.yaml
- Gates are user-defined parameters with no automatic threshold optimization described; mis-set gates will propagate into region-based analyses
- No software version or changelog is stated for SpatialCells, complicating exact reproducibility of gating behavior across code revisions

## Evidence

- [other] SOX10+ marker-defined population used for region delineation: [evidence span withheld — anchor status: not_found]
