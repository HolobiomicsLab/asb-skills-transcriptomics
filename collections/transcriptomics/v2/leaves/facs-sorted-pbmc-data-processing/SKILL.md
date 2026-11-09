---
name: facs-sorted-pbmc-data-processing
description: Use when benchmarking cell-label assignment methods (e.g., NeuCA vs scmap,
  CHETAH, Seurat, SC3) on labeled 10X PBMC scRNA-seq data with Monte Carlo training/testing
  splits.
when_to_use_negative:
- Test data are expected to contain unknown or novel cell types not present in the
  training/reference data — NeuCA assumes all cell types in the new sample already
  exist in the reference data
- Training data do not exist for the tissue of interest — unsupervised clustering,
  not supervised label assignment, is required
- Input is raw/unlabeled scRNA-seq with no known cell labels to train a supervised
  classifier on
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: Supervised neural-network classifier (small/medium/big sizes) with tau=0.95
    correlation-based strategy selection between feed-forward and hierarchical networks
  repo: https://bioconductor.org/packages/NeuCA
- name: scmap
  role: Correlation-based supervised baseline (scmap-cluster, scmap-cell) run on identical
    splits
- name: CHETAH
  role: Hierarchical supervised baseline method on identical splits
- name: Seurat
  role: Unsupervised clustering baseline method on identical splits
- name: SC3
  role: Unsupervised clustering baseline method on identical splits
- name: SingleCellExperiment
  role: Bioconductor container for preparing train/test normcounts and cell labels
    as NeuCA input
- name: keras
  role: R API underlying NeuCA's neural-network implementation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:7c7b5004e82815ad614bf4f50410382faa01df1563b1e730190fd26fd3d9ea6c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - NeuCA
  - R
  - SC3
  - Seurat
  - SingleCellExperiment
  - keras
  - scmap
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# facs-sorted-pbmc-data-processing

## Summary

Method for building and benchmarking supervised/unsupervised scRNA-seq cell-annotation experiments on the FACS-sorted 10X Genomics PBMC data (Zheng_8ct_PBMC), using Monte Carlo train/test splits evaluated by accurately assigned rate and ARI.

## When to use

Use when benchmarking cell-label assignment methods (e.g., NeuCA vs scmap, CHETAH, Seurat, SC3) on labeled 10X PBMC scRNA-seq data with Monte Carlo training/testing splits.

## When NOT to use

- Test data are expected to contain unknown or novel cell types not present in the training/reference data — NeuCA assumes all cell types in the new sample already exist in the reference data
- Training data do not exist for the tissue of interest — unsupervised clustering, not supervised label assignment, is required
- Input is raw/unlabeled scRNA-seq with no known cell labels to train a supervised classifier on

## Inputs

- FACS-sorted 10X Genomics PBMC scRNA-seq count matrix (Zheng_8ct_PBMC, from DuoClustering2018)
- Published/cell-type label vector for each cell (e.g., CD4 T helper, B, CD14 monocytes, NK, memory T, naive cytotoxic T, naive T, regulatory T)
- SingleCellExperiment objects with normcounts assay and colData cell labels

## Outputs

- benchmark_results.csv of accurately assigned rate and ARI per method per split
- predicted.label vector (one predicted cell type per test cell)
- Confusion matrix / Sankey visualization comparing predicted vs true test labels

## How to apply

Download the FACS-sorted 10X Genomics PBMC data (9139 CD4 T helper, 7959 B, 718 CD14 monocytes, 7601 NK, 9338 memory T, 10993 naive cytotoxic T, 7700 naive T, 7861 regulatory T cells), e.g. Zheng_8ct_PBMC from the R/Bioconductor package DuoClustering2018, then normalize the expression matrix and subset training and testing matrices to overlapped genes. Generate Monte Carlo splits by drawing p = 10%, 20%, 50%, or 80% of cells as training data and n = 800, 1600, or 4000 testing cells from the remainder; add a T-cell-only setting with the five correlated T cell types and a merged setting collapsing all T cells into one type. Prepare train and test data as SingleCellExperiment objects with normcounts assays and feature symbols in rowData, removing duplicated gene names. Run NeuCA with model.size in {"small", "medium", "big"} (3/4/5 hidden layers, 64 to 256 nodes) using tau = 0.95 as the correlation threshold that selects between feed-forward and cell lineage-based hierarchical neural networks, with ReLU activation and RMSprop optimization. Run scmap-cluster, scmap-cell, CHETAH, Seurat, and SC3 on the identical splits, compute the accurately assigned rate and Adjusted Rand Index (ARI) per split, and aggregate to check that NeuCA-med exceeds scmap-cluster by about 10 percentage points and NeuCA-small by at least 5 percentage points.

## Related tools

- **NeuCA** (Supervised neural-network classifier (small/medium/big sizes) with tau=0.95 correlation-based strategy selection between feed-forward and hierarchical networks) — https://bioconductor.org/packages/NeuCA
- **scmap** (Correlation-based supervised baseline (scmap-cluster, scmap-cell) run on identical splits)
- **CHETAH** (Hierarchical supervised baseline method on identical splits)
- **Seurat** (Unsupervised clustering baseline method on identical splits)
- **SC3** (Unsupervised clustering baseline method on identical splits)
- **SingleCellExperiment** (Bioconductor container for preparing train/test normcounts and cell labels as NeuCA input)
- **keras** (R API underlying NeuCA's neural-network implementation)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- Accurately assigned rate and ARI computed per method on every Monte Carlo split; all three NeuCA versions should rank at the top on both metrics
- NeuCA-med exceeds scmap-cluster (best existing method) by ~10 percentage points and NeuCA-small by at least 5 percentage points in accurately assigned rate
- predicted.label length equals the number of test cells and its table() counts sum to the test-set size
- Training and testing matrices are restricted to overlapped genes with no duplicated feature symbols in rowData
- Confusion matrix off-diagonal entries near zero when cell types are well separated (e.g., Baron/Seg example: 328/331 alpha correct)

## Limitations

- NeuCA is designed for well-studied situations where all cell types in the new sample already exist in the reference/training data
- Existing methods, including CHETAH, label a considerable number of cells as unassigned even when unknown cell types are not expected; deciding whether unknown/novel cell types exist is not addressed and expert consultation is recommended
- No version numbers are given for keras, TensorFlow, or R in the software description
- Cross-condition prediction was shown only for the ASD data application and may not hold in other scenarios

## Evidence

- [methods] Real PBMC data from the 10X Genomics website were downloaded and processed to generate our simulation datasets: "Real PBMC data from the 10X Genomics website (https://support.10xgenomics.com/)31 were downloaded and processed to generate our simulation datasets"
- [methods] The Zheng_8ct_PBMC data were downloaded from the R/Bioconductor package DuoClustering2018: "The Zheng_8ct_PBMC data were downloaded from the R/Bioconductor package DuoClustering2018"
- [methods] normalize expression matrix and subset training and testing data by overlapped genes: [evidence span withheld — anchor status: not_found]
- [readme] model.size argument choices reflect network complexity: [evidence span withheld — anchor status: external_doc]
- [intro] NeuCA is available through a freely available R/Bioconductor package: "NeuCA is available through a freely available R/Bioconductor package at https://bioconductor.org/packages/NeuCA/"
