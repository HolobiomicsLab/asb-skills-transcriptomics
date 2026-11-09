---
name: st-super-pixel-data-enhancement
description: Use when preparing a 10x Visium training reference for CeLEry 2D location
  recovery, e.g. the mouse posterior brain dataset, where spot-level resolution or
  a limited gene panel would limit spatial-location prediction accuracy.
when_to_use_negative:
- The task is spatial domain recovery rather than 2D location recovery — then differentially
  expressed genes are selected with rank_genes_groups in Scanpy instead of spatially
  variable genes in spaGCN
- The input is scRNA-seq test data, which is consumed by the CeLEry model but should
  not be TESLA-enhanced
- Cells have low UMI counts or a small number of expressed genes — such cells are
  eliminated rather than enhanced
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0202
tools:
- name: CeLEry
  role: Downstream supervised deep neural network that consumes the TESLA-enhanced,
    gene-selected, normalized ST reference for 2D location recovery
  repo: https://github.com/QihuangZhang/CeLEry
- name: spaGCN
  role: Provides the rank_genes_groups function used to select spatially variable
    genes for 2D location recovery
- name: 10x Visium
  role: Spatial transcriptomics platform supplying the spot-level ST data that is
    enhanced to super-pixel resolution
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:3133c3c764e1e197a16a496f890ba6f0d3a1a38068362ae00119ca1040f22c2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QihuangZhang/CeLEry
  tools:
  - 10x Visium
  - CeLEry
  - Tangram
  - novoSpaRc
  - spaGCN
  - spaOTsc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# st-super-pixel-data-enhancement

## Summary

Enhance 10x Visium spatial transcriptomics data to super-pixel resolution with TESLA and select spatially variable genes (SpaGCN rank_genes_groups) so that downstream CeLEry location-recovery training uses a denser, higher-quality spatial reference.

## When to use

Use when preparing a 10x Visium training reference for CeLEry 2D location recovery, e.g. the mouse posterior brain dataset, where spot-level resolution or a limited gene panel would limit spatial-location prediction accuracy.

## When NOT to use

- The task is spatial domain recovery rather than 2D location recovery — then differentially expressed genes are selected with rank_genes_groups in Scanpy instead of spatially variable genes in spaGCN
- The input is scRNA-seq test data, which is consumed by the CeLEry model but should not be TESLA-enhanced
- Cells have low UMI counts or a small number of expressed genes — such cells are eliminated rather than enhanced

## Inputs

- 10x Visium spatial transcriptomics dataset (e.g. mouse posterior brain ST data)
- Gene expression count matrix (UMI counts per spot)
- Spot spatial coordinates for the ST reference section

## Outputs

- TESLA-enhanced super-pixel-resolution spatial expression matrix
- Set of 358 spatially variable genes selected via spaGCN rank_genes_groups
- Two-step-normalized (spot log-normalized, gene-standardized) expression matrix ready for CeLEry training
- Optional VAE-generated replicate training data

## How to apply

Load the 10x Visium spatial transcriptomics data (e.g. mouse posterior brain) and enhance it to super-pixel resolution with TESLA before supervised training. Select spatially variable genes with the rank_genes_groups function in the spaGCN package (358 genes in the reference 2D location-recovery task). Normalize expression in two steps — spot-level log transformation of UMI counts, then gene-level standardization (subtract gene mean, divide by gene standard deviation across spots). Optionally augment the training spots with variational-autoencoder-generated replicates (D=512, beta=1e-5, 2D conv encoder with 16/8/4 channels), choosing the replicate number via cross-validation, so the enhanced reference feeds a robust CeLEry neural network (3 hidden layers of 50/10/5 nodes, ReLU activations, sigmoid output rescaled to [0,1]).

## Related tools

- **CeLEry** (Downstream supervised deep neural network that consumes the TESLA-enhanced, gene-selected, normalized ST reference for 2D location recovery) — https://github.com/QihuangZhang/CeLEry
- **spaGCN** (Provides the rank_genes_groups function used to select spatially variable genes for 2D location recovery)
- **10x Visium** (Spatial transcriptomics platform supplying the spot-level ST data that is enhanced to super-pixel resolution)

## Examples

```
pip install CeLEryPy
```

## Evaluation signals

- The selected gene set matches the expected scale (358 spatially variable genes chosen via spaGCN rank_genes_groups)
- Normalization invariant: after the second step, each selected gene has mean 0 and standard deviation 1 across spots
- Enhanced reference supports CeLEry pairwise-distance Pearson correlations that exceed Tangram, spaOTsc, and novoSpaRc at each holdout fraction (10/30/50%)
- Reconstructed 2D gene expression maps achieve higher structural similarity index to ground truth than baseline methods
- VAE augmentation replicates are only used for training spots; held-out super-pixels remain fully masked

## Limitations

- The optimal number of VAE-generated replicates is a user-specified hyperparameter requiring cross-validation rather than a fixed recommended setting
- CeLEry is suboptimal on single-cell-resolution platforms with small gene panels and would benefit from ST data with single-cell resolution and transcriptome-wide coverage
- No software versions are reported for CeLEry or the competing tools, so reproducibility of the exact enhancement stack is uncertain

## Evidence

- [methods] Spatially variable genes are selected using the rank_genes_groups function in the spaGCN package (for 2D location recovery): "Spatially variable genes are selected using the rank_genes_groups function in the spaGCN package (for 2D location recovery)"
- [methods] The gene expression normalization involves two steps. In the first step, cell/spot-level normalization is performed: "The gene expression normalization involves two steps. In the first step, cell/spot-level normalization is performed"
- [methods] we apply the decoder to reconstruct the spatial gene expression matrix, which will be combined with the original ST data for model training: "we apply the decoder to reconstruct the spatial gene expression matrix, which will be combined with the original ST data for model training"
- [readme] CeLEry takes spatial transcriptomic data as input for the training data and the scRNA-seq as testing data set.: "CeLEry takes spatial transcriptomic data as input for the training data and the scRNA-seq as testing data set."
- [readme] CeLEry optionally generates replicates of the spatial transcriptomic data via variational autoencoder then includes them as the training data together with original spatial transcriptomic data.: "CeLEry optionally generates replicates of the spatial transcriptomic data via variational autoencoder then includes them as the training data together with original spatial transcriptomic data."
