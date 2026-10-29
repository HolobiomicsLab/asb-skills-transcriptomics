---
name: anndata-single-cell-marker-gating
description: Use when you have per-cell marker intensity tables from multiplexed imaging
  (e.g., MCMICRO output for the MEL1 CyCIF sample) and need to define marker-positive
  populations (SOX10+ tumor cells, Ki67+ proliferating cells, MITF-high state) before
  computing region-level composition metrics.
when_to_use_negative:
- Input is image-level or whole-slide data that has not yet been converted to per-cell
  measurements — run MCMICRO first, since SpatialCells requires single-cell data as
  input
- The goal is neighborhood-graph or cell-interaction analysis rather than marker-based
  cell phenotyping within regions — Squidpy-style tools or spatial.getMinCellTypesDistance
  are better suited
- Manual annotation of regions in commercial software (e.g., HALO) is acceptable and
  desired; SpatialCells targets automated, scalable, user-parameterized workflows
  over millions of cells
tools:
- name: SpatialCells
  role: Provides prep.setGate/setGates for marker gating, msmt.getMPI for the MITF-high
    proliferative index, and msmt.getRegionComposition for per-region gated fraction
    computation
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream tool that transforms multi-channel whole-slide CyCIF images into
    the single-cell data that is gated by this skill
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:0bbab7c043780c0a469f770e77178c50d4f5f5682f138729af78dc23f7e774d1
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# anndata-single-cell-marker-gating

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Gate single-cell marker intensities (e.g., SOX10, Ki67, MITF from CyCIF multiplexed imaging) into binary positive/negative or graded MPI values using SpatialCells' prep.setGate/setGates, then use the gated labels to quantify regional marker composition such as Ki67+/SOX10+ proliferating tumor-cell fractions and the MITF-high proliferative index (MPI).

## When to use

Use when you have per-cell marker intensity tables from multiplexed imaging (e.g., MCMICRO output for the MEL1 CyCIF sample) and need to define marker-positive populations (SOX10+ tumor cells, Ki67+ proliferating cells, MITF-high state) before computing region-level composition metrics.

## When NOT to use

- Input is image-level or whole-slide data that has not yet been converted to per-cell measurements — run MCMICRO first, since SpatialCells requires single-cell data as input
- The goal is neighborhood-graph or cell-interaction analysis rather than marker-based cell phenotyping within regions — Squidpy-style tools or spatial.getMinCellTypesDistance are better suited
- Manual annotation of regions in commercial software (e.g., HALO) is acceptable and desired; SpatialCells targets automated, scalable, user-parameterized workflows over millions of cells

## Inputs

- Single-cell marker intensity table (per-cell values for SOX10, Ki67, MITF channels) from MCMICRO preprocessing of CyCIF multiplexed imaging data
- Per-cell region annotations defining tumor subregions (e.g., from spatial.assignPointsToRegions over the MEL1 main tumor area and invasive front)

## Outputs

- Per-cell binary marker gates (SOX10+, Ki67+, MITF-high labels) and MPI scores
- Per-subregion composition table of gated marker fractions (e.g., regional_marker_fractions.csv with Ki67+/SOX10+ and MITF+/SOX10+ fractions per tumor subregion)

## How to apply

Start from the single-cell data produced by MCMICRO, which 'transforms multi-channel whole-slide images into single-cell data' and serves as the input to SpatialCells. Apply spatialcells.prep.setGate/setGates to the marker intensity channels (SOX10, Ki67, MITF) to assign per-cell positivity, restricting downstream analysis to SOX10+ tumor cells within the main tumor area (the fragmented SOX10+ area is excluded to avoid artificially inflated distances). For proliferation, derive the MITF-high proliferative index with spatialcells.msmt.getMPI, where MPI = 1 denotes proliferating tumor cells — the paper reports an overall 7.4% MPI=1 fraction in MEL1 as the baseline. Then pass the gated labels to spatialcells.msmt.getRegionComposition to compute per-subregion fractions such as Ki67+/SOX10+ and MITF+/SOX10+ (e.g., across tumor core, peripheral/invasive-front bands, immune-rich periphery, or subregions defined by distance from the centroid or angle from a zero-degree reference). Gate thresholds and region definitions are user-defined parameters, so record them explicitly to keep the analysis standardized and reproducible.

## Related tools

- **SpatialCells** (Provides prep.setGate/setGates for marker gating, msmt.getMPI for the MITF-high proliferative index, and msmt.getRegionComposition for per-region gated fraction computation) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream tool that transforms multi-channel whole-slide CyCIF images into the single-cell data that is gated by this skill)

## Examples

```
from spatialcells.prep import setGates; from spatialcells.msmt import getMPI, getRegionComposition; setGates(adata, ...); getMPI(...); getRegionComposition(...)
```

## Evaluation signals

- The output composition table contains one row per tumor subregion with Ki67+/SOX10+ and MITF+/SOX10+ fraction fields
- The recomputed overall percentage of proliferating tumor cells (MPI = 1) matches the reported MEL1 baseline of 7.4%
- Per-subregion fractions qualitatively reproduce the Figure 3D/E directional trends: higher Ki67+/SOX10+ fraction at the tumor periphery and advancing edge of the invasive front, and a MITF+/SOX10+ gradient across subregions
- Gate assignments are reproducible: user-defined gate parameters are recorded and re-running setGates on the same intensity table yields identical per-cell labels
- SOX10+ cells in the fragmented area are excluded from the main-tumor-area gating/quantification, as including them would artificially increase distance measurements

## Limitations

- Gate thresholds are user-defined parameters, so results depend on analyst choices and should be standardized across samples
- SpatialCells can currently only be installed from source (pip install -r requirements.txt; pip install .), and no software version or changelog is documented
- Including fragmented SOX10+ areas with intervening empty spaces artificially inflates spatial measurements, so region definitions must exclude such fragments before gating-based composition analysis

## Evidence

- [methods] setGate/setGates are the SpatialCells prep functions used to gate marker intensities into per-cell positive labels: "setGate
-------
.. autofunction:: spatialcells.prep.setGate"
- [results] Overall percentage of proliferating tumor cells (MPI = 1) in MEL1 serves as the validation baseline for gated Ki67/MITF composition: "The overall percentage of proliferating tumor cells (MPI = 1) in the MEL1 sample is 7.4%."
- [results] Expected directional result when gating Ki67/SOX10 across tumor subregions: "a greater proportion of Ki67+/SOX10+ cells are along the periphery of the main tumor area and, most prominently, at the advancing edge of the invasive front."
- [results] MCMICRO output is the single-cell input that marker gating operates on: "preprocessed using MCMICRO [7], which transforms multi-channel whole-slide images into single-cell data."
- [readme] SpatialCells is designed for region-based analysis of multiplexed single-cell data, per the repository README: "This package aims to develop computational methods for region-based analyses of multiplexed single-cell data."
