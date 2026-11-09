---
name: autoencoder-latent-vector-extraction
description: Use when segmented single-cell masks from multiplexed tissue imaging
  need quantitative shape descriptors, e.g., feeding cell-shape latent vectors into
  t-SNE analysis downstream of MIRIAM segmentation.
when_to_use_negative:
- Analysis goal is t-SNE on marker intensities rather than cell shape — the article
  treats intensity-based t-SNE (Fig. 3A) as a separate input modality
- No cell segmentation has been performed yet; this skill requires MIRIAM segmented
  cell masks as input, not raw multiplexed image stacks
- Latent vectors already exist in latent_vectors.csv for all cells — re-training the
  autoencoder is unnecessary
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Upstream pipeline whose segmentation stage produces the binary cell masks
    that are the autoencoder's training and encoding input
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Primary implementation environment; autoencoder trained with the Matlab Deep
    Learning Toolbox, weights saved as model.mat
- name: Python
  role: Alternative implementation using the Keras library, producing equivalent results
    and weights as autoencoder_model.h5
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:5e1742fa66fecc844b7c9447469d5a49a8dde8ffe67ac92394d21cec592e0f99
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
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# autoencoder-latent-vector-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains an autoencoder neural network on binary segmented cell masks from the MIRIAM pipeline to produce compact latent-vector encodings of cell shape, implemented in Matlab (Deep Learning Toolbox) or Python (Keras) with 256-unit hidden layers.

## When to use

Use when segmented single-cell masks from multiplexed tissue imaging need quantitative shape descriptors, e.g., feeding cell-shape latent vectors into t-SNE analysis downstream of MIRIAM segmentation.

## When NOT to use

- Analysis goal is t-SNE on marker intensities rather than cell shape — the article treats intensity-based t-SNE (Fig. 3A) as a separate input modality
- No cell segmentation has been performed yet; this skill requires MIRIAM segmented cell masks as input, not raw multiplexed image stacks
- Latent vectors already exist in latent_vectors.csv for all cells — re-training the autoencoder is unnecessary

## Inputs

- Segmented cell masks produced by the upstream MIRIAM segmentation stage (binary cell masks per cell)
- Matlab Deep Learning Toolbox environment, or a Python environment with the Keras library

## Outputs

- latent_vectors.csv with one encoded latent vector per segmented cell (256-unit hidden-layer encoding)
- Trained model weights persisted as autoencoder_model.h5 (Python/Keras) or model.mat (Matlab)

## How to apply

Load the segmented cell masks from upstream MIRIAM segmentation, binarize each mask, and rotate it so its principal (major) axis is aligned horizontally. Resize each aligned binary mask to a fixed 128x128 matrix that serves as the autoencoder input. Train the autoencoder with 256-unit hidden layers using Matlab's Deep Learning Toolbox or the equivalent Keras implementation in Python, training on a random subset of segmented cells (20% default per the Materials and Methods). Pass every cell shape matrix through the trained encoder and record the latent vectors, excluding those cells used to train the autoencoder from downstream analysis subsets such as t-SNE. Save the latent vectors to latent_vectors.csv and persist the trained model weights (autoencoder_model.h5 for Python/Keras or model.mat for Matlab).

## Related tools

- **MIRIAM** (Upstream pipeline whose segmentation stage produces the binary cell masks that are the autoencoder's training and encoding input) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Primary implementation environment; autoencoder trained with the Matlab Deep Learning Toolbox, weights saved as model.mat)
- **Python** (Alternative implementation using the Keras library, producing equivalent results and weights as autoencoder_model.h5)

## Evaluation signals

- latent_vectors.csv exists and contains one latent vector per segmented cell
- Encoding uses the specified 256-unit hidden-layer architecture
- Autoencoder satisfies the input/output contract: 128x128 binary input matrices mapped to trained-encoder latent-vector outputs
- Training set is a random held-out subset (~20% default) of segmented cells; those cells are excluded from downstream t-SNE subsets
- Latent vectors are valid inputs to t-SNE cell-shape analysis as in Fig. 3B

## Limitations

- t-SNE shape analysis using only the latent vectors 'did not reveal any discernible cellular populations or clear patterns in marker expression' (Supplemental Figure 5), so shape encoding may not separate biologically meaningful populations
- No software version numbers are stated for the Matlab or Python implementations, complicating exact reproduction
- Matlab and Python implementations were only shown to give 'largely similar results' (Supplemental Figure 6), not identical ones

## Evidence

- [intro] Finally, cell shapes are characterized using an autoencoder neural network.: "Finally, cell shapes are characterized using an autoencoder neural network."
- [results] When only the encoded latent vectors for cell shape, excluding those used to train the autoencoder, are used as an input to t-SNE analysis (Fig. 3B): "When only the encoded latent vectors for cell shape, excluding those used to train the autoencoder, are used as an input to t-SNE analysis (Fig. 3B)"
- [discussion] Results of MIRIAM using Matlab or Python implementations show largely similar results (Supplemental Figure 6).: "Results of MIRIAM using Matlab or Python implementations show largely similar results (Supplemental Figure 6)."
