---
name: cell2location-deconvolved-data-handling
description: Use when the input is a Visium spatial transcriptomics dataset deconvoluted
  with Cell2location (spot-level cell-type compositions in AnnData) and the goal is
  SpatialCells region-based analysis, e.g. community clustering and boundary delineation.
when_to_use_negative:
- Input is multiplexed imaging single-cell data (e.g. CyCIF/MCMICRO output), not spot-level
  Visium data deconvoluted with Cell2location — the MCMICRO single-cell output is
  the input path for that modality instead.
- The data have not been deconvoluted with Cell2location, so there is no spot-level
  cell-type composition for getCommunities to cluster on.
- The goal is generic neighborhood graph analysis rather than region-based community/boundary
  delineation, where a toolkit like Squidpy may be more appropriate.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: SpatialCells
  role: Consumes the Cell2location-deconvoluted AnnData; provides getCommunities,
    getBoundary, assignPointsToRegions, and exportRegion for community clustering,
    boundary delineation, region assignment, and annotation export.
  repo: https://github.com/SemenovLab/SpatialCells
- name: Cell2location
  role: Deconvolves the Visium lymph node data to spot-level cell-type compositions
    prior to SpatialCells analysis.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:f471ef7fe6c928a1c807f23b2cc458dce70fb615ab6638dd17f8e42ef162652b
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
  - Cell2location
  - SpatialCells
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cell2location-deconvolved-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Handles Visium spatial transcriptomics data that have been deconvoluted with the standard Cell2location protocol, loading them into an AnnData object so that SpatialCells can cluster spots into spatial communities and delineate region boundaries such as germinal centers.

## When to use

Use when the input is a Visium spatial transcriptomics dataset deconvoluted with Cell2location (spot-level cell-type compositions in AnnData) and the goal is SpatialCells region-based analysis, e.g. community clustering and boundary delineation.

## When NOT to use

- Input is multiplexed imaging single-cell data (e.g. CyCIF/MCMICRO output), not spot-level Visium data deconvoluted with Cell2location — the MCMICRO single-cell output is the input path for that modality instead.
- The data have not been deconvoluted with Cell2location, so there is no spot-level cell-type composition for getCommunities to cluster on.
- The goal is generic neighborhood graph analysis rather than region-based community/boundary delineation, where a toolkit like Squidpy may be more appropriate.

## Inputs

- Visium spatial transcriptomics data deconvoluted with the standard Cell2location protocol
- Cell2location-deconvoluted spot-level data in AnnData format (the framework used by SpatialCells)

## Outputs

- Spot-level spatial community labels from spatialcells.spatial.getCommunities
- Community region boundaries from spatialcells.spatial.getBoundary (optionally bufferBoundary)
- Per-spot region assignments from spatialcells.spatial.assignPointsToRegions
- Spot-level region annotation CSV (e.g. gc_annotations.csv) from spatialcells.utils.exportRegion

## How to apply

First deconvolve the Visium data following the standard Cell2location protocol, since the paper states the lymph node data were 'deconvoluted following the standard Cell2location protocol and analyzed with SpatialCells'. Load the Cell2location-deconvoluted data into an AnnData object, the framework used by SpatialCells. Run spatialcells.spatial.getCommunities to cluster spots into spatial communities based on their deconvolved cell-type composition, then use spatialcells.spatial.getBoundary (with bufferBoundary as needed) to delineate community boundaries, and spatialcells.spatial.assignPointsToRegions to assign every spot to a labeled region (e.g. germinal-center communities). Export the spot-level annotations with spatialcells.utils.exportRegion (e.g. to gc_annotations.csv) for downstream comparison against manual annotations.

## Related tools

- **SpatialCells** (Consumes the Cell2location-deconvoluted AnnData; provides getCommunities, getBoundary, assignPointsToRegions, and exportRegion for community clustering, boundary delineation, region assignment, and annotation export.) — https://github.com/SemenovLab/SpatialCells
- **Cell2location** (Deconvolves the Visium lymph node data to spot-level cell-type compositions prior to SpatialCells analysis.)

## Examples

```
conda env create --name spatialcells --file=conda.yaml; python -c "import spatialcells.spatial as sp; sp.getCommunities(adata); bnd = sp.getBoundary(adata); sp.assignPointsToRegions(adata, bnd)"
```

## Evaluation signals

- gc_annotations.csv exists and contains one annotation entry per spot (no spots dropped during exportRegion).
- Predicted spot-level germinal-center labels overlap the manual germinal-center annotation qualitatively consistent with the agreement reported for the Visium lymph node analysis.
- getCommunities produces spatial communities whose membership reflects the deconvolved cell-type composition (contiguous spatial clusters, not scattered spots).
- getBoundary delineates closed region boundaries for each community, and bufferBoundary applied where boundary expansion is needed before assignPointsToRegions.
- Every spot is assigned to exactly one region after assignPointsToRegions.

## Limitations

- SpatialCells can currently only be installed from source (download code, pip install -r requirements.txt; pip install .).
- No software version is stated for SpatialCells in the article or repository documentation, and no changelog is available, complicating reproducibility.
- Validation of germinal-center boundaries is against the authors' manual annotation; consistency is reported qualitatively rather than via a numeric agreement metric in this context.

## Evidence
