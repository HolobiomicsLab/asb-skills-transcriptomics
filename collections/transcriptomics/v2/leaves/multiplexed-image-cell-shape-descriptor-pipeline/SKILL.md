---
name: multiplexed-image-cell-shape-descriptor-pipeline
description: Use when segmented single-cell masks from multiplexed tissue imaging
  need to be converted into quantitative shape features via an autoencoder, e.g. for
  t-SNE or morphology-based cell characterization.
when_to_use_negative:
- Input is already a feature/latent table (e.g. marker intensity matrix) rather than
  raw segmented cell masks — use marker-intensity t-SNE instead
- A deep learning-based shape descriptor is not needed and simple morphometrics suffice;
  note the paper's own t-SNE shape analysis 'did not reveal any discernible cellular
  populations or clear patterns in marker expression'
- Nuclei detection/segmentation is the goal — the article states deep learning methods
  for nuclei detection 'were not implemented' in MIRIAM
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Upstream pipeline that produces the segmented cell masks and hosts the autoencoder
    cell shape descriptor
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Implementation environment using the Deep Learning Toolbox to train and run
    the autoencoder
- name: Python
  role: Alternative implementation using the Keras library for the autoencoder; results
    reported as largely similar to Matlab
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:031364708ebe7c85e860c9b74a98ae623425b36b7e9e81055a3cd2ccba8ee1a1
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# multiplexed-image-cell-shape-descriptor-pipeline

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements MIRIAM's deep learning-based cell shape descriptor: segmented cell masks are binarized, principal-axis aligned, resized to 128x128, and passed through an autoencoder with 256-unit hidden layers to yield latent shape vectors for downstream analyses like t-SNE.

## When to use

Use when segmented single-cell masks from multiplexed tissue imaging need to be converted into quantitative shape features via an autoencoder, e.g. for t-SNE or morphology-based cell characterization.

## When NOT to use

- Input is already a feature/latent table (e.g. marker intensity matrix) rather than raw segmented cell masks — use marker-intensity t-SNE instead
- A deep learning-based shape descriptor is not needed and simple morphometrics suffice; note the paper's own t-SNE shape analysis 'did not reveal any discernible cellular populations or clear patterns in marker expression'
- Nuclei detection/segmentation is the goal — the article states deep learning methods for nuclei detection 'were not implemented' in MIRIAM

## Inputs

- Segmented cell masks from MIRIAM segmentation (binary mask images)
- Random subset (default 20%) of segmented cells for autoencoder training

## Outputs

- latent_vectors.csv containing one encoded latent vector per segmented cell
- Trained autoencoder model weights (autoencoder_model.h5 or model.mat)

## How to apply

Load the segmented cell masks produced by the upstream MIRIAM segmentation stage, binarize each mask, and rotate it so its principal (major) axis is aligned horizontally. Resize each aligned binary mask to a fixed 128x128 matrix to serve as the autoencoder input. Train the autoencoder with 256-unit hidden layers on a random subset (default 20%) of all segmented cells, using the Matlab Deep Learning Toolbox or the equivalent Keras implementation in Python (the article reports largely similar results for the Matlab and Python implementations). Pass every cell shape matrix through the trained encoder and record the encoded latent vectors, excluding cells used to train the autoencoder from downstream analysis subsets (e.g. t-SNE input in Fig. 3B). Save latent vectors to latent_vectors.csv and persist model weights (autoencoder_model.h5 / model.mat).

## Related tools

- **MIRIAM** (Upstream pipeline that produces the segmented cell masks and hosts the autoencoder cell shape descriptor) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Implementation environment using the Deep Learning Toolbox to train and run the autoencoder)
- **Python** (Alternative implementation using the Keras library for the autoencoder; results reported as largely similar to Matlab)

## Evaluation signals

- latent_vectors.csv exists and contains one latent vector per segmented cell
- Latent vectors reflect the specified 256-unit hidden-layer encoding
- Autoencoder satisfies its input/output contract: 128x128 binary input matrices, latent-vector outputs from the trained encoder
- Cells used for autoencoder training (20% random subset) are excluded from downstream analysis subsets such as t-SNE (Fig. 3B)
- Matlab and Python implementations yield largely similar results (per Supplemental Figure 6 comparison)

## Limitations

- t-SNE shape analysis using only autoencoder latent vectors did not reveal any discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5)
- No software version numbers are stated for the Matlab or Python implementations
- Pipeline requires Ilastik (Iltastik) for some tasks per the README
- Example data in the repository omits image stacks (too large); users must regenerate them

## Evidence
