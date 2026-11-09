---
name: celery-2d-location-prediction-training
description: Use when you need to recover 2D spatial coordinates of scRNA-seq cells
  (or held-out ST spots) by training a supervised model on a reference ST dataset
  with known locations, e.g. 10x Visium mouse brain sections.
when_to_use_negative:
- The goal is spatial domain/layer prediction rather than 2D coordinates — use the
  categorical spatial-domain recovery variant instead (Scanpy rank_genes_groups DE
  genes).
- Only single-cell-resolution ST data with a small gene panel is available — CeLEry
  was noted to be suboptimal on single-cell-resolution platforms with small gene panels.
- No spatial transcriptomics training data with known locations exists to learn from.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3170
tools:
- name: CeLEry
  role: Supervised deep neural network trained to learn the relationship between spotwise
    gene expression and 2D spatial locations; installed via pip install CeLEryPy
  repo: https://github.com/QihuangZhang/CeLEry
- name: spaGCN
  role: Provides the rank_genes_groups function used to select spatially variable
    genes for the 2D location recovery task
- name: TESLA
  role: Enhances 10x Visium spot data to super-pixel resolution used as the ST training
    reference
- name: 10x Visium
  role: Source platform of the spatial transcriptomics training data (mouse posterior
    brain) with known spot coordinates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:d5487f3d2cbe23e8100b435c3106a8fe6903eefbe756e0dd376deae40607b517
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QihuangZhang/CeLEry
  tools:
  - 10x Visium
  - CeLEry
  - TESLA
  - Tangram
  - novoSpaRc
  - spaGCN
  - spaOTsc
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# celery-2d-location-prediction-training

## Summary

Trains a CeLEry supervised deep neural network on spatial transcriptomics (ST) data to predict 2D coordinates of held-out spots/cells in scRNA-seq, using optional VAE-based data augmentation to improve robustness against noise.

## When to use

Use when you need to recover 2D spatial coordinates of scRNA-seq cells (or held-out ST spots) by training a supervised model on a reference ST dataset with known locations, e.g. 10x Visium mouse brain sections.

## When NOT to use

- The goal is spatial domain/layer prediction rather than 2D coordinates — use the categorical spatial-domain recovery variant instead (Scanpy rank_genes_groups DE genes).
- Only single-cell-resolution ST data with a small gene panel is available — CeLEry was noted to be suboptimal on single-cell-resolution platforms with small gene panels.
- No spatial transcriptomics training data with known locations exists to learn from.

## Inputs

- 10x Visium spatial transcriptomics data with known spot coordinates (e.g., mouse posterior brain, optionally TESLA-enhanced super-pixels)
- scRNA-seq query data (held-out spots/cells) with matched genes
- Spatially variable gene list selected via spaGCN rank_genes_groups (e.g., 358 genes for 2D location recovery)

## Outputs

- Predicted 2D coordinates for each held-out spot/cell
- Uncertainty estimates (prediction regions) for recovered locations
- Trained CeLEry deep neural network model

## How to apply

Start from an ST reference such as 10x Visium mouse posterior brain data, optionally enhanced to super-pixel resolution with TESLA, and select spatially variable genes using the rank_genes_groups function in spaGCN (358 SVGs for 2D location recovery). Normalize expression in two steps: spot-level log transformation of UMI counts followed by gene-level standardization (subtract mean, divide by gene SD across spots). Mask a holdout fraction of super-pixels (10/30/50%) as test data and optionally augment remaining training spots with variational-autoencoder replicates (D=512, beta=1e-5, 2D conv encoder with 16/8/4 channels); choose replicate number via cross-validation. Train the CeLEry supervised deep neural network (3 hidden layers of 50, 10, and 5 nodes, ReLU activations, sigmoid output rescaled to [0,1]) to predict 2D coordinates by minimizing summed squared-error loss on true spot coordinates, with output layer dimension 2 for 2D recovery.

## Related tools

- **CeLEry** (Supervised deep neural network trained to learn the relationship between spotwise gene expression and 2D spatial locations; installed via pip install CeLEryPy) — https://github.com/QihuangZhang/CeLEry
- **spaGCN** (Provides the rank_genes_groups function used to select spatially variable genes for the 2D location recovery task)
- **TESLA** (Enhances 10x Visium spot data to super-pixel resolution used as the ST training reference)
- **10x Visium** (Source platform of the spatial transcriptomics training data (mouse posterior brain) with known spot coordinates)

## Examples

```
pip install CeLEryPy
```

## Evaluation signals

- Pearson correlation between pairwise distances of predicted vs. true spot coordinates should exceed that of Tangram, spaOTsc, and novoSpaRc at each holdout fraction (10/30/50%), matching Fig. 5a-c ordering
- Reconstructed 2D gene expression maps should show higher structural similarity index (SSIM) to ground truth than baseline methods
- Two-step normalization must be applied: spot-level log UMI transform then gene-level standardization (mean 0, SD 1 across spots)
- Loss on held-out spots (summed squared error against true coordinates) decreases and converges during training
- Number of VAE replicates chosen via cross-validation, not set arbitrarily

## Limitations

- CeLEry is suboptimal on single-cell-resolution platforms with small gene panels; it will perform better with ST data of single-cell resolution and transcriptome-wide coverage
- The number of VAE replicates is a user-specified hyperparameter requiring cross-validation; no fixed optimal value is recommended
- Spatially variable gene selection depends on the spatial platform quality; how many gradient genes exist in tissues is unknown

## Evidence

- [methods] Spatially variable genes are selected using the rank_genes_groups function in the spaGCN package (for 2D location recovery): "Spatially variable genes are selected using the rank_genes_groups function in the spaGCN package (for 2D location recovery)"
- [methods] If the goal is to predict the 2D coordinates and its associated prediction region, the dimension of the output layer is 2: "If the goal is to predict the 2D coordinates and its associated prediction region, the dimension of the output layer is 2"
- [methods] The gene expression normalization involves two steps. In the first step, cell/spot-level normalization is performed: "The gene expression normalization involves two steps. In the first step, cell/spot-level normalization is performed"
- [methods] we propose a data augmentation procedure to generate “replicates” of the original ST data: "we propose a data augmentation procedure to generate “replicates” of the original ST data"
- [readme] The `CeLEry` package requires only a standard computer with enough RAM to support the operations defined by a user.: "The `CeLEry` package requires only a standard computer with enough RAM to support the operations defined by a user."
- [abstract] CeLEry can infer the spatial origins of cells in scRNA-seq at multiple levels, including 2D location and spatial domain of a cell: "CeLEry can infer the spatial origins of cells in scRNA-seq at multiple levels, including 2D location and spatial domain of a cell"
