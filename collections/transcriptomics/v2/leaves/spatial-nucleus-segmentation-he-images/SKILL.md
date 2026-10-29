---
name: spatial-nucleus-segmentation-he-images
description: Use when processing seq-based ST data (e.g., 10x Visium) that has matched
  H&E images, and per-spot cell counts plus per-cell spatial locations (cell_locations)
  are needed to resolve spot-level data (~55 μm spots) to single-cell resolution.
when_to_use_negative:
- Input is Slide-seq V2 data without histology images — SpatialScope replaces nucleus
  segmentation with singlet/doublet classification, assuming at most two cell types
  co-exist within a spot
- Input is image-based ST (e.g., MERFISH, seqFISH) that already resolves single cells
  — segmentation is redundant
- Only a single-channel nuclear image (e.g., DAPI) or no image is available — SpatialScope
  notes these circumstances complicate cell count estimation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: StarDist
  role: Nucleus segmentation engine; SpatialScope default for H&E images, used to
    obtain cell counts and cell_locations per spot
- name: SpatialScope
  role: Unified pipeline that consumes segmentation output (Step 1) for cell type
    identification and gene expression decomposition
  repo: https://github.com/YangLabHKUST/SpatialScope
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:9e7bd34ef0b028489d50ab6cff5b4af08207cc922d2582c475e41b2a6bccb39a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - PASTE
  - RCTD
  - SpatialScope
  - StarDist
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Spatial nucleus segmentation from H&E images

## Summary

Step 1 of the SpatialScope pipeline: run nucleus segmentation (StarDist, the SpatialScope default for H&E) on matched H&E histology images to quantify the number of cells per spatial transcriptomics spot and locate individual cells, providing the single-cell scaffold for downstream cell type identification and gene expression decomposition.

## When to use

Use when processing seq-based ST data (e.g., 10x Visium) that has matched H&E images, and per-spot cell counts plus per-cell spatial locations (cell_locations) are needed to resolve spot-level data (~55 μm spots) to single-cell resolution.

## When NOT to use

- Input is Slide-seq V2 data without histology images — SpatialScope replaces nucleus segmentation with singlet/doublet classification, assuming at most two cell types co-exist within a spot
- Input is image-based ST (e.g., MERFISH, seqFISH) that already resolves single cells — segmentation is redundant
- Only a single-channel nuclear image (e.g., DAPI) or no image is available — SpatialScope notes these circumstances complicate cell count estimation

## Inputs

- Matched H&E histology image paired with a seq-based ST run (e.g., 10x Visium)
- Visium spot-level count matrix (filtered: spots with <100 total UMI counts and spots outside the tissue region removed)
- Optional region-of-interest mask or annotation (e.g., segmented cerebral cortex region)

## Outputs

- Per-spot cell count matrix (number of segmented nuclei per Visium spot)
- cell_locations file giving spatial coordinates of each segmented cell
- Segmented nuclei/region masks from the H&E image for downstream SpatialScope steps

## How to apply

First preprocess the ST tissue: remove spots not mapping to the tissue region and spots with total UMI counts below 100 (the Visium mouse brain cortex analysis left 812 and 794 cortex spots), and optionally segment the region of interest (e.g., cerebral cortex) from the matched H&E image. Then run StarDist nucleus segmentation — SpatialScope's default segmentation engine for H&E images — on the histology image to detect nuclei within each spot, producing a per-spot cell count and a cell_locations file giving each segmented cell's coordinates. These outputs are the building blocks for SpatialScope Steps 2 and 3: cell type identification (MAP/Potts inference with smoothness hyperparameter ν = 10) and gene expression decomposition with the score-based generative model, which together enhance Visium data to single-cell resolution.

## Related tools

- **StarDist** (Nucleus segmentation engine; SpatialScope default for H&E images, used to obtain cell counts and cell_locations per spot)
- **SpatialScope** (Unified pipeline that consumes segmentation output (Step 1) for cell type identification and gene expression decomposition) — https://github.com/YangLabHKUST/SpatialScope

## Evaluation signals

- Segmentation produces a cell count and cell_locations entry for each tissue-mapped spot (e.g., 812 and 794 cortex spots for the two mouse brain slices)
- Cells pre-filtering: spots with <100 total UMI counts and spots outside the tissue region are excluded before segmentation
- Downstream SpatialScope cell type identification reproduces anatomically correct structure (e.g., L4 and L6 IT cells in their corresponding cortical layers), which depends on accurate per-cell locations
- Output h5ad contains decomposed per-cell expression for exactly the segmented cells located in Step 1

## Limitations

- Nucleus segmentation quality bounds the entire pipeline: it 'serves as a building block to quantify the cell count at each spot', so segmentation errors propagate to cell type identification and decomposition
- Histological images may be unavailable or replaced by single-channel nuclear images (e.g., DAPI), complicating cell count estimation
- The Potts-model smoothness assumption (neighboring cells belong to the same cell type) applied after segmentation may not always be biologically valid

## Evidence

- [methods] SpatialScope three-step pipeline includes segmentation: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [methods] Rationale for segmentation step: "Accurate segmentation of nuclei/cells in microscopy images is an important step to locate cells"
- [discussion] Segmentation as foundation for cell counts: "Noted that Step 1, Nucleus segmentation serves as a building block to quantify the cell count at each spot."
- [intro] Spot resolution limitation motivating segmentation: "considering the larger spot size of 55 μm, a Visium spot often contains multiple cells, which limits its usage in resolving detailed tissue structure"
