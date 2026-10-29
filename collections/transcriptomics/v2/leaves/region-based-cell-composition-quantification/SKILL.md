---
name: region-based-cell-composition-quantification
description: Use when a MCMICRO-derived single-cell table from multiplexed imaging
  (e.
when_to_use_negative:
- The input is already an aggregated composition table or lacks per-cell spatial coordinates,
  so no region assignment can be performed
- The sample has fragmented SOX10+ satellite areas that cannot be excluded — including
  them artificially increases distance-based region measurements
- The analysis goal is neighborhood-graph or interaction analysis rather than per-region
  phenotype fractions (SpatialCells positions itself for region-based composition/clustering
  within whole-slide images)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SpatialCells
  role: Provides setGate/setGates, assignPointsToRegions, getRegionComposition, and
    getMPI to compute per-region Ki67+/SOX10+ and MITF+/SOX10+ fractions from gated
    single-cell data
  repo: https://semenovlab.github.io/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide CyCIF images
    into the single-cell data that serve as input to SpatialCells
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:519f2cd2fbc9c9505694956ac462099ad429d3281a1bc0d6a09774b33bf0076f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/SemenovLab/SpatialCells
  tools:
  - MCMICRO
  - SpatialCells
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# region-based-cell-composition-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the fraction of marker-defined cell phenotypes (e.g., Ki67+/SOX10+ proliferating tumor cells, MITF+/SOX10+ cells) within user-defined spatial subregions of multiplexed imaging single-cell data, using SpatialCells' msmt module. It reveals spatial gradients such as proliferation enrichment at the invasive front of the MEL1 melanoma sample.

## When to use

Use when a MCMICRO-derived single-cell table from multiplexed imaging (e.g., CyCIF with SOX10, Ki67, MITF channels) must be split into spatial subregions (distance bands from the tumor centroid or angular sectors from a 0° reference) and per-region marker-positive fractions are needed to test spatial gradients.

## When NOT to use

- The input is already an aggregated composition table or lacks per-cell spatial coordinates, so no region assignment can be performed
- The sample has fragmented SOX10+ satellite areas that cannot be excluded — including them artificially increases distance-based region measurements
- The analysis goal is neighborhood-graph or interaction analysis rather than per-region phenotype fractions (SpatialCells positions itself for region-based composition/clustering within whole-slide images)

## Inputs

- MCMICRO output single-cell feature table (per-cell x/y coordinates and marker intensities for SOX10, Ki67, MITF from a 30-marker CyCIF slide)
- Per-cell region annotations mapping cells to main-tumor subregions (tumor core, peripheral/invasive-front bands, immune-rich periphery)
- Gate definitions (thresholds) for SOX10, Ki67, and MITF positivity

## Outputs

- regional_marker_fractions.csv: one row per tumor subregion with Ki67+/SOX10+ and MITF+/SOX10+ fraction fields
- Barplots of per-region Ki67+/SOX10+ and MITF+/SOX10+ fractions (analogous to Figure 3D and 3E)

## How to apply

Start from the MCMICRO single-cell feature table (MEL1: 1,110,585 cells, 30 CyCIF antibody markers) plus per-cell region annotations; gate marker intensities with spatialcells.prep.setGate/setGates, and first exclude fragmented SOX10+ areas so distance measurements are not artificially increased. Define subregions with getCommunities/getBoundary and spatialcells.spatial.assignPointsToRegions, using bands from the main tumor centroid or sectors relative to the zero-degree reference, then restrict to SOX10+ tumor cells. Call spatialcells.msmt.getRegionComposition on each subregion to obtain the Ki67+/SOX10+ fraction, and spatialcells.msmt.getMPI to derive the MITF-high proliferative index and the MITF+/SOX10+ fraction per subregion. Assemble per-region fractions into a single CSV (regional_marker_fractions.csv) and benchmark the recomputed values against the reported overall proliferating tumor cell percentage (MPI = 1) of 7.4%.

## Related tools

- **SpatialCells** (Provides setGate/setGates, assignPointsToRegions, getRegionComposition, and getMPI to compute per-region Ki67+/SOX10+ and MITF+/SOX10+ fractions from gated single-cell data) — https://semenovlab.github.io/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide CyCIF images into the single-cell data that serve as input to SpatialCells)

## Examples

```
conda env create --name spatialcells --file=conda.yaml
```

## Evaluation signals

- Output CSV exists and contains exactly one row per tumor subregion with numeric Ki67+/SOX10+ and MITF+/SOX10+ fraction fields in [0, 1]
- The overall proliferating tumor-cell percentage (MPI = 1) recomputed across the whole main tumor area is consistent with the reported 7.4% baseline
- Directional trend reproduced: greatest Ki67+/SOX10+ proportion along the main tumor periphery and at the advancing edge of the invasive front, as in Figure 3D
- MITF+/SOX10+ fractions show a monotonic spatial gradient across subregions ordered by distance from the centroid or by angle from the 0° reference, as in Figure 3E
- Region assignment excludes the fragmented SOX10+ area, so distance-to-boundary measurements match the curated main tumor area rather than inflated values

## Limitations

- Fragmented SOX10+ cell areas must be excluded from the main tumor area before distance measurement; otherwise measurements are artificially increased
- Results depend on user-defined gates and region boundaries (user-defined parameters), so composition fractions are not threshold-independent
- SpatialCells can currently only be installed from source, and no software version is stated in the article or changelog

## Evidence
