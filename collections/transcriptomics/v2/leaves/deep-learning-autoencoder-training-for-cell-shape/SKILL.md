---
name: deep-learning-autoencoder-training-for-cell-shape
description: Use when segmented single-cell binary masks are available and a morphology-only,
  label-free shape descriptor is needed for downstream analyses such as t-SNE clustering
  of cell shapes.
when_to_use_negative:
- When the goal is cell segmentation or marker intensity quantification rather than
  shape encoding — use MIRIAM's random forest pixel classification and membrane-extension
  steps instead.
- When the input is already a per-cell feature table (e.g., marker intensities or
  precomputed shape descriptors) rather than binary segmented cell masks.
- 'When downstream analysis requires marker-expression patterns: the article reports
  that t-SNE on shape latent vectors alone did not reveal discernible cellular populations
  or clear marker-expression patterns.'
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Pipeline providing the upstream segmented cell masks and the autoencoder-based
    cell shape descriptor stage
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Implementation environment for the autoencoder via the Matlab Deep Learning
    Toolbox, saving model.mat
- name: Python
  role: Alternative implementation environment using the Keras library, saving autoencoder_model.h5
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:e9500d38fb6bebb6add88024885f4ba8c8972b4a4b36918f716d4150ec8ef967
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
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# deep-learning-autoencoder-training-for-cell-shape

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains an autoencoder neural network on binary segmented cell masks to produce a deep learning-based cell shape descriptor, as implemented in the MIRIAM pipeline (Matlab Deep Learning Toolbox or Keras in Python). Each mask is binarized, rotated to align its principal axis horizontally, resized to 128x128, and encoded into a latent vector.

## When to use

Use when segmented single-cell binary masks are available and a morphology-only, label-free shape descriptor is needed for downstream analyses such as t-SNE clustering of cell shapes.

## When NOT to use

- When the goal is cell segmentation or marker intensity quantification rather than shape encoding — use MIRIAM's random forest pixel classification and membrane-extension steps instead.
- When the input is already a per-cell feature table (e.g., marker intensities or precomputed shape descriptors) rather than binary segmented cell masks.
- When downstream analysis requires marker-expression patterns: the article reports that t-SNE on shape latent vectors alone did not reveal discernible cellular populations or clear marker-expression patterns.

## Inputs

- Segmented binary cell masks from the MIRIAM segmentation stage
- Random training subset of segmented cells (default 20%)

## Outputs

- latent_vectors.csv with one 256-unit latent vector per segmented cell
- Trained autoencoder model weights (autoencoder_model.h5 for Keras / model.mat for Matlab)
- Encoded shape latent vectors usable as t-SNE input, excluding training cells

## How to apply

Load segmented cell masks from the upstream MIRIAM segmentation stage, binarize each mask, and rotate it so its principal (major) axis is aligned horizontally to remove orientation variance. Resize each aligned mask to a fixed 128x128 matrix that serves as the autoencoder input. Train the autoencoder with 256-unit hidden layers using the Matlab Deep Learning Toolbox or the equivalent Keras implementation in Python, training on a random subset of cells (20% default per the Materials and Methods specification). Pass every cell shape matrix through the trained encoder, record the latent vectors for all cells, and exclude the cells used to train the autoencoder from downstream analysis subsets (e.g., when feeding shape latent vectors into t-SNE). Save latent vectors to latent_vectors.csv and persist model weights as autoencoder_model.h5 (Keras) or model.mat (Matlab).

## Related tools

- **MIRIAM** (Pipeline providing the upstream segmented cell masks and the autoencoder-based cell shape descriptor stage) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Implementation environment for the autoencoder via the Matlab Deep Learning Toolbox, saving model.mat)
- **Python** (Alternative implementation environment using the Keras library, saving autoencoder_model.h5)

## Examples

```
from keras.layers import Dense, Input; inp = Input((128*128,)); enc = Dense(256, activation='relu')(inp)  # Keras autoencoder on 128x128 masks
```

## Evaluation signals

- latent_vectors.csv exists and contains one latent vector per segmented cell with the specified 256-unit hidden-layer encoding
- Autoencoder input/output contract holds: 128x128 binary mask inputs, latent-vector outputs from the trained encoder
- Cells used to train the autoencoder (the 20% random subset) are excluded from the latent vectors fed to downstream analyses such as t-SNE
- Matlab and Python implementations yield largely similar results (per Supplemental Figure 6 comparison in the article)

## Limitations

- The article's t-SNE shape analysis did not reveal any discernible cellular populations or clear patterns in marker expression, limiting interpretive value of shape-only latent vectors.
- No software version numbers are stated for the Matlab or Python implementations.
- The repository README describes the code as Matlab-based and points users to a tutorial and ilastik for some tasks; Python/Keras behavior must be verified against the tutorial.

## Evidence
