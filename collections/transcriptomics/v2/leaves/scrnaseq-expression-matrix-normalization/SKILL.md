---
name: scrnaseq-expression-matrix-normalization
description: Use when raw scRNA-seq count matrices for a labeled training set and
  a testing set (e.g., ASD snRNA-seq patients vs controls) must be normalized and
  restricted to their shared genes prior to annotation or benchmarking.
when_to_use_negative:
- The expression data are already normalized ('normcounts') and train/test feature
  spaces already match — re-normalizing and re-subsetting is redundant
- The target sample is expected to contain unknown or novel cell types not present
  in the reference/training data, since NeuCA and this preprocessing path assume all
  cell types exist in the reference
- The task is de novo cell discovery with no labeled training data — unsupervised-only
  workflows are the appropriate setting
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: NeuCA
  role: Downstream supervised annotation method that consumes the normalized, gene-overlapped
    SingleCellExperiment train/test objects
  repo: https://bioconductor.org/packages/NeuCA/
- name: SingleCellExperiment
  role: Bioconductor container holding the normalized expression matrix ('normcounts'
    assay) and cell labels required by NeuCA
- name: scmap
  role: Baseline supervised annotation method run on the same normalized, gene-overlapped
    preprocessed data during benchmarking
- name: CHETAH
  role: Baseline supervised annotation method run on the same normalized, gene-overlapped
    preprocessed data during benchmarking
- name: Seurat
  role: Unsupervised baseline clustering method applied to the same preprocessed data
    in both train-test directions
- name: SC3
  role: Unsupervised baseline clustering method applied to the same preprocessed data
    in both train-test directions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:8dfa976b80f72d5196a852116d05b03447bc01d7ed57bafb70658cc376c0d672
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
  - scmap
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# scrnaseq-expression-matrix-normalization

## Summary

Normalizes scRNA-seq expression matrices and subsets training and testing data to their overlapped genes before supervised cell-type annotation (e.g., NeuCA) in intra-study, inter-study, and cross-condition settings.

## When to use

Use when raw scRNA-seq count matrices for a labeled training set and a testing set (e.g., ASD snRNA-seq patients vs controls) must be normalized and restricted to their shared genes prior to annotation or benchmarking.

## When NOT to use

- The expression data are already normalized ('normcounts') and train/test feature spaces already match — re-normalizing and re-subsetting is redundant
- The target sample is expected to contain unknown or novel cell types not present in the reference/training data, since NeuCA and this preprocessing path assume all cell types exist in the reference
- The task is de novo cell discovery with no labeled training data — unsupervised-only workflows are the appropriate setting

## Inputs

- Raw scRNA-seq count matrix for the training dataset with known cell-type labels (e.g., 15 ASD patient snRNA-seq samples)
- Raw scRNA-seq count matrix for the testing dataset (e.g., control samples)
- Gene/feature symbol names as row identifiers
- Published cell label vector for the training cells

## Outputs

- Normalized SingleCellExperiment objects with a 'normcounts' assay and non-duplicated rowData feature_symbol
- Gene-overlapped (intersected) training and testing expression matrices on an identical feature space
- Normalized inputs ready for NeuCA and baseline annotation methods (scmap, CHETAH, Seurat, SC3)

## How to apply

Wrap each count matrix in a Bioconductor SingleCellExperiment with the normalized expression stored in the 'normcounts' assay, setting rowData feature_symbol to gene names and removing features with duplicated names. Normalize the expression matrices, then subset the training and testing data to their overlapped genes so classifier training and prediction operate on an identical feature space. This shared, normalized representation is what NeuCA (averaged cell-type profiles, Pearson correlation matrix, tau=0.95 feed-forward vs hierarchical network selection) and baseline methods (scmap-cluster, scmap-cell, CHETAH, Seurat, SC3) consume, ensuring benchmark comparisons use the same preprocessed input.

## Related tools

- **NeuCA** (Downstream supervised annotation method that consumes the normalized, gene-overlapped SingleCellExperiment train/test objects) — https://bioconductor.org/packages/NeuCA/
- **SingleCellExperiment** (Bioconductor container holding the normalized expression matrix ('normcounts' assay) and cell labels required by NeuCA)
- **scmap** (Baseline supervised annotation method run on the same normalized, gene-overlapped preprocessed data during benchmarking)
- **CHETAH** (Baseline supervised annotation method run on the same normalized, gene-overlapped preprocessed data during benchmarking)
- **Seurat** (Unsupervised baseline clustering method applied to the same preprocessed data in both train-test directions)
- **SC3** (Unsupervised baseline clustering method applied to the same preprocessed data in both train-test directions)

## Examples

```
Baron_sce <- SingleCellExperiment(assays = list(normcounts = as.matrix(Baron_counts)), colData = Baron_anno); rowData(Baron_sce)$feature_symbol <- rownames(Baron_sce); Baron_sce <- Baron_sce[!duplicated(rownames(Baron_sce)), ]
```

## Evaluation signals

- After subsetting, training and testing matrices share an identical gene set (row dimension and rownames match)
- rowData feature_symbol is set for every feature and contains no duplicated names
- Expression values are stored in a 'normcounts' assay of the SingleCellExperiment, not raw counts
- Every benchmarked method (NeuCA, scmap-cluster, scmap-cell, CHETAH, Seurat, SC3) is run on the same normalized, gene-overlapped input in both train-test directions (ASD→control and control→ASD)
- Downstream predicted.label vector length equals the number of cells in the testing dataset, and accuracy/ARI can be computed against published true labels

## Limitations

- NeuCA, the downstream consumer of this preprocessing, is restricted to settings where all cell types in the new sample already exist in the reference/training data
- Cross-condition prediction success (demonstrated on ASD snRNA-seq data) may not hold in scenarios other than the ASD application
- Deciding whether unknown/novel cell types exist in the data is not addressed; biological/clinical expert consultation is recommended, and rare/unknown cell identification tools are cited only by reference numbers

## Evidence

- [methods] After the preprocessing steps, such as normalizing the expression matrix and taking subsets of: "After the preprocessing steps, such as normalizing the expression matrix and taking subsets of"
- [readme] Baron_sce = SingleCellExperiment(assays = list(normcounts = as.matrix(Baron_counts)), colData = Baron_anno): "Baron_sce = SingleCellExperiment(assays = list(normcounts = as.matrix(Baron_counts)), colData = Baron_anno)"
- [readme] remove features with duplicated names: "remove features with duplicated names"
- [discussion] NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data.: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
- [discussion] Although our application to ASD data shows that NeuCA can be applied to cross-condition predictions, this may not hold in other scenarios.: "Although our application to ASD data shows that NeuCA can be applied to cross-condition predictions, this may not hold in other scenarios."
