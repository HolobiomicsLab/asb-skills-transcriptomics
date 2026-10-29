---
name: cell-mask-binarization-and-major-axis-alignment
description: Use when you have segmented single-cell masks from multiplexed tissue
  images and need to build the autoencoder cell shape descriptor inputs described
  by MIRIAM (binarize, rotate major axis horizontal, resize to 128x128).
when_to_use_negative:
- Input is already a normalized feature or latent-vector table rather than raw segmented
  cell masks — go directly to downstream analysis (e.g., t-SNE) instead.
- Cells lack complete nuclear/membrane segmentation, since the shape descriptor depends
  on well-defined single-cell masks from the MIRIAM segmentation stage.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Pipeline whose segmentation stage produces the cell masks that are binarized,
    aligned, and resized for the autoencoder shape descriptor
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Implementation environment using the Deep Learning Toolbox to train the autoencoder
    on the aligned 128x128 binary masks
- name: Python
  role: Alternative implementation environment providing the Keras equivalent of the
    Matlab autoencoder
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:1811406401fcad1fbb8d9362fc8953497a2c8ee1aaa781469a57a32f6f73a55d
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
  - Python
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cell-mask-binarization-and-major-axis-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocesses segmented single-cell masks from the MIRIAM pipeline into normalized 128x128 binary matrices for training the autoencoder cell shape descriptor. Binarization, major-axis alignment, and fixed-size resizing give every cell a canonical orientation and scale so latent shape vectors are comparable across cells.

## When to use

Use when you have segmented single-cell masks from multiplexed tissue images and need to build the autoencoder cell shape descriptor inputs described by MIRIAM (binarize, rotate major axis horizontal, resize to 128x128).

## When NOT to use

- Input is already a normalized feature or latent-vector table rather than raw segmented cell masks — go directly to downstream analysis (e.g., t-SNE) instead.
- Cells lack complete nuclear/membrane segmentation, since the shape descriptor depends on well-defined single-cell masks from the MIRIAM segmentation stage.

## Inputs

- Segmented single-cell masks (binary/cell label masks from MIRIAM segmentation of multiplexed imaging data)
- Random subset (~20% default) of segmented cells selected for autoencoder training

## Outputs

- 128x128 binary cell shape matrices, major-axis aligned and resized
- Trained autoencoder model weights (autoencoder_model.h5 or model.mat)
- latent_vectors.csv with one encoded latent vector per segmented cell

## How to apply

Load the cell masks produced by upstream MIRIAM segmentation and binarize each single-cell mask. Rotate each binary mask so its principal (major) axis is aligned horizontally, removing orientation as a confounder for shape encoding. Resize each aligned binary mask to a fixed 128x128 matrix, which serves as the autoencoder input. In the MIRIAM workflow, the autoencoder is trained with 256-unit hidden layers in Matlab (Deep Learning Toolbox) or the equivalent Keras implementation in Python, using a random subset (20% default) of all segmented cells as training data. The trained encoder then maps every cell shape matrix to a latent vector; vectors from cells excluded from training are used downstream (e.g., as t-SNE inputs).

## Related tools

- **MIRIAM** (Pipeline whose segmentation stage produces the cell masks that are binarized, aligned, and resized for the autoencoder shape descriptor) — https://github.com/Coffey-Lab/MIRIAM
- **Matlab** (Implementation environment using the Deep Learning Toolbox to train the autoencoder on the aligned 128x128 binary masks)
- **Python** (Alternative implementation environment providing the Keras equivalent of the Matlab autoencoder)

## Examples

```
model.save('autoencoder_model.h5')
```

## Evaluation signals

- Each processed cell mask is a 128x128 binary matrix with the cell's major axis aligned horizontally.
- The autoencoder uses 256-unit hidden layers and is trained on a random subset (20% default) of segmented cells.
- latent_vectors.csv exists and contains one latent vector per segmented cell with the specified 256-unit encoding.
- Trained model weights persist in the specified format (autoencoder_model.h5 for Keras/Python or model.mat for Matlab).
- Downstream t-SNE on shape latent vectors excludes the cells used to train the autoencoder.

## Limitations

- MIRIAM's t-SNE shape analysis did not reveal discernible cellular populations or clear patterns in marker expression, so latent shape vectors may not always yield biologically separable clusters.
- No software version numbers are stated for the Matlab or Python implementations, so reproducibility of the exact training environment is not guaranteed.
- The README provides only high-level setup guidance (Matlab code, ilastik dependency, Dropbox example data) without explicit preprocessing scripts for mask alignment.

## Evidence

- [methods] Binarize each segmented cell mask and rotate it so its principal (major) axis is aligned horizontally.: [evidence span withheld — anchor status: not_found]
- [methods] Resize each aligned binary cell mask to a fixed 128x128 matrix to serve as the autoencoder input.: [evidence span withheld — anchor status: not_found]
- [methods] MIRIAM's cell shape autoencoder is trained on a random subset (20% default) of all segmented cells, run in Matlab using the Deep Learning Toolbox or the Keras library in Python, using 256 hidden layers.: [evidence span withheld — anchor status: not_found]
- [intro] Finally, cell shapes are characterized using an autoencoder neural network.: "Finally, cell shapes are characterized using an autoencoder neural network."
- [results] When only the encoded latent vectors for cell shape, excluding those used to train the autoencoder, are used as an input to t-SNE analysis (Fig. 3B): "When only the encoded latent vectors for cell shape, excluding those used to train the autoencoder, are used as an input to t-SNE analysis"
