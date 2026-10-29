---
name: spatial-tumor-microenvironment-fraction-analysis
description: Use when you have region-annotated single-cell data from multiplexed
  imaging (e.g., MCMICRO output) and need per-subregion fractions of marker-defined
  cell populations, such as Ki67+/SOX10+ or MITF+/SOX10+ fractions across tumor core,
  peripheral bands, or invasive-front subregions.
when_to_use_negative:
- Input is raw multi-channel whole-slide imagery not yet converted to single-cell
  data — MCMICRO must run first, as 'The output of MCMICRO is the input of SpatialCells.'
- The analysis goal is neighborhood-graph or single-cell transcriptomics spatial analysis
  rather than region-based composition of marker-gated populations (Squidpy-style
  graph analysis is a different toolkit).
- The region of interest includes fragmented SOX10+ satellite areas whose inclusion
  would artificially inflate distance measurements — exclude them before computing
  region-based fractions.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_2640
tools:
- name: SpatialCells
  role: Computes region composition fractions (spatialcells.msmt.getRegionComposition)
    and the MITF-high proliferative index (spatialcells.msmt.getMPI) over assigned
    spatial regions
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
  source_content_hash: sha256:7b40c16dfacfeadc61401c7d8a90a8d532fa5d393766a31401937c1deb75d5f0
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spatial-tumor-microenvironment-fraction-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes region-resolved cell-composition fractions (e.g., Ki67+/SOX10+ proliferating tumor cells and MITF+/SOX10+ cells) within user-defined spatial subregions of a tumor using SpatialCells, on MCMICRO-derived multiplexed single-cell data. This enables quantitative comparison of marker enrichment across tumor subregions, as demonstrated on the MEL1 melanoma CyCIF sample (1,110,585 cells).

## When to use

Use when you have region-annotated single-cell data from multiplexed imaging (e.g., MCMICRO output) and need per-subregion fractions of marker-defined cell populations, such as Ki67+/SOX10+ or MITF+/SOX10+ fractions across tumor core, peripheral bands, or invasive-front subregions.

## When NOT to use

- Input is raw multi-channel whole-slide imagery not yet converted to single-cell data — MCMICRO must run first, as 'The output of MCMICRO is the input of SpatialCells.'
- The analysis goal is neighborhood-graph or single-cell transcriptomics spatial analysis rather than region-based composition of marker-gated populations (Squidpy-style graph analysis is a different toolkit).
- The region of interest includes fragmented SOX10+ satellite areas whose inclusion would artificially inflate distance measurements — exclude them before computing region-based fractions.

## Inputs

- MCMICRO output single-cell marker intensity table for a multiplexed imaging sample (e.g., MEL1 CyCIF, 1,110,585 cells, SOX10/Ki67/MITF channels among 30 markers)
- Per-cell region annotations for the main tumor area subregions (e.g., from task_001 region assignment or spatialcells.spatial.assignPointsToRegions)
- Gating definitions for marker-positive phenotypes (e.g., SOX10+ tumor cells, Ki67+ proliferating cells) set via spatialcells.prep.setGate/setGates

## Outputs

- CSV table of per-subregion composition fractions (e.g., regional_marker_fractions.csv) with one row per tumor subregion and Ki67+/SOX10+ and MITF+/SOX10+ fraction columns
- MITF-high proliferative index (MPI) values per cell/region from spatialcells.msmt.getMPI
- Barplots of per-subregion marker fractions analogous to Figure 3D/E

## How to apply

Load the MCMICRO-produced single-cell marker intensity table (e.g., SOX10, Ki67, MITF channels from the MEL1 CyCIF sample imaged with 30 antibody markers) together with per-cell region annotations defining main-tumor subregions (tumor core, peripheral/invasive-front bands, immune-rich periphery), having first excluded the fragmented SOX10+ area so distance measurements are not artificially increased. Use spatialcells.spatial.assignPointsToRegions (built on getCellKDE, getBoundary, getCommunities) to assign cells to regions if annotations are not yet present. Then call spatialcells.msmt.getRegionComposition to compute, per subregion, the fraction of cells that are Ki67+/SOX10+ proliferating tumor cells, and spatialcells.msmt.getMPI to derive the MITF-high proliferative index and the fraction of MITF+/SOX10+ tumor cells per subregion. Assemble per-region fractions into a single CSV (e.g., regional_marker_fractions.csv) with one row per tumor subregion and fraction fields for each marker combination.

## Related tools

- **SpatialCells** (Computes region composition fractions (spatialcells.msmt.getRegionComposition) and the MITF-high proliferative index (spatialcells.msmt.getMPI) over assigned spatial regions) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide images into the single-cell data consumed by SpatialCells)

## Examples

```
from spatialcells.msmt import getRegionComposition, getMPI; comp = getRegionComposition(adata, region_col='region', phenotypes=['Ki67+SOX10+', 'MITF+SOX10+']); comp.to_csv('regional_marker_fractions.csv')
```

## Evaluation signals

- Output CSV exists and contains one row per tumor subregion with Ki67+/SOX10+ and MITF+/SOX10+ fraction fields.
- Per-subregion fractions qualitatively match the Figure 3D/E directional trends: highest Ki67+/SOX10+ proliferating fraction at the invasive-front/periphery subregions and a MITF+/SOX10+ gradient across subregions.
- Overall proliferating tumor-cell percentage (MPI = 1) is consistent with the reported 7.4% baseline for MEL1.
- The fragmented SOX10+ area was excluded from the main tumor area before region assignment, so distance-based subregion boundaries are not artificially increased.
- Sanity-check related region metrics if computed alongside (e.g., tumor isolation index in the reference sample is 46.9%, the percentage of tumor cells in the immune-isolated region).

## Limitations

- SpatialCells can currently only be installed from source (pip install -r requirements.txt; pip install . or conda env create from conda.yaml).
- No software version or changelog is stated for SpatialCells, complicating reproducibility pinning.
- Subregion definition choices (distance from centroid vs. angle from a zero-degree reference) determine the gradient patterns observed; fragmented tumor areas must be manually identified and excluded.

## Evidence

- [readme] Installation from source requirement: [evidence span withheld — anchor status: not_found]
