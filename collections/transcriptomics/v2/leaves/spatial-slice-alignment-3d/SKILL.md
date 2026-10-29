---
name: spatial-slice-alignment-3d
description: Use when multiple adjacent tissue slices profiled by seq-based ST (e.g.,
  10x Visium) must be combined into a single 3D coordinate system before cell type
  identification or spatially informed modeling.
when_to_use_negative:
- Only a single slice is available and no neighboring-slice information can be borrowed
  for 3D priors
- The analysis target is spot-level deconvolution alone (e.g., RCTD, Cell2location,
  CARD) that does not exploit cross-slice spatial structure
- Slices are from non-adjacent or biologically non-corresponding regions, where pairwise
  alignment assumptions are invalid
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PASTE
  role: Computes the pairwise alignment between the two segmented cortex slices to
    construct the aligned 3D mouse brain cortex ST data
- name: SpatialScope
  role: Consumes the aligned 3D ST data for downstream cell type identification using
    3D spatial priors (Potts-model smoothness, ν = 10)
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: StarDist
  role: Nucleus segmentation of the matched H&E images producing cell counts and cell_locations
    per spot used as input to the aligned 3D pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:cc02952a80299323b9ecd842cba601da34ed70fd43be2d43e28bae3696d08da4
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# spatial-slice-alignment-3d

## Summary

Aligns adjacent spatial transcriptomics slices (e.g., two 10x Visium mouse brain cortex sections) into a common coordinate frame with PASTE to construct aligned 3D ST data, so that downstream methods like SpatialScope can incorporate 3D spatial structure and neighboring-slice information for spatially smooth cell type inference.

## When to use

Use when multiple adjacent tissue slices profiled by seq-based ST (e.g., 10x Visium) must be combined into a single 3D coordinate system before cell type identification or spatially informed modeling.

## When NOT to use

- Only a single slice is available and no neighboring-slice information can be borrowed for 3D priors
- The analysis target is spot-level deconvolution alone (e.g., RCTD, Cell2location, CARD) that does not exploit cross-slice spatial structure
- Slices are from non-adjacent or biologically non-corresponding regions, where pairwise alignment assumptions are invalid

## Inputs

- Two adjacent 10x Visium sagittal mouse brain cortex slices (spot-level ST count matrices)
- Matched H&E images for cortex region segmentation
- Segmented/filtered spot tables (tissue-mapped spots with ≥100 total read counts)

## Outputs

- Pairwise slice alignment mapping spots between adjacent sections
- Aligned 3D mouse brain cortex ST dataset with spatially consistent coordinates across slices

## How to apply

First preprocess each slice: remove spots not mapping to the tissue region and spots with fewer than 100 total read counts, and segment the region of interest (e.g., the cerebral cortex from matched H&E, leaving 812 and 794 cortex spots in the mouse brain example). Then compute a pairwise alignment of the segmented slices with PASTE, producing spot coordinates in a shared frame that together form the aligned 3D mouse brain cortex ST data. This 3D structure supplies spatial priors for downstream SpatialScope cell type identification (MAP/Potts-model inference with smoothness hyperparameter ν = 10), which relies on the assumption that neighboring cells in 3D space tend to share cell type. The rationale is that borrowing information from adjacent slices restores layer-level spatial smoothness that single-slice spot-level deconvolution lacks, correcting misplacements such as L4 and L6 IT cells assigned to wrong cortical layers by StarDist+RCTD.

## Related tools

- **PASTE** (Computes the pairwise alignment between the two segmented cortex slices to construct the aligned 3D mouse brain cortex ST data)
- **SpatialScope** (Consumes the aligned 3D ST data for downstream cell type identification using 3D spatial priors (Potts-model smoothness, ν = 10)) — https://github.com/YangLabHKUST/SpatialScope
- **StarDist** (Nucleus segmentation of the matched H&E images producing cell counts and cell_locations per spot used as input to the aligned 3D pipeline)

## Evaluation signals

- Aligned slices preserve the correct number of retained cortex spots (812 and 794 in the mouse brain cortex example) after removing non-tissue and low-count (<100 UMI) spots
- Downstream SpatialScope labels place L4 and L6 IT cells within their corresponding cortical layers (L2/3, L4, L5, L6), unlike StarDist+RCTD which misidentifies them
- Discrete cell type label map exhibits spatial smoothness consistent with the ν = 10 Potts-model constraint in 3D space
- Output CellTypeLabel_nu10.csv exists and labels reproduce the layered cortical glutamatergic organization

## Limitations

- The Potts-model assumption that neighboring cells belong to the same cell type may not always be valid
- Paired histological images may be unavailable, and alternative image types (e.g., DAPI) or no images complicate cell count estimation needed before alignment
- Platform effects between ST and scRNA-seq reference data must be corrected or downstream cell type identification and gene expression decomposition results are unsatisfactory

## Evidence

- [methods] Adjacent Visium slices used for the 3D alignment workflow: "The two adjacent sagittal slices of mouse brain anterior tissue were from BioIVT Asterand and profiled by 10x Visium"
- [methods] Filtering prerequisite before alignment and 3D inference: "We filtered out genes that have zero counts across all spots and filtered out spots with total UMI counts less than 100"
- [readme] SpatialScope is the consuming downstream framework for the aligned 3D data: "A unified approach for integrating spatial and single-cell transcriptomics data by leveraging deep generative models"
