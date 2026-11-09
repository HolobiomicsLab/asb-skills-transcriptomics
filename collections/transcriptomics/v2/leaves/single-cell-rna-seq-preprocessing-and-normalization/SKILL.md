---
name: single-cell-rna-seq-preprocessing-and-normalization
description: Use when a dissociated scRNA-seq/snRNA-seq matrix (e.g. AD middle frontal
  neocortex snRNA-seq, ~122,606 nuclei, loaded via scanpy/anndata) must be normalized
  before a supervised spatial-domain or 2D-location prediction model.
when_to_use_negative:
- The data are already spatial transcriptomics spots (Visium/MERSCOPE/MERFISH/Xenium)
  serving as the training side of the CeLEry pipeline — this normalization targets
  the dissociated scRNA-seq test data
- The downstream method requires raw integer counts or library-size-only normalization
  (e.g. differential expression testing), since gene-level z-scoring destroys the
  count scale
- The gene space cannot be matched to the pretrained model (few or no overlapping
  non-DE genes), so gene-level standardization on the shared set is not meaningful
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: CeLEry
  role: Downstream deep neural network that consumes the normalized query scRNA-seq
    matrix; its training-data gene set and normalization scheme define the required
    preprocessing
  repo: https://github.com/QihuangZhang/CeLEry
- name: Scanpy
  role: Loading/filtering of the snRNA-seq AnnData, cell-level logarithm transformation
    of UMI counts, gene-level standardization, and rank_genes_groups-based DE gene
    selection
- name: LIBD
  role: Source of the pretrained spatial domain model (DLPFC sections 151673-151676)
    whose shared non-DE gene set constrains the normalization
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:a3e6d2fdd043397044f9551bf0ca1ff2a2f9921a98e1d1419921058a6c8ac9e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QihuangZhang/CeLEry
  tools:
  - CeLEry
  - LIBD
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-cell-rna-seq-preprocessing-and-normalization

## Summary

Two-step scRNA-seq preprocessing for CeLEry location-recovery inputs: eliminate low-quality nuclei/cells, apply cell-level logarithm transformation of UMI counts, then gene-level standardization (z-score across cells), restricted to the gene set shared with the training model.

## When to use

Use when a dissociated scRNA-seq/snRNA-seq matrix (e.g. AD middle frontal neocortex snRNA-seq, ~122,606 nuclei, loaded via scanpy/anndata) must be normalized before a supervised spatial-domain or 2D-location prediction model.

## When NOT to use

- The data are already spatial transcriptomics spots (Visium/MERSCOPE/MERFISH/Xenium) serving as the training side of the CeLEry pipeline — this normalization targets the dissociated scRNA-seq test data
- The downstream method requires raw integer counts or library-size-only normalization (e.g. differential expression testing), since gene-level z-scoring destroys the count scale
- The gene space cannot be matched to the pretrained model (few or no overlapping non-DE genes), so gene-level standardization on the shared set is not meaningful

## Inputs

- Raw UMI count matrix of dissociated scRNA-seq/snRNA-seq cells (anndata/AnnData object, e.g. AD postmortem middle frontal neocortex snRNA-seq of ~122,606 nuclei)
- Gene list shared between the query scRNA-seq and the CeLEry training model (non-DE gene set)
- Differentially expressed genes selected via rank_genes_groups in Scanpy (to be excluded from the shared set)

## Outputs

- Filtered AnnData object containing only cells passing the low-UMI/low-gene-count quality cutoff
- Cell-level log-transformed, gene-level z-scored expression matrix (per-gene mean subtracted, per-gene standard deviation divided) restricted to the shared non-DE gene set
- Normalized query matrix ready as test input to a pretrained CeLEry spatial-domain prediction network

## How to apply

Load the raw count matrix with scanpy/anndata and eliminate cells with low UMI counts or a small number of expressed genes. Perform cell-level normalization by logarithm transformation of UMI counts, then gene-level standardization by subtracting the mean and dividing by the standard deviation of each gene across cells — the article's two-step normalization. Restrict the matrix to the non-differentially-expressed gene set shared with the training model (DE genes are pre-selected with rank_genes_groups in Scanpy for spatial domain recovery); holding back DE genes prevents the model from learning disease-state signatures instead of expression–location relationships. The result is a z-scored expression matrix whose gene space matches the pretrained network (e.g. the LIBD DLPFC spatial-domain model trained on sections 151673-151676).

## Related tools

- **CeLEry** (Downstream deep neural network that consumes the normalized query scRNA-seq matrix; its training-data gene set and normalization scheme define the required preprocessing) — https://github.com/QihuangZhang/CeLEry
- **Scanpy** (Loading/filtering of the snRNA-seq AnnData, cell-level logarithm transformation of UMI counts, gene-level standardization, and rank_genes_groups-based DE gene selection)
- **LIBD** (Source of the pretrained spatial domain model (DLPFC sections 151673-151676) whose shared non-DE gene set constrains the normalization)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('AD_middle_frontal_neocortex.h5ad'); sc.pp.filter_cells(adata, min_counts=500); sc.pp.log1p(adata); sc.pp.scale(adata, zero_center=True)
```

## Evaluation signals

- All low-quality cells (low UMI counts or a small number of expressed genes) are eliminated before transformation
- Each retained gene column in the processed matrix has mean ~0 and standard deviation ~1 across cells (gene-level standardization verified)
- The gene set of the processed query equals the non-differentially-expressed gene set shared with the pretrained training model
- Downstream CeLEry predictions for every nucleus fall within the valid spatial domain set {L1, L2, L3, L4, L5, L6, WM}
- Reproduced layer-composition patterns qualitatively match expectations: L1 dominated by astrocytes, oligodendrocytes enriched in L6/WM, reduced neuron fractions in pathology groups (Fig. 4 of the article)

## Limitations

- Optimal preprocessing for the optional variational-autoencoder data augmentation depends on the number of augmented replicates, which is a user-specified hyperparameter requiring cross-validation rather than a fixed setting
- Preprocessing for single-cell-resolution platforms with small gene panels (e.g. targeted MERFISH/Xenium panels) may be suboptimal for CeLEry, which performs better with transcriptome-wide ST training data
- Genes showing spatially graded expression patterns that could aid location recovery are not yet systematically identified, so the shared non-DE gene restriction may discard informative gradient genes

## Evidence

- [methods] Two-step normalization of scRNA-seq query data: "The gene expression normalization involves two steps. In the first step, cell/spot-level normalization is performed"
- [methods] Low-quality cell filtering before normalization: "cells with low UMI counts or a small number of expressed genes were eliminated"
- [methods] DE gene selection with Scanpy for spatial domain recovery: "differentially expressed genes are selected using the rank_genes_groups function in the Scanpy package (for spatial domain recovery)"
- [readme] Scanpy and anndata are required dependencies of the CeLEry package: "Python (>3.8) support packages: torch>=1.8, pandas>=1.4, numpy>=1.20, scipy, tqdm, scanpy>=1.5, anndata, sklearn, scikit-image"
- [abstract] Optional VAE data augmentation on the training ST data: "CeLEry has an optional data augmentation procedure via a variational autoencoder, which improves the method's robustness and allows it to overcome noise in scRNA-seq data."
