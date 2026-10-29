---
name: spatial-transcriptomics-preprocessing
description: Use when raw ST count matrices (10x Visium, MERSCOPE, MERFISH, Xenium)
  must be normalized and filtered before training CeLEry for 2D location or spatial
  domain recovery, or before VAE-based data augmentation.
when_to_use_negative:
- Input is already a feature-normalized or preprocessed expression matrix — re-applying
  gene-level z-scoring would double-standardize
- The downstream goal does not involve spatial location or domain prediction (e.g.,
  generic clustering) — gene selection should follow task-specific criteria instead
  of spaGCN/Scanpy rank_genes_groups
- Data are not spatially resolved (bulk RNA-seq or dissociated scRNA-seq without ST
  training reference)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Loads and preprocesses ST count data; rank_genes_groups for differentially
    expressed gene selection (spatial domain recovery)
- name: spaGCN
  role: rank_genes_groups function for spatially variable gene selection (2D location
    recovery)
- name: CeLEry
  role: Downstream package consuming the preprocessed ST data (pip install CeLEryPy);
    preprocessing utilities for various ST formats
  repo: github.com/QihuangZhang/CeLEry
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  - task_006
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:14f63af26c9efa877b471dbf60599fb84053f51ccaf82feb40675691f978b689
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BASS
  - BayesSpace
  - CeLEry
  - PyTorch
  - R
  - SC.MEB
  - Scanpy
  - SpaGCN
  - Tangram
  - devtools
  - iSC.MEB
  - mclust
  - novoSpaRc
  - scikit-learn
  - spaGCN
  - spaOTsc
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# spatial-transcriptomics-preprocessing

## Summary

Two-step normalization and gene/cell filtering of spatial transcriptomics (ST) data (Visium, MERSCOPE, MERFISH, 10X Xenium) prior to CeLEry training: eliminate low-UMI cells, apply spot/cell-level log transformation of UMI counts followed by gene-level z-scoring, and select informative genes per task.

## When to use

Use when raw ST count matrices (10x Visium, MERSCOPE, MERFISH, Xenium) must be normalized and filtered before training CeLEry for 2D location or spatial domain recovery, or before VAE-based data augmentation.

## When NOT to use

- Input is already a feature-normalized or preprocessed expression matrix — re-applying gene-level z-scoring would double-standardize
- The downstream goal does not involve spatial location or domain prediction (e.g., generic clustering) — gene selection should follow task-specific criteria instead of spaGCN/Scanpy rank_genes_groups
- Data are not spatially resolved (bulk RNA-seq or dissociated scRNA-seq without ST training reference)

## Inputs

- Raw spatial transcriptomics count matrices (10x Visium, MERSCOPE, MERFISH, 10X Xenium breast cancer, LIBD DLPFC sections) in Scanpy/AnnData-readable format
- Spot/cell spatial coordinate tables for coordinate rescaling
- scRNA-seq test expression matrix aligned to the selected gene panel

## Outputs

- Normalized ST expression matrix (log-transformed, gene-level z-scored) filtered to informative genes
- Gene selection lists: spatially variable genes (spaGCN rank_genes_groups) or differentially expressed genes (Scanpy rank_genes_groups)
- Section-merged training matrix subset to common genes across sections (multi-section scenarios)
- Spatial coordinates rescaled to [0,1]

## How to apply

First eliminate cells/spots with low UMI counts or a small number of expressed genes, which the paper applies uniformly to MERSCOPE, MERFISH, and 10X Xenium data. Then normalize in two steps: cell/spot-level log transformation of UMI counts, followed by per-gene standardization (subtract the gene mean, divide by the gene standard deviation across spots/cells). Next select informative genes with rank_genes_groups in spaGCN when the downstream goal is 2D location recovery, or rank_genes_groups in Scanpy when the goal is spatial domain recovery. For multi-section training scenarios (e.g., LIBD DLPFC sections or MERFISH Slices 4+5), normalize each section separately, then subset to the common genes shared by all sections before merging. Finally, rescale the 2D spatial coordinates to the [0,1] range so the CeLEry Sigmoid output layer can be trained with squared-error loss.

## Related tools

- **Scanpy** (Loads and preprocesses ST count data; rank_genes_groups for differentially expressed gene selection (spatial domain recovery))
- **spaGCN** (rank_genes_groups function for spatially variable gene selection (2D location recovery))
- **CeLEry** (Downstream package consuming the preprocessed ST data (pip install CeLEryPy); preprocessing utilities for various ST formats) — github.com/QihuangZhang/CeLEry

## Examples

```
pip install CeLEryPy
```

## Evaluation signals

- Filtered cells/spots retain sufficient UMI counts and expressed genes; low-UMI or low-gene cells eliminated as in MERSCOPE/MERFISH/Xenium pipelines
- Expression matrix is log-transformed per spot/cell and each gene has mean 0 and SD 1 across spots after z-scoring
- Gene selection matches the task: spatially variable genes from spaGCN rank_genes_groups for 2D location recovery, DE genes from Scanpy rank_genes_groups for spatial domain recovery
- Multi-section training sets share exactly the common gene set across all sections after separate per-section normalization
- Training coordinates fall within [0,1] after rescaling, matching the CeLEry Sigmoid output range

## Limitations

- Software versions are not reported in the article; the README specifies Python >3.8 with torch>=1.8, scanpy>=1.5, pandas>=1.4, numpy>=1.20
- Filtering thresholds for 'low UMI counts or a small number of expressed genes' are not given numerically in the extracted text
- CeLEry is suboptimal on single-cell-resolution platforms with small gene panels, limiting preprocessing benefit on such inputs

## Evidence

- [methods] Two-step normalization with gene filtering: "The gene expression normalization involves two steps. In the first step, cell/spot-level normalization is performed"
- [methods] Spatially variable gene selection for 2D location recovery: "Spatially variable genes are selected using the rank_genes_groups function in the spaGCN package (for 2D location recovery)"
- [methods] Cell quality filtering applied to MERSCOPE, MERFISH, and 10X Xenium data: "cells with low UMI counts or a small number of expressed genes were eliminated"
- [readme] CeLEry input data roles for training vs testing: "CeLEry takes spatial transcriptomic data as input for the training data and the scRNA-seq as testing data set."
- [readme] Preprocessing-capable package with supported dependency versions: "Python (>3.8) support packages: torch>=1.8, pandas>=1.4, numpy>=1.20, scipy, tqdm, scanpy>=1.5, anndata, sklearn, scikit-image"
