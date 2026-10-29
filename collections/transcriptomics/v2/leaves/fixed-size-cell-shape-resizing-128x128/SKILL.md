---
name: fixed-size-cell-shape-resizing-128x128
description: Use when segmented single-cell masks (e.g. from MIRIAM's membrane-extended
  segmentation) must be converted into uniform binary matrices to train or apply a
  deep learning-based cell shape descriptor.
when_to_use_negative:
- Input is already a feature/latent-vector table or marker intensity table — resizing
  operates on raw cell masks, not on extracted features
- Cells of interest are not segmented (no membrane-resolved masks); MIRIAM's resizing
  step assumes the membrane-extension segmentation has already produced single-cell
  masks
- The analysis goal is marker intensity quantification over nucleus/membrane/cytoplasm
  compartments rather than shape characterization
tools:
- name: MIRIAM
  role: Pipeline whose segmentation stage produces the cell masks that are binarized,
    rotated, and resized to 128x128 for the shape autoencoder
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Reference implementation of the 128x128 resizing and the downstream autoencoder
    training (Deep Learning Toolbox)
- name: Python
  role: Alternative Keras-based implementation of the same resizing and autoencoder
    workflow; results largely similar to Matlab
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:27bc2d0a1eeaad6be2ffb35be88d6af861b00c1b97c205f4d973bb2fe6b395eb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Matlab
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# Fixed-size cell shape resizing (128x128) for autoencoder input

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares segmented cell masks from the MIRIAM pipeline as fixed-size 128x128 binary matrices so they can serve as standardized inputs to the autoencoder neural network used for the cell shape descriptor.

## When to use

Use when segmented single-cell masks (e.g. from MIRIAM's membrane-extended segmentation) must be converted into uniform binary matrices to train or apply a deep learning-based cell shape descriptor.

## When NOT to use

- Input is already a feature/latent-vector table or marker intensity table — resizing operates on raw cell masks, not on extracted features
- Cells of interest are not segmented (no membrane-resolved masks); MIRIAM's resizing step assumes the membrane-extension segmentation has already produced single-cell masks
- The analysis goal is marker intensity quantification over nucleus/membrane/cytoplasm compartments rather than shape characterization

## Inputs

- Segmented cell masks from the MIRIAM segmentation stage (multi-dimensional tissue images, e.g. colorectal adenoma multiplexed immunofluorescence or MIBI breast carcinoma data)
- Per-cell binary mask matrices

## Outputs

- 128x128 binary cell shape matrices (one per segmented cell) ready for autoencoder encoding
- Latent shape vectors for downstream analysis (e.g. t-SNE, saved to latent_vectors.csv)

## How to apply

Take the segmented cell masks produced by the upstream MIRIAM segmentation stage (random forest pixel classification plus membrane extension) and binarize each cell mask. Rotate each mask so its principal (major) axis is aligned horizontally, then resize the aligned binary mask to a fixed 128x128 matrix that serves as the autoencoder input. Feed these matrices into the MIRIAM cell shape autoencoder, which is trained on a random subset (20% default) of all segmented cells with 256-unit hidden layers, implemented in Matlab (Deep Learning Toolbox) or Python (Keras). The rationale is that a fixed, orientation-normalized grid removes shape-invariant size and rotation effects so the encoder captures intrinsic cell morphology rather than position or scale.

## Related tools

- **MIRIAM** (Pipeline whose segmentation stage produces the cell masks that are binarized, rotated, and resized to 128x128 for the shape autoencoder) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Reference implementation of the 128x128 resizing and the downstream autoencoder training (Deep Learning Toolbox))
- **Python** (Alternative Keras-based implementation of the same resizing and autoencoder workflow; results largely similar to Matlab)

## Evaluation signals

- Every aligned, resized mask is exactly a 128x128 binary matrix (fixed dimensionality check)
- Each resized mask's major axis is horizontal after principal-axis rotation (orientation invariant)
- The number of resized matrices matches the number of segmented cells from the MIRIAM segmentation stage
- The trained autoencoder accepts 128x128 binary inputs and 256-unit hidden layers, and latent vectors are produced for all cells with the 20% default training subset held out
- Matlab and Python implementations yield largely similar results (per Supplemental Figure 6 comparison)

## Limitations

- The article provides no software version numbers for the Matlab or Python implementations, so reproducibility depends on repository/tutorial instructions
- MIRIAM segmentation quality (e.g. DSC 0.65 ± 0.08 against manual annotation) bounds the quality of the shapes fed to the autoencoder; segmentation errors propagate into shape encodings
- t-SNE analysis using only the autoencoder latent vectors, excluding training cells, did not reveal discernible cellular populations or clear patterns in marker expression, so shape encoding alone may not separate cell phenotypes

## Evidence

- [methods] Resize each aligned binary cell mask to a fixed 128x128 matrix to serve as the autoencoder input.: [evidence span withheld — anchor status: not_found]
- [methods] MIRIAM's cell shape autoencoder is trained on a random subset (20% default) of all segmented cells, run in Matlab using the Deep Learning Toolbox or the Keras library in Python, using 256 hidden layers.: [evidence span withheld — anchor status: not_found]
