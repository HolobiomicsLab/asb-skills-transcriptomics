---
name: cell-quality-filtering-umi-counts
description: Use when loading high-resolution spatial transcriptomics data (e.g.,
  MERSCOPE, MERFISH, or 10X Xenium cells) or scRNA-seq test data prior to CeLEry training/prediction,
  and cells with sparse UMIs or few detected genes would otherwise add noise to expression-to-location
  learning.
when_to_use_negative:
- Data are already spot-level Visium counts with adequate depth per spot — the paper's
  filtering is described for MERSCOPE, MERFISH, and 10X Xenium cell-level data
- The downstream method is not CeLEry-style supervised learning, and aggressive cell
  removal would discard biologically relevant rare or low-UML cells from a descriptive
  analysis
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Scanpy
  role: Loads Xenium/MERSCOPE/MERFISH data (AnnData) and provides cell-level QC and
    filtering utilities prior to normalization
  repo: https://github.com/QihuangZhang/CeLEry
- name: CeLEry
  role: Downstream consumer of the filtered counts; trains its deep neural network
    on quality-controlled expression profiles
  repo: https://github.com/QihuangZhang/CeLEry
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:c0c07d43fd6f2ad13634d6f35d27dc8ed07e1d61cadb0474e096f3b81096bff3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QihuangZhang/CeLEry
  tools:
  - CeLEry
  - PyTorch
  - Scanpy
  - Tangram
  - novoSpaRc
  - spaGCN
  - spaOTsc
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Cell quality filtering by UMI counts and expressed-gene numbers

## Summary

Quality-control step that eliminates single-cell or spot-level observations with low UMI counts or a small number of expressed genes before downstream normalization and model training. In the CeLEry pipeline this filtering is applied to single-cell-resolution ST platforms (MERSCOPE, MERFISH, 10X Xenium) so that the supervised location-prediction network is not trained on low-information cells.

## When to use

Use when loading high-resolution spatial transcriptomics data (e.g., MERSCOPE, MERFISH, or 10X Xenium cells) or scRNA-seq test data prior to CeLEry training/prediction, and cells with sparse UMIs or few detected genes would otherwise add noise to expression-to-location learning.

## When NOT to use

- Data are already spot-level Visium counts with adequate depth per spot — the paper's filtering is described for MERSCOPE, MERFISH, and 10X Xenium cell-level data
- The downstream method is not CeLEry-style supervised learning, and aggressive cell removal would discard biologically relevant rare or low-UML cells from a descriptive analysis

## Inputs

- Single-cell-resolution spatial transcriptomics count matrix (e.g., 10X Xenium cell-by-gene counts from breast cancer Replicate 1 and Replicate 2)
- MERSCOPE or MERFISH cell-level count data
- scRNA-seq query AnnData to be mapped by CeLEry

## Outputs

- Filtered AnnData/cell-by-gene matrix containing only cells passing the UMI-count and expressed-gene criteria
- Filtered dataset ready for two-step normalization (spot-level log transformation, gene-level z-scoring) and spatially variable gene selection

## How to apply

Load the spatial data (e.g., Xenium breast cancer Replicate 1 and Replicate 2) into an AnnData object with Scanpy, then remove cells that have low total UMI counts or only a small number of expressed genes; the CeLEry paper applies this step to MERSCOPE, MERFISH, and 10X Xenium datasets. After filtering, apply the two-step normalization — cell/spot-level UMI normalization followed by gene-level z-scoring — before selecting spatially variable genes with spaGCN's rank_genes_groups. The rationale is that dissociated or poorly segmented cells with sparse profiles carry little gene-expression signal, so retaining them would degrade the gene-expression-to-spatial-location relationship learned by the deep neural network (3 hidden layers with 50/10/5 nodes, ReLU activations, Sigmoid output). The same filtering logic is applied to the scRNA-seq test data so that training and prediction inputs share comparable quality. Filtering thresholds are not fixed in the paper, so sanity-check the retained fraction of cells and expression depth distributions before and after.

## Related tools

- **Scanpy** (Loads Xenium/MERSCOPE/MERFISH data (AnnData) and provides cell-level QC and filtering utilities prior to normalization) — https://github.com/QihuangZhang/CeLEry
- **CeLEry** (Downstream consumer of the filtered counts; trains its deep neural network on quality-controlled expression profiles) — https://github.com/QihuangZhang/CeLEry

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('xenium_breast_cancer_rep2.h5ad'); sc.pp.filter_cells(adata, min_genes=10); sc.pp.filter_cells(adata, min_counts=50)
```

## Evaluation signals

- No retained cell should have an anomalously low UMI total or expressed-gene count relative to the platform's distribution (inspect QC histograms before/after).
- After filtering, the two-step normalization (cell/spot-level log transformation of UMI counts, then gene-level z-scoring) should run without NaNs or zero-variance genes.
- spaGCN's rank_genes_groups should return a non-empty set of spatially variable genes on the filtered matrix.
- Downstream CeLEry pairwise-distance correlations should reach the reported ranges (0.74 single-cell, 0.46 artificial Visium, 0.51 TESLA schemes) on Xenium breast cancer Replicate 2 predictions, indicating the filter did not remove informative cells.

## Limitations

- The paper does not state explicit numeric thresholds for 'low UMI counts' or 'small number of expressed genes', so cutoffs must be chosen by the user and validated empirically.
- Overly aggressive filtering could remove cells that contribute to the spatial expression gradients (e.g., ERBB2/ESR1/GATA3 patterns) being recovered.

## Evidence

- [methods] In the study of MERSCOPE, MERFISH, and 10X Xenium data, cells with low UMI counts or a small number of expressed genes were eliminated: "In the study of MERSCOPE, MERFISH, and 10X Xenium data, cells with low UMI counts or a small number of expressed genes were eliminated"
- [methods] The gene expression normalization involves two steps. In the first step, cell/spot-level normalization is performed: "The gene expression normalization involves two steps. In the first step, cell/spot-level normalization is performed"
- [readme] Python (>3.8) support packages: torch>=1.8, pandas>=1.4, numpy>=1.20, scipy, tqdm, scanpy>=1.5, anndata, sklearn, scikit-image: "Python (>3.8) support packages: torch>=1.8, pandas>=1.4, numpy>=1.20, scipy, tqdm, scanpy>=1.5, anndata, sklearn, scikit-image"
