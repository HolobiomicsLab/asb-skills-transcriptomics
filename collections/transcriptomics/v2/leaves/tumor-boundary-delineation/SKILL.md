---
name: tumor-boundary-delineation
description: Use when multiplexed single-cell data with x/y coordinates (e.g., MCMICRO-processed
  CyCIF output) needs region-based analysis requiring tumor/stromal region boundaries
  and per-cell region labels.
when_to_use_negative:
- Input lacks spatial coordinates (no x/y per cell) or is already a per-cell region/feature
  table — boundary construction from point coordinates is meaningless
- The goal is cell-phenotyping or neighborhood-graph analysis rather than region-based
  delineation (e.g., Squidpy specializes in neighborhood graph construction and analysis)
- No user-specified marker-defined cell population is available to anchor the boundary
  (e.g., no SOX10+ tumor cell gate set via setGates)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SpatialCells
  role: Provides getBoundary, getShrunkenBoundary/getExtendedBoundary, bufferBoundary,
    and assignPointsToRegions to construct tumor boundaries and assign each cell to
    one region
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide images (e.g.,
    CyCIF) into the single-cell data that is the input of SpatialCells
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:44a0783bc888bbd3596a67d25442fcdfb6ecd26e9f60eaeb82c023a0b178bd0f
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
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tumor-boundary-delineation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Delineates tumor regional boundaries from marker-defined cell populations (e.g., SOX10+ melanoma cells) using SpatialCells' getBoundary/getShrunkenBoundary/getExtendedBoundary, then assigns every cell to exactly one named region via assignPointsToRegions for region-based TME analysis.

## When to use

Use when multiplexed single-cell data with x/y coordinates (e.g., MCMICRO-processed CyCIF output) needs region-based analysis requiring tumor/stromal region boundaries and per-cell region labels.

## When NOT to use

- Input lacks spatial coordinates (no x/y per cell) or is already a per-cell region/feature table — boundary construction from point coordinates is meaningless
- The goal is cell-phenotyping or neighborhood-graph analysis rather than region-based delineation (e.g., Squidpy specializes in neighborhood graph construction and analysis)
- No user-specified marker-defined cell population is available to anchor the boundary (e.g., no SOX10+ tumor cell gate set via setGates)

## Inputs

- AnnData object with per-cell x/y coordinates and 30-marker intensity matrix (MCMICRO output from CyCIF imaging, e.g., MEL1 melanoma, ~1.1M cells)
- Marker intensity gates/labels from spatialcells.preprocessing.setGates defining the boundary population (e.g., SOX10+ tumor cells)
- Spatial community assignments from spatialcells.spatial.getCommunities (used to select the main tumor community and exclude fragments)

## Outputs

- Tumor boundary polygon(s) from getBoundary, plus nested region boundaries from getShrunkenBoundary/getExtendedBoundary with bufferBoundary
- Per-cell region label annotation (cell_region_annotations.csv) where every cell has exactly one region assignment
- Region-partitioned ROI enabling downstream region-based measurements (area, cell density, composition)

## How to apply

Start from an AnnData object of single-cell x/y coordinates and marker intensities preprocessed with MCMICRO (e.g., MEL1 CyCIF sample with 30 antibody markers, 1,110,585 cells). Use spatialcells.preprocessing.setGates/setGate to threshold marker intensities and label cell types, then run spatialcells.spatial.getCommunities on the marker-defined population (e.g., SOX10+ tumor cells) to cluster them into spatial communities. Call getBoundary on the main tumor community, excluding fragmented SOX10+ areas so only the main tumor area is delineated; derive nested subregion boundaries with getShrunkenBoundary/getExtendedBoundary and bufferBoundary (e.g., immune-isolated vs immune-rich subregions split by CD3D+ presence). Run spatialcells.spatial.assignPointsToRegions to give each cell exactly one named region, then export labels to cell_region_annotations.csv.

## Related tools

- **SpatialCells** (Provides getBoundary, getShrunkenBoundary/getExtendedBoundary, bufferBoundary, and assignPointsToRegions to construct tumor boundaries and assign each cell to one region) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide images (e.g., CyCIF) into the single-cell data that is the input of SpatialCells)

## Examples

```
from spatialcells.spatial import getBoundary, assignPointsToRegions; boundary = getBoundary(); assignPointsToRegions(adata, boundaries)
```

## Evaluation signals

- Every cell in the AnnData object receives exactly one non-null region label after assignPointsToRegions (no cells assigned to zero or multiple regions)
- Boundary-derived regions partition the ROI: no unassigned leftover areas within the delineated region of interest
- Fragmented SOX10+ cell areas are excluded from the boundary so only the main tumor area is delineated (check that disconnected fragments are not inside the polygon)
- Exported cell_region_annotations.csv covers all ~1.1M cells with region labels consistent with the nested boundary structure (e.g., tumor core vs immune-isolated vs immune-rich subregions)
- Region-based downstream metrics are computable per region (area, cell density, composition) without undefined regions

## Limitations

- Fragmented marker-defined areas must be excluded; including separate fragments with intervening empty spaces would artificially increase distance measurements
- SpatialCells can currently only be installed from source (pip install -r requirements.txt; pip install .), and no software version or changelog is stated in the article/README
- Boundary derivation depends on user-defined gating parameters (setGates) and community structure, so boundaries are sensitive to the marker thresholds chosen for the anchor population

## Evidence
