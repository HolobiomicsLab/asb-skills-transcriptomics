---
name: spatial-region-annotation-of-single-cell-data
description: Use when a multiplexed single-cell dataset (e.g., MCMICRO output with
  per-cell coordinates and phenotype markers such as SOX10+ tumor and CD3D+ immune
  cells) must be partitioned into spatial subregions to quantify region composition
  or tumor-immune spatial relationships.
when_to_use_negative:
- Input is a raw multi-channel whole-slide image that has not yet been converted to
  single-cell data — preprocess with MCMICRO first, as its output is the input of
  SpatialCells
- The goal is neighborhood-graph construction and spot-level analysis (e.g., Visium
  data) rather than region-based annotation of single-cell imaging data — Squidpy
  specializes in neighborhood graph construction and analysis
- The data lack spatial coordinates or usable phenotype gates (e.g., no SOX10/CD3D
  markers) for defining tumor and immune regions
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: SpatialCells
  role: Provides getCommunities, getBoundary, bufferBoundary, and assignPointsToRegions
    to delineate and assign spatial subregions, plus msmt functions (getRegionComposition,
    getMinCellTypesDistance) for region metrics
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide images into
    the single-cell data consumed by SpatialCells
- name: AnnData
  role: Container format for the per-cell region annotations and coordinates loaded
    before region assignment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:033bcdc6ba562188cc7d35203eeb0b6dfe708c10f1571539c5b90f78a3fa1e76
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spatial-region-annotation-of-single-cell-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partition the overall region of interest (ROI) of a multiplexed single-cell sample (e.g., the MEL1 CyCIF melanoma sample with 1,110,585 cells) into spatial subregions such as immune-isolated and immune-rich areas, using SpatialCells' community clustering, boundary delineation, and region-assignment functions. This enables region-based downstream metrics like the tumor isolation index (46.9% for MEL1).

## When to use

Use when a multiplexed single-cell dataset (e.g., MCMICRO output with per-cell coordinates and phenotype markers such as SOX10+ tumor and CD3D+ immune cells) must be partitioned into spatial subregions to quantify region composition or tumor-immune spatial relationships.

## When NOT to use

- Input is a raw multi-channel whole-slide image that has not yet been converted to single-cell data — preprocess with MCMICRO first, as its output is the input of SpatialCells
- The goal is neighborhood-graph construction and spot-level analysis (e.g., Visium data) rather than region-based annotation of single-cell imaging data — Squidpy specializes in neighborhood graph construction and analysis
- The data lack spatial coordinates or usable phenotype gates (e.g., no SOX10/CD3D markers) for defining tumor and immune regions

## Inputs

- AnnData object with per-cell spatial coordinates from multiplexed imaging (e.g., CyCIF with 30 antibody markers, preprocessed via MCMICRO)
- Per-cell phenotype annotations/gates for markers such as SOX10 (tumor) and CD3D (immune), e.g., via spatialcells.prep.setGate/setGates
- Immune-cell coordinates for community clustering and boundary delineation

## Outputs

- Subregion assignment table labeling each cell as immune-isolated, immune-rich, or outside main tumor area
- Region-level indices, e.g., tumor isolation index as a single numeric value (MEL1: 46.9%) written to tumor_isolation_index.csv
- Region composition summaries from spatialcells.msmt.getRegionComposition

## How to apply

Load the AnnData object carrying per-cell region annotations and tumor (SOX10+) and immune (CD3D+) cell coordinates, preprocessing having been done with MCMICRO. Use spatialcells.spatial.getCommunities to cluster immune cells into spatial communities, then spatialcells.spatial.getBoundary (with bufferBoundary for margin buffers) to delineate polygon boundaries around immune subregions. Call spatialcells.spatial.assignPointsToRegions to partition the overall ROI into subregions defined by CD3D+ presence — the immune-isolated region and the immune-rich region — while excluding the fragmented SOX10+ areas outside the main tumor area, since including fragments with intervening empty spaces would artificially increase distance measurements. Then use spatialcells.msmt.getMinCellTypesDistance and getRegionComposition to compute region-level indices such as the tumor isolation index (percentage of tumor cells in the immune-isolated region).

## Related tools

- **SpatialCells** (Provides getCommunities, getBoundary, bufferBoundary, and assignPointsToRegions to delineate and assign spatial subregions, plus msmt functions (getRegionComposition, getMinCellTypesDistance) for region metrics) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide images into the single-cell data consumed by SpatialCells)
- **AnnData** (Container format for the per-cell region annotations and coordinates loaded before region assignment)

## Examples

```
from spatialcells.spatial import getCommunities, getBoundary, assignPointsToRegions; pts = assignPointsToRegions(adata, boundaries, regions=['immune-isolated', 'immune-rich'])
```

## Evaluation signals

- Validation: the recomputed tumor isolation index must agree with the reported MEL1 value of 46.9%
- Every cell in the ROI should receive a subregion assignment from assignPointsToRegions (immune-isolated, immune-rich, or excluded fragmented area)
- Fragmented SOX10+ areas outside the main tumor area must be excluded, otherwise minimum-distance measurements between cell types are artificially increased
- The index must equal the percentage of SOX10+ tumor cells falling within the CD3D-defined immune-isolated region, as computed with getRegionComposition/getMinCellTypesDistance
- Region assignment depends on sensible bufferBoundary margins on getBoundary polygons; boundaries should tightly cover the immune communities

## Limitations

- SpatialCells can currently only be installed from source (pip install -r requirements.txt; pip install ., or conda env create --name spatialcells --file=conda.yaml); no software version is stated
- Including separate fragments of SOX10+ cells together with intervening empty spaces artificially increases distance measurements, so fragmented areas must be excluded before region annotation
- No changelog is available for the repository

## Evidence

- [results] ROI subregions: [evidence span withheld — anchor status: not_found]
