---
name: tumor-microenvironment-region-partitioning
description: Use when a multiplexed imaging dataset (e.g., CyCIF MEL1, ~1.1M cells)
  has per-cell markers like SOX10+ (tumor) and CD3D+ (immune), and the goal is to
  split the ROI into tumor/immune subregions or compute a tumor isolation index.
when_to_use_negative:
- Input is raw multi-channel whole-slide image data not yet converted to single-cell
  form — preprocess with MCMICRO first, since SpatialCells' input is MCMICRO's output
- The analysis goal is neighborhood-graph construction rather than region-based partitioning
  — Squidpy specializes in neighborhood graph construction and analysis
- Spatial transcriptomics needing deconvolution before region analysis — deconvolute
  first (e.g., Cell2location) before applying SpatialCells
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_2640
tools:
- name: SpatialCells
  role: Provides getCommunities, getBoundary, assignPointsToRegions, getMinCellTypesDistance,
    and getRegionComposition for partitioning the ROI and computing the tumor isolation
    index
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide images into
    the single-cell data consumed by SpatialCells
- name: AnnData
  role: Container format for the per-cell region annotations and marker gates used
    as region-partitioning input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:b98efdc00f8d19bae279d1f74787cadc0066293ce32fd900e46828fa2e6749ca
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
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tumor-microenvironment-region-partitioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partitions the region of interest (ROI) of a multiplexed single-cell imaging sample into spatial subregions — e.g., immune-isolated vs immune-rich — using SpatialCells' community clustering and boundary polygon functions, and quantifies regional metrics such as the tumor isolation index.

## When to use

Use when a multiplexed imaging dataset (e.g., CyCIF MEL1, ~1.1M cells) has per-cell markers like SOX10+ (tumor) and CD3D+ (immune), and the goal is to split the ROI into tumor/immune subregions or compute a tumor isolation index.

## When NOT to use

- Input is raw multi-channel whole-slide image data not yet converted to single-cell form — preprocess with MCMICRO first, since SpatialCells' input is MCMICRO's output
- The analysis goal is neighborhood-graph construction rather than region-based partitioning — Squidpy specializes in neighborhood graph construction and analysis
- Spatial transcriptomics needing deconvolution before region analysis — deconvolute first (e.g., Cell2location) before applying SpatialCells

## Inputs

- AnnData object of multiplexed single-cell data (e.g., MCMICRO output of a CyCIF whole-slide image with 30 antibody markers), with per-cell coordinates
- Per-cell phenotype gates, e.g., SOX10+ for tumor cells and CD3D+ for immune cells (set via spatialcells.prep.setGate/setGates)
- Region annotations from prior tasks (e.g., main tumor area definition with fragmented SOX10+ areas flagged)

## Outputs

- Per-cell subregion assignment table (immune-isolated vs immune-rich region labels), e.g., written to tumor_isolation_index.csv
- Polygon boundary objects for immune subregions from getBoundary
- Scalar regional metrics such as the tumor isolation index (percentage of tumor cells in the immune-isolated region)

## How to apply

Load the per-cell AnnData object (e.g., MEL1 with 1,110,585 cells) carrying marker gates for SOX10+ tumor and CD3D+ immune cells. Cluster immune cells into spatial communities with spatialcells.spatial.getCommunities, then delineate polygon boundaries for the immune subregions with spatialcells.spatial.getBoundary (optionally bufferBoundary to add margin buffers). Use spatialcells.spatial.assignPointsToRegions to divide the overall ROI into two subregions based on CD3D+ presence: immune-rich and immune-isolated. Exclude fragmented SOX10+ areas outside the main tumor area first, since including separate fragments with intervening empty spaces artificially inflates distance measurements. Then compute the tumor isolation index as the percentage of tumor cells in the immune-isolated region (reported value for MEL1: 46.9%) using getMinCellTypesDistance and region-composition functions.

## Related tools

- **SpatialCells** (Provides getCommunities, getBoundary, assignPointsToRegions, getMinCellTypesDistance, and getRegionComposition for partitioning the ROI and computing the tumor isolation index) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide images into the single-cell data consumed by SpatialCells)
- **AnnData** (Container format for the per-cell region annotations and marker gates used as region-partitioning input)

## Examples

```
import spatialcells.spatial as sp; communities = sp.getCommunities(immune_cells); boundary = sp.getBoundary(communities, 'CD3D+', buffer=100); sp.assignPointsToRegions(adata, [boundary], ['immune_rich', 'immune_isolated'])
```

## Evaluation signals

- Recomputed tumor isolation index for MEL1 must agree with the reported 46.9% (percentage of tumor cells in the immune-isolated region)
- Every cell in the ROI receives exactly one subregion label (immune-isolated or immune-rich) from assignPointsToRegions
- Fragmented SOX10+ areas outside the main tumor area are excluded, so distance measurements are not artificially increased
- Subregion assignment table exports cleanly to tumor_isolation_index.csv with the single numeric index matching validation target

## Limitations

- SpatialCells can currently only be installed from source (download code from the repository and pip install)
- If fragmented SOX10+ cell areas and intervening empty spaces are included, measurements would be artificially increased — exclusion of fragments is required
- No software version stated for SpatialCells; installation requires a conda environment from conda.yaml or requirements.txt

## Evidence

- [results] ROI divided into two subregions based on CD3D+ presence: [evidence span withheld — anchor status: not_found]
- [readme] README install instructions: [evidence span withheld — anchor status: not_found]
