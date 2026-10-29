---
name: multiplexed-imaging-cell-phenotyping
description: Use when you have single-cell data from multiplexed imaging (e.g. MCMICRO
  output from a CyCIF slide) and need to assign phenotypic identities via marker-intensity
  gates — e.g. separating SOX10+ tumor cells from CD3D+ immune cells or quantifying
  Ki67+ proliferation fractions.
when_to_use_negative:
- Input is not multiplexed single-cell data (e.g. bulk RNA-seq or unsegmented whole-slide
  pixels) — SpatialCells requires per-cell marker intensities from a tool like MCMICRO
- The goal is neighborhood-graph spatial network analysis rather than marker-based
  cell phenotyping — Squidpy specializes in neighborhood graph construction and analysis
- Spatial annotation requires extensive manual annotation workflows (e.g. HALO-style
  manual markup) rather than automated region-based analysis
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_2648
tools:
- name: SpatialCells
  role: 'Core package: setGate/setGates for marker-intensity gating, getMPI for proliferation
    index, getRegionComposition for marker-positive fraction quantification'
  repo: https://github.com/SemenovLab/SpatialCells
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide images into
    the single-cell data used as input to SpatialCells
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:5006a5fe8bbe1e542e7b7a2093286378824259b26d969bd805805d8cfb7aa9a7
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
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_2648
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# multiplexed-imaging-cell-phenotyping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Phenotype cells in multiplexed imaging data (e.g. CyCIF with 30 antibody markers) by gating marker intensities to define marker-positive populations such as SOX10+ tumor cells and CD3D+ immune cells, then measuring marker co-expression like Ki67+/SOX10+ proliferation. This converts multi-channel whole-slide images into analyzable single-cell phenotypic labels.

## When to use

Use when you have single-cell data from multiplexed imaging (e.g. MCMICRO output from a CyCIF slide) and need to assign phenotypic identities via marker-intensity gates — e.g. separating SOX10+ tumor cells from CD3D+ immune cells or quantifying Ki67+ proliferation fractions.

## When NOT to use

- Input is not multiplexed single-cell data (e.g. bulk RNA-seq or unsegmented whole-slide pixels) — SpatialCells requires per-cell marker intensities from a tool like MCMICRO
- The goal is neighborhood-graph spatial network analysis rather than marker-based cell phenotyping — Squidpy specializes in neighborhood graph construction and analysis
- Spatial annotation requires extensive manual annotation workflows (e.g. HALO-style manual markup) rather than automated region-based analysis

## Inputs

- Single-cell data from MCMICRO preprocessing of multi-channel whole-slide CyCIF images (per-cell marker intensity matrix)
- Multiplexed imaging dataset with multiple antibody markers (e.g. MEL1 cutaneous melanoma, CyCIF, 30 antibody markers, 1,110,585 cells)

## Outputs

- Per-cell phenotypic labels (e.g. SOX10+ tumor cells, CD3D+ immune cells, Ki67+/SOX10+ proliferating cells)
- Marker-positive fractions and MPI = 1 proliferating tumor cell percentages (e.g. 7.4% for MEL1)

## How to apply

Start from MCMICRO output, which transforms multi-channel whole-slide images into single-cell data and serves as the input to SpatialCells. In the spatialcells.prep module, set gates on marker intensities (setGate/setGates) to dichotomize each marker into positive/negative states, defining cell populations such as SOX10+ tumor cells and CD3D+ immune cells in the CyCIF-stained sample. Once cells are phenotyped, quantify co-expression readouts such as the multivariate proliferation index via spatialcells.msmt.getMPI, where MPI = 1 denotes proliferating (Ki67+) tumor cells — in the MEL1 melanoma sample (1,110,585 cells) this yields 7.4% proliferating tumor cells, with Ki67+/SOX10+ cells concentrated along the periphery of the main tumor area and the invasive front. SpatialCells processes samples containing millions of cells and is installed from source in a conda environment (conda.yaml).

## Related tools

- **SpatialCells** (Core package: setGate/setGates for marker-intensity gating, getMPI for proliferation index, getRegionComposition for marker-positive fraction quantification) — https://github.com/SemenovLab/SpatialCells
- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide images into the single-cell data used as input to SpatialCells)

## Examples

```
conda env create --name spatialcells --file=conda.yaml; python -c "from spatialcells.prep import setGate, setGates; from spatialcells.msmt import getMPI, getRegionComposition"
```

## Evaluation signals

- Recomputed MPI = 1 fraction among tumor cells in MEL1 falls within ±0.5 percentage points of the reported 7.4%
- Gate decisions reproduce the expected marker-positive populations: SOX10+ cells define tumor, CD3D+ cells define immune compartments
- Cell counts match the sample scale (e.g. 1,110,585 cells for MEL1) and phenotyping covers all cells in the ROI
- Reproduction of the reported spatial pattern: a greater proportion of Ki67+/SOX10+ cells along the periphery of the main tumor area and at the advancing edge of the invasive front

## Limitations

- SpatialCells can currently only be installed from source; a conda environment must be created from conda.yaml
- Including fragmented areas of SOX10+ cells together with intervening empty spaces would artificially increase distance-based measurements, so such fragments must be excluded when focusing on the main tumor area
- No software version or changelog is stated for SpatialCells, complicating reproducibility

## Evidence

- [readme] README: install via pip from the repository; documentation and tutorials at project site: [evidence span withheld — anchor status: not_found]
