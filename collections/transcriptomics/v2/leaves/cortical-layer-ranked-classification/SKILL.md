---
name: cortical-layer-ranked-classification
description: Use when scRNA-seq/snRNA-seq cells must be assigned to ordered tissue
  layers or spatial domains (e.g., human DLPFC cortical layers) and Visium ST sections
  with layer labels are available as training data.
when_to_use_negative:
- The goal is continuous 2D coordinate prediction rather than ordered spatial-domain/layer
  labels — use CeLEry's 2D location recovery mode with spaGCN spatially variable genes
  instead
- Input ST data has single-cell resolution but only a small gene panel (e.g., MERFISH/Xenium
  panels) — the article notes CeLEry is suboptimal on single-cell-resolution platforms
  with small gene panels
- No spatially referenced training data with domain labels exist — the model learns
  expression-location relationships from labeled ST, which is a prerequisite
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: CeLEry
  role: Supervised deep neural network trained with rank-consistent logistic regression
    loss for ordered spatial-domain (cortical layer) prediction; installed via pip
    install CeLEryPy
  repo: github.com/QihuangZhang/CeLEry
- name: Scanpy
  role: Provides rank_genes_groups for differentially expressed gene selection used
    in spatial domain recovery
- name: spaGCN
  role: Provides rank_genes_groups for spatially variable gene selection (used for
    2D location recovery, contrasted with domain recovery)
- name: Tangram
  role: Baseline comparator for spatial domain recovery accuracy
- name: spaOTsc
  role: Baseline comparator (optimal transport spatial mapping)
- name: novoSpaRc
  role: Baseline comparator (optimal transport spatial mapping)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:903231c635220ff89f6e80e6bf1a49ef9cd3c48a0a4e3822c6bcdee47f506652
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CeLEry
  - Scanpy
  - Tangram
  - novoSpaRc
  - spaGCN
  - spaOTsc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cortical-layer-ranked-classification

## Summary

A supervised deep-learning skill from CeLEry that predicts the spatial domain (e.g., cortical layer) of scRNA-seq cells by training on 10x Visium spatial transcriptomics data with a rank-consistent logistic regression loss for ordered layer labels.

## When to use

Use when scRNA-seq/snRNA-seq cells must be assigned to ordered tissue layers or spatial domains (e.g., human DLPFC cortical layers) and Visium ST sections with layer labels are available as training data.

## When NOT to use

- The goal is continuous 2D coordinate prediction rather than ordered spatial-domain/layer labels — use CeLEry's 2D location recovery mode with spaGCN spatially variable genes instead
- Input ST data has single-cell resolution but only a small gene panel (e.g., MERFISH/Xenium panels) — the article notes CeLEry is suboptimal on single-cell-resolution platforms with small gene panels
- No spatially referenced training data with domain labels exist — the model learns expression-location relationships from labeled ST, which is a prerequisite

## Inputs

- 10x Visium spatial transcriptomics section(s) with spot-level UMI counts and layer/spatial-domain labels (e.g., LIBD human DLPFC)
- scRNA-seq or snRNA-seq expression matrix as the test set
- Gene selection inputs: Scanpy AnnData for rank_genes_groups-based DE gene selection (spatial domain recovery)

## Outputs

- Predicted spatial domain / cortical layer label per scRNA-seq cell with uncertainty estimates
- Top-1 and top-2 accuracy metrics (e.g., benchmark_accuracy_table.csv comparing CeLEry vs. Tangram, spaOTsc, novoSpaRc under Scenarios 1-4)

## How to apply

Normalize the Visium training sections with the two-step procedure (spot/cell-level log transform of UMI counts, then gene-level z-scoring). For spatial domain recovery, select differentially expressed genes with rank_genes_groups in Scanpy (rather than spaGCN's spatially variable genes, which serve 2D location recovery); for multi-section training, normalize each section separately, subset the common genes shared by all sections, and merge expression with layer labels. Train the CeLEry deep neural network (3 hidden layers of 50/10/5 nodes, ReLU activations, Sigmoid output rescaled to [0,1]) minimizing the rank-consistent logistic regression loss that respects the ordered nature of cortical layers. Optionally add VAE-generated ST replicates (K-means one-hot gene grouping; beta=1e-5, 512-dim embedding, 16/8/4-channel convolutional encoder), tuning replicate number by cross-validation, which lifted top-1 accuracy from 53.8% to 63.2% (Scenario 1) and 44.0% to 51.2% (Scenario 2) on the LIBD DLPFC benchmark.

## Related tools

- **CeLEry** (Supervised deep neural network trained with rank-consistent logistic regression loss for ordered spatial-domain (cortical layer) prediction; installed via pip install CeLEryPy) — github.com/QihuangZhang/CeLEry
- **Scanpy** (Provides rank_genes_groups for differentially expressed gene selection used in spatial domain recovery)
- **spaGCN** (Provides rank_genes_groups for spatially variable gene selection (used for 2D location recovery, contrasted with domain recovery))
- **Tangram** (Baseline comparator for spatial domain recovery accuracy)
- **spaOTsc** (Baseline comparator (optimal transport spatial mapping))
- **novoSpaRc** (Baseline comparator (optimal transport spatial mapping))

## Examples

```
pip install CeLEryPy
```

## Evaluation signals

- Reproduced top-1/top-2 accuracy should match reported values: CeLEry 53.8/89.2% (Scenario 1) and 44.0/82.6% (Scenario 2); augmented 63.2/92.1% and 51.2/81.6%
- CeLEry should outperform Tangram, spaOTsc, and novoSpaRc under the same preprocessed training/test splits (per Fig. 3d baselines)
- Data augmentation with VAE replicates should improve accuracy when training uses only one tissue section (Scenarios 1-2); verify by sweeping replicate number via cross-validation
- Output layer must produce a categorical/ordered prediction with the rank-consistent loss; sanity-check that predicted layers respect the ordered-layer structure (top-2 accuracy substantially exceeds top-1, e.g., 92.1% vs 63.2%)

## Limitations

- The number of VAE-generated replicates is a hyperparameter prespecified by users and requires cross-validation rather than a fixed recommended setting
- CeLEry is suboptimal on single-cell-resolution platforms with small gene panels; performance is expected to improve with transcriptome-wide single-cell-resolution ST data
- Uncertainty in how many spatially variable gradient genes exist in tissues; metagene-based gradient identification remains future work

## Evidence

- [methods] Spatially variable genes are selected using the rank_genes_groups function in the spaGCN package (for 2D location recovery): "Spatially variable genes are selected using the rank_genes_groups function in the spaGCN package (for 2D location recovery)"
- [methods] differentially expressed genes are selected using the rank_genes_groups function in the Scanpy package (for spatial domain recovery): "differentially expressed genes are selected using the rank_genes_groups function in the Scanpy package (for spatial domain recovery)"
- [methods] we apply the decoder to reconstruct the spatial gene expression matrix, which will be combined with the original ST data for model training: "we apply the decoder to reconstruct the spatial gene expression matrix, which will be combined with the original ST data for model training"
- [readme] CeLEry takes spatial transcriptomic data as input for the training data and the scRNA-seq as testing data set.: "CeLEry takes spatial transcriptomic data as input for the training data and the scRNA-seq as testing data set."
- [discussion] The number of replicates to be generated is a hyperparameter prespecified by users: "The number of replicates to be generated is a hyperparameter prespecified by users"
