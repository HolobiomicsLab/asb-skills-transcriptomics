---
name: multiplexed-imaging-marker-fraction-analysis
description: Use when you have MCMICRO-derived single-cell data from a multiplexed
  imaging sample (e.g. CyCIF MEL1, 1,110,585 cells) with per-cell region annotations
  and channel intensities, and need marker co-expression fractions per spatial subregion.
when_to_use_negative:
- Input is whole-slide image data not yet converted to single-cell measurements —
  run MCMICRO first, since SpatialCells takes the MCMICRO output as input
- The analysis goal is neighborhood-graph or cell-interaction analysis rather than
  per-region marker composition (Squidpy specializes in neighborhood graph construction)
- Region annotations do not yet exist and subregions have not been defined (region
  assignment must precede composition measurement)
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: SpatialCells
  role: Provides spatialcells.msmt.getRegionComposition and spatialcells.msmt.getMPI
    to compute per-region marker co-expression fractions and the MITF-high proliferative
    index
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide images into
    the single-cell data consumed by SpatialCells
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:c9966e689c958f13df748cae4835d56f1c481159f38fd6fb61e41ce9be85368d
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
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multiplexed-imaging-marker-fraction-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-region marker co-expression fractions (e.g. Ki67+/SOX10+ proliferating tumor cells, MITF+SOX10+ via the MITF-high proliferative index) across spatially defined subregions of multiplexed imaging data using SpatialCells' msmt functions. It reproduces the paper's MEL1 melanoma analysis showing a spatial gradient of MITF+SOX10+ cells and peripheral enrichment of Ki67+/SOX10+ cells.

## When to use

Use when you have MCMICRO-derived single-cell data from a multiplexed imaging sample (e.g. CyCIF MEL1, 1,110,585 cells) with per-cell region annotations and channel intensities, and need marker co-expression fractions per spatial subregion.

## When NOT to use

- Input is whole-slide image data not yet converted to single-cell measurements — run MCMICRO first, since SpatialCells takes the MCMICRO output as input
- The analysis goal is neighborhood-graph or cell-interaction analysis rather than per-region marker composition (Squidpy specializes in neighborhood graph construction)
- Region annotations do not yet exist and subregions have not been defined (region assignment must precede composition measurement)

## Inputs

- MCMICRO-output single-cell table from a multiplexed (CyCIF) image with SOX10, Ki67, MITF channel intensities (MEL1: 1,110,585 cells)
- Per-cell region annotations defining main tumor area subregions (centroid/distance bands, angular sectors, invasive front, immune-rich periphery) from prior region-assignment steps
- Gate definitions for SOX10+ tumor cells and MITF-high proliferating cells (setGates)

## Outputs

- CSV table (regional_marker_fractions.csv) with one row per tumor subregion and columns for Ki67+/SOX10+ fraction and MITF+/SOX10+ fraction
- Per-subregion MPI values identifying MITF-high proliferating tumor cells (MPI = 1)
- Barplots of per-subregion marker fractions comparable to Figure 3D/E

## How to apply

Load the per-cell region annotations produced by spatial region assignment (assignPointsToRegions) together with the single-cell marker intensity table (SOX10, Ki67, MITF channels). Restrict to SOX10+ tumor cells within the main tumor area subregions, having first excluded the fragmented area with SOX10+ cells so distance measurements are not artificially increased. Use spatialcells.msmt.getRegionComposition to compute, per subregion (tumor core, peripheral/invasive front bands, immune-rich periphery), the fraction of Ki67+/SOX10+ cells, and spatialcells.msmt.getMPI to derive the MITF-high proliferative index (MPI = 1 for MITF-high proliferating cells) and the MITF+/SOX10+ fraction per subregion. Assemble the per-region fractions into a single CSV (e.g. regional_marker_fractions.csv) and check that the directional trends match: greatest proportion of Ki67+/SOX10+ cells along the periphery of the main tumor area and at the advancing edge of the invasive front, with an overall proliferating tumor-cell percentage (MPI = 1) near the reported 7.4% baseline.

## Related tools

- **SpatialCells** (Provides spatialcells.msmt.getRegionComposition and spatialcells.msmt.getMPI to compute per-region marker co-expression fractions and the MITF-high proliferative index) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide images into the single-cell data consumed by SpatialCells)

## Examples

```
from spatialcells.msmt import getRegionComposition, getMPI; comp = getRegionComposition(adata, regionCol='regions', markers=['SOX10', 'Ki67']); mpi = getMPI(adata, marker='MITF', proliferation='Ki67')
```

## Evaluation signals

- Output CSV exists with one row per tumor subregion and explicit Ki67+/SOX10+ and MITF+/SOX10+ fraction fields
- Overall proliferating tumor-cell percentage (MPI = 1) is consistent with the reported 7.4% baseline in MEL1
- Recomputed fractions qualitatively reproduce Figure 3D/E directional trends: highest proliferating fraction at the invasive-front/periphery subregions
- Ki67+/SOX10+ enrichment appears along the periphery of the main tumor area and, most prominently, at the advancing edge of the invasive front
- Fragmented SOX10+ areas were excluded before distance-based subregion definition so measurements are not artificially increased

## Limitations

- SpatialCells can currently only be installed from source (download code and pip install); no software version or changelog is stated
- The fragmented area with SOX10+ cells must be excluded from the main tumor area; including it would artificially increase distance measurements
- Results are sample-specific (demonstrated on MEL1 cutaneous melanoma); directional gradients require meaningful prior region definitions to be reproducible

## Evidence
