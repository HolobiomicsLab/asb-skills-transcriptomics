---
name: cell-to-region-assignment
description: Use when single-cell coordinates with marker intensities (e.g. MCMICRO-processed
  CyCIF data in an AnnData object) must be annotated with region labels derived from
  marker-defined cell populations like SOX10+ tumor cells.
when_to_use_negative:
- Data are not spatially resolved single-cell coordinates (e.g. bulk or plate-based
  readouts with no x/y positions) — boundary construction and point-to-region assignment
  have no geometric input
- Cells are already annotated with region labels, so the goal is downstream measurement
  rather than assignment — go directly to spatialcells.msmt functions like getRegionComposition
  or getMPI
- The aim is neighborhood graph analysis of spatial data rather than region delineation
  — the paper positions Squidpy as the specialized toolkit for neighborhood graph
  construction and analysis
tools:
- name: SpatialCells
  role: 'Provides the region-definition pipeline: getCommunities clusters marker-defined
    cells, getBoundary/getShrunkenBoundary/getExtendedBoundary build tumor and nested
    region boundaries, and spatial.assignPointsToRegions assigns each cell to one
    named region'
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide CyCIF images
    into the single-cell data whose output is the input of SpatialCells
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:36bc56ef1f224ff1e7aa7daea1c2a980d87088cf3ecc24e1921ae0992af284a7
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cell-to-region-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns each of the ~1.1M single cells in an ROI to exactly one named spatial region using SpatialCells' boundary-derived regions (getBoundary/assignPointsToRegions), enabling region-based TME measurements such as area, cell density, region composition, and tumor isolation index.

## When to use

Use when single-cell coordinates with marker intensities (e.g. MCMICRO-processed CyCIF data in an AnnData object) must be annotated with region labels derived from marker-defined cell populations like SOX10+ tumor cells.

## When NOT to use

- Data are not spatially resolved single-cell coordinates (e.g. bulk or plate-based readouts with no x/y positions) — boundary construction and point-to-region assignment have no geometric input
- Cells are already annotated with region labels, so the goal is downstream measurement rather than assignment — go directly to spatialcells.msmt functions like getRegionComposition or getMPI
- The aim is neighborhood graph analysis of spatial data rather than region delineation — the paper positions Squidpy as the specialized toolkit for neighborhood graph construction and analysis

## Inputs

- AnnData object of single-cell x/y coordinates and per-cell marker intensities (MCMICRO output from CyCIF imaging, e.g. MEL1 with 30 antibody markers and 1,110,585 cells)
- User-defined marker gates from spatialcells.preprocessing.setGates/setGate defining cell type labels (e.g. SOX10+ tumor cells, CD3D+ cells)
- Boundary objects from spatialcells.spatial.getBoundary (with getShrunkenBoundary/getExtendedBoundary and bufferBoundary for nested regions)

## Outputs

- Per-cell region label assignment where every cell receives exactly one named region (cell_region_annotations.csv)
- Boundary-derived regions that partition the overall ROI into subregions (e.g. immune-isolated and immune-rich), usable by spatialcells.msmt functions (getRegionComposition, getCellKDE, getMPI, getMinCellTypesDistance)

## How to apply

Load the MCMICRO single-cell output (x/y coordinates plus 30-marker intensities for MEL1 CyCIF) into an AnnData object and use spatialcells.preprocessing.setGates to threshold marker intensities into cell type labels (e.g. SOX10+ tumor cells). Run spatialcells.spatial.getCommunities to cluster marker-defined cells into spatial communities, then construct the tumor boundary with spatialcells.spatial.getBoundary (using getShrunkenBoundary/getExtendedBoundary and bufferBoundary for nested regions). Apply the paper's filter step: exclude fragmented SOX10+ areas so only the main tumor area is delineated, since including separate fragments and intervening empty spaces would artificially increase distance measurements. Finally run spatialcells.spatial.assignPointsToRegions to label each cell with exactly one named region (e.g. dividing the ROI into immune-isolated vs. immune-rich subregions based on CD3D+ presence) and export the labels for downstream msmt functions like getRegionComposition, getMinCellTypesDistance, and getMPI.

## Related tools

- **SpatialCells** (Provides the region-definition pipeline: getCommunities clusters marker-defined cells, getBoundary/getShrunkenBoundary/getExtendedBoundary build tumor and nested region boundaries, and spatial.assignPointsToRegions assigns each cell to one named region) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide CyCIF images into the single-cell data whose output is the input of SpatialCells)

## Examples

```
from spatialcells.spatial import getCommunities, getBoundary, assignPointsToRegions; assignPointsToRegions(adata, boundaries, region_col='region')
```

## Evaluation signals

- Every cell in the AnnData object receives exactly one non-null region label — no cell is left unassigned and no cell is double-assigned
- The boundary-derived regions partition the overall ROI, so regional cell counts sum to the total cell count (1,110,585 cells for MEL1)
- Fragmented SOX10+ areas outside the main tumor are excluded from the delineated boundary; verify distance-based measurements are not artificially increased by intervening empty spaces
- Downstream sanity metrics match paper magnitudes: tumor isolation index of 46.9% (percentage of tumor cells in the immune-isolated region) and 7.4% of SOX10+ cells with MPI = 1 for MEL1
- Region composition statistics (e.g. proportions of Ki67+/SOX10+ cells) show enrichment at the tumor periphery and advancing edge of the invasive front, consistent with the paper's biological expectations

## Limitations

- Fragmented marker-positive areas must be manually excluded; if included with intervening empty spaces, distance measurements would be artificially increased
- SpatialCells can currently only be installed from source; no software version or changelog is stated, so reproducibility depends on pinning the repository commit
- Region definitions depend on user-defined marker gates (setGates) and boundary parameters (shrunken/extended boundaries, buffer), so results are sensitive to these choices and the package 'allows preprocessing and analyzing data in a standardized manner with user-defined parameters'

## Evidence

- [results] the overall ROI is divided into two distinct subregions based on the presence of CD3D+ cells: (1) immune-isolated region... and (2) immune-rich region: [evidence span withheld — anchor status: not_found]
