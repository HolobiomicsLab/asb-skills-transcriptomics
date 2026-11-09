---
name: tma-core-image-tiling-and-subimage-sampling
description: Use when benchmarking cell segmentation pipelines on TMA cores or other
  large tissue images that must first be divided into uniform tiles (e.g., 512x512)
  to match an evaluation scheme.
when_to_use_negative:
- Input images are already small single-cell crops where tiling would fragment cells
  across tile boundaries
- The goal is whole-core segmentation output rather than tiled sub-image-level evaluation
  against manual annotations
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Segmentation pipeline run on each 512x512 sub-image (Python implementation)
    whose outputs are scored per tile against manual annotations
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Mesmer
  role: Deep learning segmentation run on the same 512x512 sub-images using DAPI and
    NaKATPase channels for comparison
- name: Python
  role: Implementation used to run MIRIAM on sub-images and compute DSC/JSC metrics
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Alternate MIRIAM implementation; repository README provides Matlab code and
    tutorial for running cell segmentation on multiplexed imaging data
  repo: https://github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:94d53f985aae0f0bdb8da192945d2953cd23e06251423ad90b0f665a59dc6961
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  repo_url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Matlab
  - Mesmer
  - Python
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# tma-core-image-tiling-and-subimage-sampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tiles manually annotated tissue microarray (TMA) core multiplexed immunofluorescence images into fixed 512x512 sub-images so that segmentation methods (MIRIAM, Mesmer, Voronoi) can be evaluated per sub-image and per cell against ground-truth cell borders.

## When to use

Use when benchmarking cell segmentation pipelines on TMA cores or other large tissue images that must first be divided into uniform tiles (e.g., 512x512) to match an evaluation scheme.

## When NOT to use

- Input images are already small single-cell crops where tiling would fragment cells across tile boundaries
- The goal is whole-core segmentation output rather than tiled sub-image-level evaluation against manual annotations

## Inputs

- Manually annotated TMA core multiplexed immunofluorescence images
- Ground-truth cell boundary annotations for each TMA core
- Tile size specification (512x512 sub-images)

## Outputs

- 512x512 sub-image stack per TMA core (240 sub-images across three cores in the reference benchmark)
- Per-sub-image and per-cell DSC/JSC values for each segmentation method (e.g., segmentation_evaluation_metrics.csv)

## How to apply

Load each manually annotated TMA core multiplexed immunofluorescence image together with its ground-truth cell boundary annotations, then tile the image into non-overlapping 512x512 sub-images to match the evaluation scheme — in the MIRIAM benchmark this produced 240 sub-images across the three manually annotated colon adenoma TMA cores. Run each segmentation method (MIRIAM Python implementation, Mesmer using the DAPI and NaKATPase channels, nuclei-based Voronoi tessellation) on the same sub-images so comparisons are made over identical tiles. Compute Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC) per 512x512 sub-image and per individual cell, then aggregate mean ± standard deviation. The fixed tile size keeps the evaluation scheme consistent across methods and enables image-level and cell-level agreement statistics against manual annotations.

## Related tools

- **MIRIAM** (Segmentation pipeline run on each 512x512 sub-image (Python implementation) whose outputs are scored per tile against manual annotations) — https://github.com/Coffey-Lab/MIRIAM
- **Mesmer** (Deep learning segmentation run on the same 512x512 sub-images using DAPI and NaKATPase channels for comparison)
- **Python** (Implementation used to run MIRIAM on sub-images and compute DSC/JSC metrics) — https://github.com/Coffey-Lab/MIRIAM
- **Matlab** (Alternate MIRIAM implementation; repository README provides Matlab code and tutorial for running cell segmentation on multiplexed imaging data) — https://github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Total sub-image count matches the evaluation scheme: 240 sub-images across the three manually annotated TMA cores
- Per-sub-image DSC/JSC computed identically for MIRIAM, Mesmer, and Voronoi so method comparisons use the same tiles
- Aggregated image-level values reproduce reference benchmarks (MIRIAM 0.65±0.08 DSC / 0.48±0.08 JSC; Mesmer 0.59±0.10 / 0.42±0.08; Voronoi 0.32±0.11 / 0.20±0.07)
- Aggregated cell-level values reproduce reference benchmarks (MIRIAM 0.67±0.14 DSC / 0.52±0.15 JSC; Mesmer 0.61±0.19 / 0.46±0.18; Voronoi 0.47±0.21 / 0.33±0.18)
- Metrics written with per-sub-image, per-cell, and aggregated levels (segmentation_evaluation_metrics.csv) retain the mean ± standard deviation structure

## Limitations

- Tiling a TMA core into 512x512 sub-images assumes the tile size matches the evaluation scheme; other tile sizes are not validated here
- MIRIAM and Mesmer results are method-dependent: Mesmer was trained on normal mouse small intestine plus normal human colon and colorectal cancer imaging data, which may not generalize to all tissue types
- Voronoi undersegments cells towards the lumen because it does not take into account cell membranes, inflating apparent differences on tiled evaluation
- The MIRIAM repository example data excludes image stacks (too large); users must regenerate or download them separately

## Evidence
