---
name: spatial-boundary-delineation
description: Use when spatially resolved data (Cell2location-deconvolved Visium spots
  or multiplexed single-cell imaging such as CyCIF/MCMICRO output) must be partitioned
  into communities with explicit boundaries and per-point region labels.
when_to_use_negative:
- Input is a raw multi-channel whole-slide image without single-cell segmentation
  — preprocess with MCMICRO first, since 'The output of MCMICRO is the input of SpatialCells'
- Goal is general neighborhood graph construction/analysis rather than explicit region
  delineation — Squidpy is the cited toolkit for that purpose
- A HALO-style manual annotation workflow is preferred — SpatialCells is positioned
  as a replacement for approaches where spatial analysis 'often demands manual annotation'
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SpatialCells
  role: Primary package providing getCommunities, getBoundary/bufferBoundary, and
    assignPointsToRegions for community clustering, boundary computation, and region
    assignment
  repo: https://github.com/SemenovLab/SpatialCells
- name: Cell2location
  role: Upstream deconvolution of Visium spots into cell-type compositions that getCommunities
    clusters on
- name: MCMICRO
  role: Upstream preprocessing of multi-channel whole-slide images into single-cell
    data for the multiplexed-imaging use case
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:c2bd3392ef7b5cf4bc3bd3f8dee384f1086021d60a0691bdaf925d2694bb1585
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
  - MCMICRO
  - SpatialCells
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spatial-boundary-delineation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Delineates spatial regions (e.g., germinal centers in a Visium lymph node sample, tumor/immune subregions in CyCIF melanoma) by clustering spots/cells on deconvolved cell-type composition and computing explicit region boundaries with SpatialCells.

## When to use

Use when spatially resolved data (Cell2location-deconvolved Visium spots or multiplexed single-cell imaging such as CyCIF/MCMICRO output) must be partitioned into communities with explicit boundaries and per-point region labels.

## When NOT to use

- Input is a raw multi-channel whole-slide image without single-cell segmentation — preprocess with MCMICRO first, since 'The output of MCMICRO is the input of SpatialCells'
- Goal is general neighborhood graph construction/analysis rather than explicit region delineation — Squidpy is the cited toolkit for that purpose
- A HALO-style manual annotation workflow is preferred — SpatialCells is positioned as a replacement for approaches where spatial analysis 'often demands manual annotation'

## Inputs

- Cell2location-deconvolved spatial transcriptomics data stored as an AnnData object (e.g., public Visium human lymph node dataset)
- MCMICRO-processed multiplexed single-cell data (output of MCMICRO is the input of SpatialCells), e.g., CyCIF MEL1 melanoma sample with 30 antibody markers
- Optional manual region annotation for validation (e.g., authors' manual germinal-center annotation)

## Outputs

- Spatial community assignments per spot/cell (from getCommunities)
- Region boundary polygons (from getBoundary, optionally buffered via bufferBoundary)
- Per-point region labels (from assignPointsToRegions)
- Point-level region annotation table exported as CSV (e.g., gc_annotations.csv via spatialcells.utils.exportRegion)

## How to apply

Load the Cell2location-deconvolved spatial data into an AnnData object, then run spatialcells.spatial.getCommunities to cluster spots/cells into spatial communities based on their deconvolved cell-type composition. Delineate community extents with spatialcells.spatial.getBoundary, applying bufferBoundary when the boundary needs expansion around the community core. Use spatialcells.spatial.assignPointsToRegions to assign every spot/cell to a labeled region (e.g., germinal-center communities versus other lymph-node regions). Export the point-level labels with spatialcells.utils.exportRegion to a CSV such as gc_annotations.csv, then compare predicted labels against any manual annotation to check boundary consistency.

## Related tools

- **SpatialCells** (Primary package providing getCommunities, getBoundary/bufferBoundary, and assignPointsToRegions for community clustering, boundary computation, and region assignment) — https://github.com/SemenovLab/SpatialCells
- **Cell2location** (Upstream deconvolution of Visium spots into cell-type compositions that getCommunities clusters on)
- **MCMICRO** (Upstream preprocessing of multi-channel whole-slide images into single-cell data for the multiplexed-imaging use case)

## Examples

```
conda env create --name spatialcells --file=conda.yaml; python -c "import spatialcells.spatial as sp; adata=...; sp.getCommunities(adata); sp.getBoundary(adata); sp.assignPointsToRegions(adata)"
```

## Evaluation signals

- gc_annotations.csv exists and contains one annotation entry per spot (no missing or duplicated spot-level labels)
- Predicted germinal-center spot labels overlap the manual annotation qualitatively consistent with the agreement reported for the Visium lymph node analysis
- Delineated community boundaries are consistent with the authors' manual annotation boundaries
- Every spot/cell in the AnnData object is assigned to a labeled region by assignPointsToRegions (no unassigned points)
- Downstream measurements (e.g., region composition via getRegionComposition) return sensible cell-type proportions per delineated region

## Limitations

- SpatialCells 'can currently only be installed from source' (via conda env from conda.yaml or pip install -r requirements.txt; pip install .), complicating deployment
- On Visium spot-level data, results depend on the quality of prior Cell2location deconvolution; boundaries are computed on deconvolved cell-type composition rather than direct single-cell measurements
- Analysis requires user-defined parameters and standardized preprocessing choices; including fragmented cell populations without exclusion can artificially distort boundary/distance measurements (as when fragmented SOX10+ areas inflated distances in MEL1)

## Evidence

- [methods] assignPointsToRegions
---------------------
.. autofunction:: spatialcells.spatial.assignPointsToRegions: ".. autofunction:: spatialcells.spatial.assignPointsToRegions"
- [readme] See https://semenovlab.github.io/SpatialCells/ for detailed documentation and tutorials.: [evidence span withheld — anchor status: external_doc]
