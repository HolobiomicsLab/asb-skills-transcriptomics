---
name: 2d-location-regression-neural-network-training
description: Use when you have ST data with spot-level gene expression and true 2D
  coordinates and need to predict 2D locations of scRNA-seq cells or held-out test
  spots by supervised regression.
when_to_use_negative:
- The goal is spatial domain / tissue layer prediction rather than continuous 2D coordinates
  — that requires the categorical-output spatial domain recovery model instead
- The target ST platform is a single-cell-resolution platform with a small gene panel;
  the article notes CeLEry is suboptimal in this setting
- No spatially variable genes or true spatial locations are available in the training
  ST data
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3308
tools:
- name: CeLEry
  role: Implements the supervised deep neural network for 2D location prediction and
    the optional VAE data augmentation
  repo: https://github.com/QihuangZhang/CeLEry
- name: spaGCN
  role: Provides the rank_genes_groups function used to select spatially variable
    genes as network input features for 2D location recovery
- name: Scanpy
  role: Used for preprocessing ST data and rank_genes_groups for differentially expressed
    gene selection (spatial domain recovery variant)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:54c834b88159fd39bdf5d7571ddc0b1fcca17036c39e7c5cb577d8cd97008b30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QihuangZhang/CeLEry
  tools:
  - CeLEry
  - Scanpy
  - spaGCN
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# 2d-location-regression-neural-network-training

## Summary

Train CeLEry's supervised deep neural network to regress 2D spatial coordinates of spots/cells from spatially variable gene expression, using spatial transcriptomics data as training input and optionally VAE-augmented ST replicates to improve robustness.

## When to use

Use when you have ST data with spot-level gene expression and true 2D coordinates and need to predict 2D locations of scRNA-seq cells or held-out test spots by supervised regression.

## When NOT to use

- The goal is spatial domain / tissue layer prediction rather than continuous 2D coordinates — that requires the categorical-output spatial domain recovery model instead
- The target ST platform is a single-cell-resolution platform with a small gene panel; the article notes CeLEry is suboptimal in this setting
- No spatially variable genes or true spatial locations are available in the training ST data

## Inputs

- Preprocessed spatial transcriptomics training data (spot-level gene expression with true 2D coordinates, e.g. 10x Visium .h5ad)
- Held-out test spots or scRNA-seq cells with gene expression (and true 2D coordinates for validation)
- Spatially variable gene list from spaGCN rank_genes_groups
- Optional VAE-augmented ST replicates (e.g. 2 or 10 replicates)

## Outputs

- Trained CeLEry 2D location prediction model
- Predicted 2D coordinates for each test spot/cell, rescaled to [0,1]
- Mean squared error (MSE) against true coordinates (e.g. augmentation_noise_mse.csv)

## How to apply

Select spatially variable genes from the preprocessed ST training data using the rank_genes_groups function in spaGCN. Train a supervised network with 3 hidden layers of 50, 10, and 5 nodes, ReLU activations, and a 2-dimensional output rescaled to [0,1] via Sigmoid, minimizing the summed squared-error loss on spot coordinates via gradient descent. Optionally, generate ST replicates with a VAE (embedding dimension 512, KL weight beta = 1e-5) and combine them with the original ST data for training to improve robustness to noise; choose the number of replicates via cross-validation. Predict 2D coordinates for held-out spots or scRNA-seq cells and compute mean squared error against true coordinates to validate.

## Related tools

- **CeLEry** (Implements the supervised deep neural network for 2D location prediction and the optional VAE data augmentation) — https://github.com/QihuangZhang/CeLEry
- **spaGCN** (Provides the rank_genes_groups function used to select spatially variable genes as network input features for 2D location recovery)
- **Scanpy** (Used for preprocessing ST data and rank_genes_groups for differentially expressed gene selection (spatial domain recovery variant))

## Examples

```
pip install CeLEryPy
```

## Evaluation signals

- MSE against true 2D coordinates should be low and decrease relative to baselines; compile MSE-versus-noise tables for validation
- With Gaussian noise added to test gene expression, augmented-model (2 and 10 VAE replicates) MSE advantage over ordinary CeLEry should widen monotonically with increasing noise standard deviation (Fig. 5d pattern)
- Predicted 2D output must lie in [0,1] after the Sigmoid output rescaling
- Accept the result if augmented models' MSE advantage over ordinary CeLEry widens monotonically as the noise standard deviation increases
- Use cross-validation over the number of VAE replicates (a user-specified hyperparameter) to pick the best setting

## Limitations

- The number of VAE replicates is a hyperparameter prespecified by users and must be tuned via cross-validation; no fixed optimal setting is recommended
- CeLEry is suboptimal on single-cell-resolution platforms with small gene panels; performance is expected to improve when transcriptome-wide, single-cell-resolution ST data become available
- Seurat-style reliance on location-specific anchor genes limits applicability when anchor gene information is unavailable (motivating the learned regression instead)

## Evidence

- [methods] Spatially variable genes are selected using the rank_genes_groups function in the spaGCN package (for 2D location recovery): "Spatially variable genes are selected using the rank_genes_groups function in the spaGCN package (for 2D location recovery)"
- [methods] If the goal is to predict the 2D coordinates and its associated prediction region, the dimension of the output layer is 2: "If the goal is to predict the 2D coordinates and its associated prediction region, the dimension of the output layer is 2"
- [methods] We train a supervised deep neural network to learn the relationship between gene expression features and the spatial locations in the ST dataset: "We train a supervised deep neural network to learn the relationship between gene expression features and the spatial locations in the ST dataset"
- [discussion] we recommend that the user explore the best number of replicates by performing cross-validation: "we recommend that the user explore the best number of replicates by performing cross-validation"
- [readme] CeLEry has a data augmentation procedure via variational autoencoder to improve the robustness of the method and overcome noise in scRNA-seq.: "CeLEry has a data augmentation procedure via variational autoencoder to improve the robustness of the method and overcome noise in scRNA-seq."
