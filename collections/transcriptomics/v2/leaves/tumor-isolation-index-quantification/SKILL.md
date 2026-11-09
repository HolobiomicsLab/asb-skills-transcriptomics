---
name: tumor-isolation-index-quantification
description: Use when you have spatially resolved single-cell coordinates with tumor
  and immune markers (e.g., SOX10+, CD3D+) from a multiplexed imaging sample and need
  the fraction of tumor cells in regions without immune infiltration.
when_to_use_negative:
- Input is unsegmented multi-channel whole-slide image data that has not yet been
  converted to single-cell form (run MCMICRO preprocessing first)
- The analysis goal is a per-cell proliferation score rather than a spatial region-based
  tumor/immune metric (use SpatialCells' getMPI instead)
- Only one cell type of interest is labeled and no immune marker (e.g., CD3D+) defines
  contrasting subregions, so no immune-isolated/immune-rich partition can be constructed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_2640
tools:
- name: SpatialCells
  role: Provides getCommunities, getBoundary/bufferBoundary, assignPointsToRegions,
    getMinCellTypesDistance, and region-composition functions used to delineate immune
    subregions and compute the isolation index
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide images into
    the single-cell data used as input to SpatialCells
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:b770904ba0800f99e24b77fae219c82aed424225b40edbf6180f52eed1d221c1
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# tumor-isolation-index-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the tumor isolation index — the percentage of tumor (SOX10+) cells located in immune-isolated regions lacking CD3D+ infiltration — using SpatialCells' community/boundary/region-assignment functions on multiplexed imaging single-cell data. Validated on the MEL1 melanoma sample (1,110,585 cells), where the index is 46.9%.

## When to use

Use when you have spatially resolved single-cell coordinates with tumor and immune markers (e.g., SOX10+, CD3D+) from a multiplexed imaging sample and need the fraction of tumor cells in regions without immune infiltration.

## When NOT to use

- Input is unsegmented multi-channel whole-slide image data that has not yet been converted to single-cell form (run MCMICRO preprocessing first)
- The analysis goal is a per-cell proliferation score rather than a spatial region-based tumor/immune metric (use SpatialCells' getMPI instead)
- Only one cell type of interest is labeled and no immune marker (e.g., CD3D+) defines contrasting subregions, so no immune-isolated/immune-rich partition can be constructed

## Inputs

- AnnData object with per-cell spatial coordinates and region annotations for a multiplexed imaging sample (e.g., MEL1, 1,110,585 cells)
- Per-cell phenotype labels for tumor (SOX10+) and immune (CD3D+) cells
- Single-cell data produced by MCMICRO preprocessing of multi-channel whole-slide CyCIF images

## Outputs

- Subregion assignment table (immune-isolated vs. immune-rich per cell)
- Tumor isolation index as a single numeric percentage (MEL1 reference value: 46.9%)
- tumor_isolation_index.csv containing the index value

## How to apply

Load the per-cell region-annotated AnnData object with tumor (SOX10+) and immune (CD3D+) cell coordinates (for MEL1, 1,110,585 CyCIF cells preprocessed with MCMICRO). Use spatialcells.spatial.getCommunities to cluster immune cells into spatial communities and getBoundary (with bufferBoundary for margin buffers) to delineate polygon boundaries for immune subregions. Use spatialcells.spatial.assignPointsToRegions to partition the overall ROI into immune-isolated and immune-rich subregions based on the presence of CD3D+ cells, excluding fragmented SOX10+ areas outside the main tumor area so that distance measurements are not artificially increased. Compute the isolation index as the percentage of tumor cells in the immune-isolated region using spatialcells.msmt.getMinCellTypesDistance and region-composition functions, then write the subregion assignments and the single numeric index to a CSV.

## Related tools

- **SpatialCells** (Provides getCommunities, getBoundary/bufferBoundary, assignPointsToRegions, getMinCellTypesDistance, and region-composition functions used to delineate immune subregions and compute the isolation index) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide images into the single-cell data used as input to SpatialCells)

## Examples

```
import spatialcells.spatial as sp; sp.getCommunities(immune_cells, markers=['CD3D+'], eps=40); boundary = sp.getBoundary(community, bufferBoundary=buffer); regions = sp.assignPointsToRegions(points, [boundary], region_col='region')
```

## Evaluation signals

- Recomputed MEL1 tumor isolation index must agree with the reported 46.9% (percentage of tumor cells in the immune-isolated region)
- Subregion assignment table must partition the overall ROI into exactly two subregions (immune-isolated and immune-rich) based on CD3D+ presence
- Fragmented SOX10+ areas outside the main tumor area must be excluded; including them would artificially increase distance measurements
- Output tumor_isolation_index.csv must contain a single numeric percentage value
- Every tumor and immune cell in the input should appear in exactly one region assignment

## Limitations

- If fragmented SOX10+ cell areas and intervening empty spaces are included, distance measurements are artificially increased
- SpatialCells can currently only be installed from source (pip install -r requirements.txt; pip install .)
- No software version or changelog is stated for SpatialCells, complicating reproducibility pinning

## Evidence
