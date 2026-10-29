---
name: multiplexed-imaging-single-cell-data-loading
description: Use when the input is multi-channel whole-slide imaging data already
  converted to single-cell form (per-cell x/y coordinates plus marker intensities,
  e.g. MCMICRO output from CyCIF) and it must be loaded into an AnnData object for
  SpatialCells workflows.
when_to_use_negative:
- Input is still raw multi-channel whole-slide imagery that has not been segmented
  by MCMICRO — segmentation/image-to-single-cell conversion happens upstream, not
  in this loading step
- Input is already a feature table without spatial coordinates (e.g., bulk expression
  or non-spatial cytometry) — SpatialCells region-based analysis requires per-cell
  x/y positions
tools:
- name: MCMICRO
  role: Upstream preprocessing that transforms multi-channel whole-slide images into
    the single-cell data consumed by this loading step
- name: SpatialCells
  role: Target package into which the MCMICRO single-cell data is loaded; installed
    from its GitHub repository / conda.yaml
  repo: https://github.com/SemenovLab/SpatialCells
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae189
    title: 'SpatialCells: automated profiling of tumor microenvironments with spatially
      resolved multiplexed single-cell data'
  source_content_hash: sha256:29e4ab523dd57112325bedc8de2471362fffaee3f4841dc24d5fc9c1641f0dc7
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multiplexed-imaging-single-cell-data-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load multiplexed imaging-derived single-cell data (e.g., MEL1 CyCIF sample with 30 antibody markers and 1,110,585 cells preprocessed by MCMICRO) into a per-cell feature table/AnnData object that SpatialCells can consume for region-based tumor microenvironment analysis.

## When to use

Use when the input is multi-channel whole-slide imaging data already converted to single-cell form (per-cell x/y coordinates plus marker intensities, e.g. MCMICRO output from CyCIF) and it must be loaded into an AnnData object for SpatialCells workflows.

## When NOT to use

- Input is still raw multi-channel whole-slide imagery that has not been segmented by MCMICRO — segmentation/image-to-single-cell conversion happens upstream, not in this loading step
- Input is already a feature table without spatial coordinates (e.g., bulk expression or non-spatial cytometry) — SpatialCells region-based analysis requires per-cell x/y positions

## Inputs

- MCMICRO output: single-cell data derived from multi-channel whole-slide images (per-cell x/y coordinates plus marker intensity columns)
- CyCIF multiplexed imaging sample metadata (e.g., MEL1 cutaneous melanoma, 30 antibody markers, 1,110,585 cells)

## Outputs

- AnnData object with one observation per cell, per-cell spatial coordinates, and a cell-by-marker intensity matrix suitable as input to spatialcells.preprocessing.setGates
- Standardized single-cell feature table carrying the 30-marker intensities from the MCMICRO segmentation

## How to apply

Start from the MCMICRO output, which transforms multi-channel whole-slide images into single-cell data and is the designated input to SpatialCells; for the reference MEL1 melanoma case this is CyCIF imaging with 30 antibody markers yielding 1,110,585 cells, each with x/y coordinates and 30-marker intensities. Install SpatialCells from source — it currently has no PyPI/wheel release — by creating a conda environment from the repository's conda.yaml (`conda env create --name spatialcells --file=conda.yaml`) or via `pip install -r requirements.txt && pip install .`. Construct an AnnData object whose observations are cells (x/y spatial coordinates in .obs) and whose variables are the antibody markers (intensity matrix in .X), matching the layout the SpatialCells tutorials use. Downstream steps (setGates thresholding of marker intensities, getBoundary/assignPointsToRegions) assume this loaded, cell-by-marker structure, so validate cell counts and marker-column identity against the MCMICRO segmentation output before proceeding.

## Related tools

- **MCMICRO** (Upstream preprocessing that transforms multi-channel whole-slide images into the single-cell data consumed by this loading step)
- **SpatialCells** (Target package into which the MCMICRO single-cell data is loaded; installed from its GitHub repository / conda.yaml) — https://github.com/SemenovLab/SpatialCells

## Examples

```
conda env create --name spatialcells --file=conda.yaml && pip install -r requirements.txt && pip install .
```

## Evaluation signals

- Cell count in the loaded AnnData matches the MCMICRO segmentation output (e.g., 1,110,585 cells for the MEL1 sample)
- Intensity matrix has exactly 30 marker columns corresponding to the 30 CyCIF antibody markers
- Every cell observation carries finite x/y coordinates so downstream spatial functions (getCommunities, getBoundary) can run
- Conda environment creation from conda.yaml and `pip install .` complete without error, confirming the source-only installation path worked

## Limitations

- SpatialCells can currently only be installed from source; no packaged release or software version is stated in the article or README
- Loading depends on the quality of the upstream MCMICRO segmentation — the output of MCMICRO is the input of SpatialCells, so segmentation errors propagate directly
- No changelog was found for the repository, so version-dependent behavior of the loaders cannot be verified

## Evidence

- [readme] README pip install commands: [evidence span withheld — anchor status: not_found]
