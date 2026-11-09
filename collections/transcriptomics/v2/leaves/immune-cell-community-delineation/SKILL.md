---
name: immune-cell-community-delineation
description: Use when analyzing CyCIF/MCMICRO-derived single-cell data with marker
  annotations (e.g., CD3D+, SOX10+) and coordinates, to define immune subregions and
  quantify tumor cell localization relative to immune infiltrates.
when_to_use_negative:
- When only cell-level phenotyping or neighborhood graph analysis is needed (e.g.,
  Squidpy-style adjacency), not ROI-level region delineation
- When the image has not been converted to single-cell data — SpatialCells expects
  the output of MCMICRO, not raw multi-channel whole-slide images
- When manual annotation-based spatial analysis is acceptable and hand-curated regions
  suffice (SpatialCells targets automated, scalable region-based profiling)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_2229
tools:
- name: SpatialCells
  role: 'Core package: getCommunities clusters immune cells, getBoundary/bufferBoundary
    delineates polygon boundaries, assignPointsToRegions partitions the ROI, msmt
    functions quantify composition and distances'
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide images into
    the single-cell data used as SpatialCells input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:d114125b023f15f75d4e55943dd2f62c24b7dc48d54f743afdabcef49fa0dbc2
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
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# immune-cell-community-delineation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Delineates spatial communities of immune cells (e.g., CD3D+ T cells) from multiplexed single-cell imaging data and uses their polygon boundaries to partition the ROI into immune-rich and immune-isolated subregions, enabling metrics such as the tumor isolation index.

## When to use

Use when analyzing CyCIF/MCMICRO-derived single-cell data with marker annotations (e.g., CD3D+, SOX10+) and coordinates, to define immune subregions and quantify tumor cell localization relative to immune infiltrates.

## When NOT to use

- When only cell-level phenotyping or neighborhood graph analysis is needed (e.g., Squidpy-style adjacency), not ROI-level region delineation
- When the image has not been converted to single-cell data — SpatialCells expects the output of MCMICRO, not raw multi-channel whole-slide images
- When manual annotation-based spatial analysis is acceptable and hand-curated regions suffice (SpatialCells targets automated, scalable region-based profiling)

## Inputs

- AnnData object with per-cell region annotations, cell-type/marker gates (CD3D+, SOX10+) and spatial coordinates
- MCMICRO output single-cell data from multiplexed (CyCIF, 30-marker) whole-slide images

## Outputs

- Immune cell spatial community labels and polygon boundaries per community
- Per-cell subregion assignment table (immune-rich vs immune-isolated)
- Region composition statistics and metrics such as the tumor isolation index (e.g., 46.9% for MEL1)

## How to apply

Starting from the per-cell AnnData object (e.g., the 1,110,585-cell MEL1 CyCIF sample), first cluster the immune (CD3D+) cells into spatial communities with spatialcells.spatial.getCommunities. Then, for each community, use spatialcells.spatial.getBoundary (with bufferBoundary for margin buffers) to construct the polygon that encloses the immune cells. Use spatialcells.spatial.assignPointsToRegions to assign every point in the overall ROI to a subregion defined by the presence of CD3D+ cells — an immune-rich region and an immune-isolated region — excluding fragmented SOX10+ areas outside the main tumor area so distances are not artificially increased. Downstream, spatialcells.msmt.getRegionComposition and spatialcells.msmt.getMinCellTypesDistance quantify composition and tumor-immune proximity, e.g., the tumor isolation index as the percentage of tumor cells in the immune-isolated region (46.9% for MEL1). Install SpatialCells from source (pip install -r requirements.txt; pip install .) or via conda (conda env create --name spatialcells --file=conda.yaml).

## Related tools

- **SpatialCells** (Core package: getCommunities clusters immune cells, getBoundary/bufferBoundary delineates polygon boundaries, assignPointsToRegions partitions the ROI, msmt functions quantify composition and distances) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide images into the single-cell data used as SpatialCells input)

## Examples

```
from spatialcells.spatial import getCommunities, getBoundary, assignPointsToRegions; communities = getCommunities(adata[adata.obs.CD3D_positive], columns=['CD3D']); boundary = getBoundary(communities, bufferBoundary=True)
```

## Evaluation signals

- Recomputed tumor isolation index for MEL1 must agree with the reported 46.9% (percentage of tumor cells in the immune-isolated region)
- Every point in the ROI receives a subregion label from assignPointsToRegions with no unassigned cells
- Fragmented SOX10+ areas outside the main tumor area are excluded before distance measurements, otherwise distances are artificially increased
- Community boundaries from getBoundary form closed polygons around CD3D+ communities, optionally with bufferBoundary margins
- Region composition output (getRegionComposition) partitions cells into the immune-rich and immune-isolated subregions consistently with CD3D+ presence

## Limitations

- SpatialCells can currently only be installed from source (pip/conda), no packaged release
- No software version stated for SpatialCells; no changelog found for the repository
- Including separate fragments of SOX10+ cells with intervening empty spaces artificially increases distance measurements, so fragmented areas must be excluded
- HALO-like manual annotation is avoided, but boundary delineation still depends on user-defined parameters and buffer choices

## Evidence

- [results] ROI partition method: [evidence span withheld — anchor status: not_found]
