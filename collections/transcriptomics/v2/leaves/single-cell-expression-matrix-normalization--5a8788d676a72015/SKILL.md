---
name: single-cell-expression-matrix-normalization
description: Use when raw scRNA-seq count matrices (e.g. Zheng_8ct_PBMC from DuoClustering2018,
  or GEO FACS-sorted datasets like GSE130430/GSE103544) must be normalized and gene-aligned
  before feeding a supervised annotation tool such as NeuCA.
when_to_use_negative:
- The data are already SingleCellExperiment objects with normalized normcounts and
  matched gene spaces — proceed directly to NeuCA training/prediction instead
- The goal is de novo cell discovery on unlabelled data with no reference training
  set; supervised preprocessing for label assignment does not apply
- A substantial fraction of cells may be novel/malignant cell types absent from the
  reference — screen them first with tools like inferCNV or copyKAT rather than normalizing
  everything for exhaustive assignment
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: Downstream annotation method whose input requirement (SingleCellExperiment
    with normcounts and overlapped genes) drives the normalization and gene-subsetting
    steps
  repo: https://bioconductor.org/packages/NeuCA/
- name: R
  role: Environment for loading DuoClustering2018/GEO datasets, normalizing matrices,
    and building SingleCellExperiment objects
  repo: https://bioconductor.org/packages/NeuCA
- name: SingleCellExperiment
  role: Bioconductor data container used to organize the normalized matrix, feature
    symbols, and cell labels for NeuCA input
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:4e6a66e35843c95eedfa303b73f9cbbcbb947c91b0e9c101530603715ecbd310
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  - SingleCellExperiment
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-cell-expression-matrix-normalization

## Summary

Preprocessing skill for NeuCA-based scRNA-seq cell annotation: normalize the expression matrix and subset training/testing matrices to overlapped, non-duplicated genes, then package them as Bioconductor SingleCellExperiment objects with a normcounts assay.

## When to use

Use when raw scRNA-seq count matrices (e.g. Zheng_8ct_PBMC from DuoClustering2018, or GEO FACS-sorted datasets like GSE130430/GSE103544) must be normalized and gene-aligned before feeding a supervised annotation tool such as NeuCA.

## When NOT to use

- The data are already SingleCellExperiment objects with normalized normcounts and matched gene spaces — proceed directly to NeuCA training/prediction instead
- The goal is de novo cell discovery on unlabelled data with no reference training set; supervised preprocessing for label assignment does not apply
- A substantial fraction of cells may be novel/malignant cell types absent from the reference — screen them first with tools like inferCNV or copyKAT rather than normalizing everything for exhaustive assignment

## Inputs

- Raw or count-level scRNA-seq expression matrix for the labelled training dataset (e.g. Baron_scRNA, Zheng_8ct_PBMC)
- Raw or count-level scRNA-seq expression matrix for the testing dataset (e.g. Seg_scRNA, FACS-sorted NK or monocyte data from GEO)
- Known cell-type labels for the training cells

## Outputs

- Normalized training expression matrix with overlapped genes, as a SingleCellExperiment object with a normcounts assay and colData cell labels
- Normalized testing expression matrix as a SingleCellExperiment object with duplicated gene features removed

## How to apply

After loading the labelled training data and the testing data, apply preprocessing steps such as normalizing the expression matrix and taking subsets of the training (X0) and testing matrices restricted to their overlapped genes, since a supervised classifier can only score features shared between studies. Represent each dataset as a SingleCellExperiment object storing the normalized matrix in a 'normcounts' assay with cell labels in colData, and set rowData$feature_symbol to gene names. Remove features with duplicated names before modeling so each gene maps to a unique network input. Rationale: NeuCA trains a keras-based neural network (feed-forward or hierarchical depending on the Pearson correlation of averaged cell-type profiles with threshold tau = 0.95) on this normalized, gene-matched input, so inconsistent gene spaces or duplicated features would break classifier training and inter-study prediction.

## Related tools

- **NeuCA** (Downstream annotation method whose input requirement (SingleCellExperiment with normcounts and overlapped genes) drives the normalization and gene-subsetting steps) — https://bioconductor.org/packages/NeuCA/
- **R** (Environment for loading DuoClustering2018/GEO datasets, normalizing matrices, and building SingleCellExperiment objects) — https://bioconductor.org/packages/NeuCA
- **SingleCellExperiment** (Bioconductor data container used to organize the normalized matrix, feature symbols, and cell labels for NeuCA input)

## Examples

```
Baron_sce <- SingleCellExperiment(assays = list(normcounts = as.matrix(Baron_counts)), colData = Baron_anno); rowData(Baron_sce)$feature_symbol <- rownames(Baron_sce); Baron_sce <- Baron_sce[!duplicated(rownames(Baron_sce)), ]
```

## Evaluation signals

- Both training and testing SingleCellExperiment objects contain a 'normcounts' assay holding a numeric matrix
- Training and testing gene sets are restricted to their overlapped genes, with identical feature_symbol rows across the two matrices
- rownames of each object contain no duplicated feature symbols after the deduplication step
- colData of the training object carries the known cell-type labels aligned to cell barcodes
- Downstream NeuCA training proceeds without gene-dimension errors and predicted labels fall within the training label set

## Limitations

- NeuCA (and hence this preprocessing) is restricted to settings where all cell types in the new sample already exist in the reference/training data
- Only genes overlapping between training and testing matrices are retained, so genes unique to one study are discarded
- No version numbers are given in the paper for keras, TensorFlow, or R, so reproducibility of the normalization environment depends on the user's session (README shows R 4.0.3, keras 2.4.0)

## Evidence

- [methods] Preprocessing: normalize expression matrix and subset training and testing data by overlapped genes: [evidence span withheld — anchor status: not_found]
- [readme] README: prepare training data as a SingleCellExperiment with normcounts assay: "assays = list(normcounts = as.matrix(Baron_counts))"
- [readme] README: use gene names as feature symbols and remove duplicated features: "Baron_sce <- Baron_sce[!duplicated(rownames(Baron_sce)), ]"
- [intro] Intro: supervised classifier is trained on data with known labels and applied to new data: "It uses existing scRNA-seq data with known labels to train a neural network-based classifier and then predict cell labels in new data of interest"
