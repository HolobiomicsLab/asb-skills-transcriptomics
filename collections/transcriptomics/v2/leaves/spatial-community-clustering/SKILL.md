---
name: spatial-community-clustering
description: Use when single-cell x/y coordinates and marker intensities (e.g., MCMICRO-processed
  CyCIF data) are available and marker-positive populations must be grouped into spatial
  communities to delineate tumor boundaries and nested regions.
when_to_use_negative:
- Input is raw multi-channel whole-slide imagery that has not yet been segmented into
  single cells — run MCMICRO first, since its output is the input of SpatialCells
- Marker gates have not been set (spatialcells.prep.setGates/setGates) — without thresholded
  marker labels there is no marker-defined cell population to cluster
- Analysis goal is graph-based neighborhood analysis without region delineation (e.g.,
  Squidpy specializes in neighborhood graph construction and analysis)
edam_operation: http://edamontology.org/operation_3432
tools:
- name: SpatialCells
  role: Provides spatial.getCommunities (and getBoundary, assignPointsToRegions) to
    cluster marker-defined cells into spatial communities and assign per-cell region
    labels
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide images into
    the single-cell data used as input for community clustering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:a35953bc8aff0cdd4925cd1e211521a6ba0bdc04ad0730011882885c43c11aad
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# spatial-community-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters marker-defined cells (e.g., SOX10+ tumor cells from CyCIF multiplexed imaging) into spatial communities using SpatialCells' spatial module, providing the community structure from which region boundaries are derived for region-based TME analysis.

## When to use

Use when single-cell x/y coordinates and marker intensities (e.g., MCMICRO-processed CyCIF data) are available and marker-positive populations must be grouped into spatial communities to delineate tumor boundaries and nested regions.

## When NOT to use

- Input is raw multi-channel whole-slide imagery that has not yet been segmented into single cells — run MCMICRO first, since its output is the input of SpatialCells
- Marker gates have not been set (spatialcells.prep.setGates/setGates) — without thresholded marker labels there is no marker-defined cell population to cluster
- Analysis goal is graph-based neighborhood analysis without region delineation (e.g., Squidpy specializes in neighborhood graph construction and analysis)

## Inputs

- AnnData object with single-cell x/y spatial coordinates and 30-marker intensity columns, produced by MCMICRO (e.g., MEL1 cutaneous melanoma CyCIF sample, 1110585 cells)
- Marker gate definitions from spatialcells.preprocessing.setGates that label cell populations (e.g., SOX10+ tumor cells, CD3D+ T cells)

## Outputs

- Spatial community labels on marker-defined cells, used to construct region boundaries via spatial.getBoundary
- Downstream per-cell region assignments exported to cell_region_annotations.csv (exactly one region label per cell)

## How to apply

Load the MCMICRO-processed single-cell data (e.g., MEL1 CyCIF sample with ~1.1M cells and 30 antibody markers) into an AnnData object, then run spatialcells.preprocessing.setGates to threshold marker intensities and define cell type labels such as SOX10+ tumor cells. Run spatialcells.spatial.getCommunities on the marker-defined cells to cluster them into spatial communities; these communities are the basis for spatial.getBoundary construction (with getShrunkenBoundary/getExtendedBoundary and bufferBoundary). Exclude fragmented SOX10+ areas during boundary derivation so only the main tumor area is delineated, since including separate fragments with intervening empty spaces would artificially increase downstream distance measurements. Finally, feed the derived region boundaries into spatialcells.spatial.assignPointsToRegions so every cell receives exactly one named region label (e.g., immune-isolated vs. immune-rich subregions based on CD3D+ presence).

## Related tools

- **SpatialCells** (Provides spatial.getCommunities (and getBoundary, assignPointsToRegions) to cluster marker-defined cells into spatial communities and assign per-cell region labels) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide images into the single-cell data used as input for community clustering)

## Examples

```
from spatialcells.spatial import getCommunities; communities = getCommunities(adata)
```

## Evaluation signals

- Every cell receives exactly one region label after spatial.spatial.assignPointsToRegions; validate cell_region_annotations.csv has no unassigned or multi-assigned cells
- Boundary-derived regions partition the ROI with no gaps or overlaps
- Fragmented SOX10+ areas are excluded so the delineated tumor boundary covers only the main tumor area, keeping distance measurements from being artificially increased
- Region-based downstream metrics (e.g., region composition, tumor isolation index such as the MEL1 46.9% value, MPI = 1 proliferating fraction of 7.4%) fall in plausible ranges for the sample
- Communities visually align with the marker-defined populations (e.g., SOX10+ tumor, CD3D+ immune) when overlaid on x/y coordinates

## Limitations

- Including fragmented marker-positive areas (e.g., separate SOX10+ fragments with intervening empty spaces) would artificially increase distance measurements, so fragments must be excluded
- SpatialCells can currently only be installed from source (pip install -r requirements.txt; pip install ., or conda env from conda.yaml)
- No software version or changelog is stated for the repository; preprocessing relies on user-defined parameters with no automatic gating

## Evidence
