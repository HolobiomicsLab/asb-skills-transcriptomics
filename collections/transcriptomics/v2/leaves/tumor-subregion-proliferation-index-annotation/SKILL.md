---
name: tumor-subregion-proliferation-index-annotation
description: Use when a multiplexed imaging single-cell table (e.g. MCMICRO output
  from the MEL1 melanoma sample, 1,110,585 cells) has SOX10/Ki67/MITF marker intensities
  and region annotations, and the goal is per-subregion Ki67+/SOX10+ and MITF+/SOX10+
  fractions as in Figure 3D/E.
when_to_use_negative:
- The input is not single-cell data — SpatialCells requires MCMICRO-transformed single-cell
  data, so raw whole-slide multi-channel images or bulk region-level readouts are
  the wrong starting point.
- The analysis goal is neighborhood-graph construction or cell–cell interaction analysis
  rather than region-based composition — that is the scope of tools like Squidpy,
  not this region-composition skill.
- 'Cells have not yet been phenotyped: without setGate/setGates on SOX10, Ki67, and
  MITF there are no Ki67+/SOX10+ or MITF+/SOX10+ populations to compute fractions
  over.'
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SpatialCells
  role: Provides getMPI and getRegionComposition to compute the MITF-high proliferative
    index and per-subregion Ki67+/SOX10+ and MITF+/SOX10+ fractions, plus spatial
    region assignment of cells
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms the multi-channel whole-slide CyCIF
    image into the single-cell data consumed by SpatialCells
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:b01cd895301f5b11c313c29cd119c5ac84b7f1e5863f04073d78d0924d578b66
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# tumor-subregion-proliferation-index-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how the fraction of Ki67+/SOX10+ proliferating tumor cells and MITF+/SOX10+ tumor cells varies across subregions of a tumor (defined by distance from the centroid or angle from a zero-degree reference) using SpatialCells' region-based measurements on MCMICRO-derived single-cell data.

## When to use

Use when a multiplexed imaging single-cell table (e.g. MCMICRO output from the MEL1 melanoma sample, 1,110,585 cells) has SOX10/Ki67/MITF marker intensities and region annotations, and the goal is per-subregion Ki67+/SOX10+ and MITF+/SOX10+ fractions as in Figure 3D/E.

## When NOT to use

- The input is not single-cell data — SpatialCells requires MCMICRO-transformed single-cell data, so raw whole-slide multi-channel images or bulk region-level readouts are the wrong starting point.
- The analysis goal is neighborhood-graph construction or cell–cell interaction analysis rather than region-based composition — that is the scope of tools like Squidpy, not this region-composition skill.
- Cells have not yet been phenotyped: without setGate/setGates on SOX10, Ki67, and MITF there are no Ki67+/SOX10+ or MITF+/SOX10+ populations to compute fractions over.

## Inputs

- Per-cell region annotations for the tumor sample (e.g. MEL1 subregions defined by distance-from-centroid or angle bands, produced by prior region-assignment steps)
- Single-cell marker intensity table from multiplexed imaging with SOX10, Ki67, and MITF channels (MCMICRO output; MEL1 contains 1,110,585 cells)
- Gates for SOX10+, Ki67+, and MITF+ phenotypes (set via spatialcells.prep.setGate/setGates)

## Outputs

- regional_marker_fractions.csv with one row per tumor subregion and Ki67+/SOX10+ and MITF+/SOX10+ fraction fields
- Per-subregion MITF-high proliferative index (MPI) values derived via spatialcells.msmt.getMPI

## How to apply

Load the per-cell region annotations for the sample of interest together with the single-cell marker intensity table (SOX10, Ki67, MITF channels), and restrict to SOX10+ tumor cells within the main tumor area, excluding any fragmented areas of SOX10+ cells so that distance measurements are not artificially increased. Define tumor subregions (e.g. tumor core, peripheral/invasive-front bands, immune-rich periphery) by distance from the tumor centroid or by angle from the zero-degree reference, and assign cells using SpatialCells' spatial region assignment (assignPointsToRegions). Compute the MITF-high proliferative index with spatialcells.msmt.getMPI, where MPI = 1 marks proliferating tumor cells, and use spatialcells.msmt.getRegionComposition to obtain, per subregion, the Ki67+/SOX10+ and MITF+/SOX10+ fractions. Assemble the per-region fractions into a single CSV (regional_marker_fractions.csv) and verify the reported directional trends rather than just numeric output.

## Related tools

- **SpatialCells** (Provides getMPI and getRegionComposition to compute the MITF-high proliferative index and per-subregion Ki67+/SOX10+ and MITF+/SOX10+ fractions, plus spatial region assignment of cells) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms the multi-channel whole-slide CyCIF image into the single-cell data consumed by SpatialCells)

## Examples

```
from spatialcells.msmt import getMPI, getRegionComposition  # per-subregion Ki67+/SOX10+ and MITF+/SOX10+ fractions for MEL1 tumor subregions
```

## Evaluation signals

- regional_marker_fractions.csv exists and contains one row per tumor subregion with both Ki67+/SOX10+ and MITF+/SOX10+ fraction fields.
- The overall percentage of proliferating tumor cells (MPI = 1) is consistent with the reported 7.4% baseline for MEL1.
- Directional check: the highest Ki67+/SOX10+ fraction occurs in periphery/advancing-edge invasive-front subregions, matching Figure 3D.
- The MITF+/SOX10+ fractions form a spatial gradient across subregions (by distance-from-centroid or angle), matching Figure 3E.
- Sanity check that fragmented SOX10+ areas were excluded from the main tumor area so that region distances were not artificially increased.

## Limitations

- If fragmented areas of SOX10+ cells are included in the main tumor area, distance measurements are artificially increased; these fragments must be excluded before defining subregions.
- SpatialCells can currently only be installed from source (pip install -r requirements.txt; pip install .), and no software version is stated for the package.
- The directional trends (peripheral enrichment of Ki67+/SOX10+, MITF+/SOX10+ gradient) are demonstrated on a single MEL1 melanoma sample and validated qualitatively against Figure 3D/E, so they are sample- and threshold-dependent rather than a general quantitative benchmark.

## Evidence
