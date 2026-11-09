---
name: spatially-variable-gene-selection
description: 'Use when preprocessing 10x Visium, MERFISH/MERSCOPE, or 10x Xenium spatial
  data as training input for CeLEry 2D location recovery: after two-step normalization,
  pick the informative gene panel (spaGCN rank_genes_groups for 2D location recovery)
  before training the deep neural network.'
when_to_use_negative:
- The downstream task is spatial domain (categorical) recovery rather than 2D location
  recovery — use Scanpy's rank_genes_groups for differentially expressed gene selection
  instead of spaGCN's
- The input is scRNA-seq data with no spatial coordinates available for training,
  since spatially variable gene identification requires ST data with known spot/cell
  locations
- Gene selection has already been performed on the shared training/testing gene panel
  and the feature matrices are already dimension-matched
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0085
tools:
- name: spaGCN
  role: Provides the rank_genes_groups function used to select spatially variable
    genes for 2D location recovery
- name: Scanpy
  role: Provides rank_genes_groups used to select differentially expressed genes when
    the target is spatial domain recovery; also used for loading/filtering ST and
    Xenium data
- name: CeLEry
  role: 'Downstream consumer: the selected spatially variable genes define the input
    feature space of the location-prediction deep neural network'
  repo: https://github.com/QihuangZhang/CeLEry
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:14d4112c2b82b2793f69d2e7702ff786c2db3dc0406d155fe424a8a51ef94c46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QihuangZhang/CeLEry
  tools:
  - 10x Visium
  - CeLEry
  - PyTorch
  - Scanpy
  - Tangram
  - novoSpaRc
  - spaGCN
  - spaOTsc
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spatially-variable-gene-selection

## Summary

Selects spatially variable genes from spatial transcriptomics data using the rank_genes_groups function of the spaGCN package (or differentially expressed genes via Scanpy's rank_genes_groups) so that CeLEry's location-prediction network is trained on informative features rather than the full transcriptome.

## When to use

Use when preprocessing 10x Visium, MERFISH/MERSCOPE, or 10x Xenium spatial data as training input for CeLEry 2D location recovery: after two-step normalization, pick the informative gene panel (spaGCN rank_genes_groups for 2D location recovery) before training the deep neural network.

## When NOT to use

- The downstream task is spatial domain (categorical) recovery rather than 2D location recovery — use Scanpy's rank_genes_groups for differentially expressed gene selection instead of spaGCN's
- The input is scRNA-seq data with no spatial coordinates available for training, since spatially variable gene identification requires ST data with known spot/cell locations
- Gene selection has already been performed on the shared training/testing gene panel and the feature matrices are already dimension-matched

## Inputs

- Preprocessed spatial transcriptomics data (log-transformed UMI counts, per-gene z-scored) as AnnData, e.g. 10x Visium mouse posterior brain, MERFISH Mouse3_sagittal slices, or 10x Xenium breast cancer Replicate 1
- Low-quality cells/spots already filtered (cells with low UMI counts or a small number of expressed genes eliminated)

## Outputs

- A ranked list/set of spatially variable genes (e.g., 358 genes for the Visium mouse posterior brain task) used as the feature matrix for CeLEry training and testing
- Gene-filtered expression matrices for training spots and held-out test spots/cells on a shared gene panel

## How to apply

After cell/spot-level log transformation of UMI counts followed by per-gene standardization (z-scoring), run spaGCN's rank_genes_groups to identify spatially variable genes — e.g., the 358 spatially variable genes selected for the mouse posterior brain Visium 2D location recovery task. Use these genes as the feature space for training the CeLEry supervised network (3 hidden layers of 50/10/5 nodes, ReLU, sigmoid output scaled to [0,1]) and apply the identical gene panel to test/held-out spots or scRNA-seq cells so training and prediction share the same feature dimensions. If the downstream goal is spatial domain (categorical) recovery rather than 2D coordinates, instead use Scanpy's rank_genes_groups to select differentially expressed genes. When training and testing come from different tissue slices (e.g., MERFISH Mouse3_sagittal Slice 5 vs Slice 7), keep the selection identical across slices to make pairwise-distance Pearson correlations comparable across methods.

## Related tools

- **spaGCN** (Provides the rank_genes_groups function used to select spatially variable genes for 2D location recovery)
- **Scanpy** (Provides rank_genes_groups used to select differentially expressed genes when the target is spatial domain recovery; also used for loading/filtering ST and Xenium data)
- **CeLEry** (Downstream consumer: the selected spatially variable genes define the input feature space of the location-prediction deep neural network) — https://github.com/QihuangZhang/CeLEry

## Examples

```
sc.pp.highly_variable_genes(adata, n_top_genes=358)  # then: spagcn_select_genes = adata.var_names[adata.var['highly_variable']]
```

## Evaluation signals

- The selected gene panel (e.g., 358 Visium genes) is applied identically to training spots, held-out super-pixels, and any scRNA-seq test cells so feature dimensions match
- Downstream CeLEry pairwise-distance Pearson correlations (e.g., ~0.89 on MERFISH single-slice; 0.74/0.46/0.51 on Xenium) exceed Tangram, spaOTsc, and novoSpaRc on the same gene panel
- Reconstructed 2D gene-expression maps (e.g., ERBB2, ESR1, GATA3 in Xenium) show spatial gradients consistent with ground truth, indicating the retained genes carry spatial signal
- Changing holdoff fraction (10/30/50%) or training scenario using the same selected genes does not change the gene feature dimensionality or break the trained model

## Limitations

- The article does not report how many spatially variable gradient genes exist in tissues in general; identifying genes or sets of genes with gradient expression patterns 'requires further investigation'
- Gene selection is tied to the specific task: spaGCN rank_genes_groups is specified for 2D location recovery while Scanpy rank_genes_groups is used for spatial domain recovery — mixing them up degrades the downstream model
- CeLEry is suboptimal on single-cell-resolution platforms with small gene panels, which constrains how informative any selected gene set can be on such data

## Evidence

- [methods] Spatially variable gene selection with spaGCN for 2D location recovery: "Spatially variable genes are selected using the rank_genes_groups function in the spaGCN package (for 2D location recovery)"
- [methods] Differentially expressed gene selection with Scanpy for spatial domain recovery: "differentially expressed genes are selected using the rank_genes_groups function in the Scanpy package (for spatial domain recovery)"
- [methods] Filtering low-quality cells before gene selection: "cells with low UMI counts or a small number of expressed genes were eliminated"
- [methods] Two-step normalization precedes gene selection: "The gene expression normalization involves two steps. In the first step, cell/spot-level normalization is performed"
- [other] Concrete gene count for the Visium mouse posterior brain task: [evidence span withheld — anchor status: not_found]
