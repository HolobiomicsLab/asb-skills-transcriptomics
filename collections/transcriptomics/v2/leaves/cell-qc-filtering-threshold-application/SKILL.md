---
name: cell-qc-filtering-threshold-application
description: Use when preparing Tabula Muris or similar scRNA-seq count matrices (genes
  x cells) as training/testing input for cell-type identification methods, so that
  low-quality cells and unannotated cells are removed with the paper's exact thresholds.
when_to_use_negative:
- Input is already a QC-filtered, normalized expression matrix (e.g., a processed
  Seurat object) rather than a raw count matrix
- The downstream task is marker-based identification (e.g., markersupervised methods
  like SCSA/SCINA) where cell-level training labels are not required in the same way
- The analysis goal is de novo clustering without reference labels, since the pipeline
  requires cells with known cell-type information
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: R pipeline used to preprocess count matrices and find cell-type specific markers
    prior to method evaluation
- name: AutomaticCellTypeIdentification
  role: Downstream R package that consumes the filtered count matrices (gene rows,
    cell columns) for automatic cell-type identification
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: R
  role: Implementation environment for the Seurat-based preprocessing and filtering
    steps
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:5ab2a076fb0dc2a301e1ad4b01938ab372b243eb3db29dfdd37571becadb3b32
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiebb123456/AutomaticCellTypeIdentification
  repo_url: https://github.com/xiebb123456/AutomaticCellTypeIdentification
  tools:
  - AutomaticCellTypeIdentification
  - R
  - SciBet
  - Seurat
  - SingleR
  - clustifyr
  - scHPL
  - scLearn
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cell-qc-filtering-threshold-application

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the paper's standardized single-cell RNA-seq QC filtering thresholds — dropping genes not expressed in any cell, cells with fewer than 200 expressed genes, cells exceeding twice the standard deviation of the mean expressed-gene count, cells with more than 20% mitochondrial reads, and cells lacking manual labels — before automatic cell-type identification.

## When to use

Use when preparing Tabula Muris or similar scRNA-seq count matrices (genes x cells) as training/testing input for cell-type identification methods, so that low-quality cells and unannotated cells are removed with the paper's exact thresholds.

## When NOT to use

- Input is already a QC-filtered, normalized expression matrix (e.g., a processed Seurat object) rather than a raw count matrix
- The downstream task is marker-based identification (e.g., markersupervised methods like SCSA/SCINA) where cell-level training labels are not required in the same way
- The analysis goal is de novo clustering without reference labels, since the pipeline requires cells with known cell-type information

## Inputs

- Raw scRNA-seq count matrix (rows = genes, columns = cells), e.g., from Tabula Muris
- Manually annotated cell-type labels for each cell

## Outputs

- Filtered count matrix with low-quality genes/cells removed
- Label table restricted to cells retaining cell-type information
- Filtered matrices ready for train/test splitting (e.g., Smart-seq2 vs 10x)

## How to apply

Start from a raw count matrix with manually annotated labels downloaded from a public resource (e.g., https://tabula-muris.ds.czbiohub.org/), typically handled in R/Seurat. Apply the filters in order: remove genes not expressed in any cells; remove cells whose number of expressed genes is less than 200; remove cells whose expressed-gene count exceeds twice the standard deviation of the mean; remove cells with more than 20% mitochondrial gene expression; and delete cells with no cell type information. The rationale is that each method evaluated (via the AutomaticCellTypeIdentification package) expects a clean count matrix (rows = genes, columns = cells) paired with labels, and inconsistent QC would distort comparisons of accuracy, F1-score, unlabeling rate and running time. After filtering, the matrix can be split (e.g., Smart-seq2 tissues as training, 10x tissues as testing) and passed to eagersupervised/lazysupervised interfaces.

## Related tools

- **Seurat** (R pipeline used to preprocess count matrices and find cell-type specific markers prior to method evaluation)
- **AutomaticCellTypeIdentification** (Downstream R package that consumes the filtered count matrices (gene rows, cell columns) for automatic cell-type identification) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **R** (Implementation environment for the Seurat-based preprocessing and filtering steps)

## Examples

```
eagersupervised(train, test, label_train, method='Seurat')
```

## Evaluation signals

- No genes remain in the filtered matrix that are not expressed in any cells
- All retained cells have at least 200 expressed genes and no more than twice the standard deviation of the mean expressed-gene count
- All retained cells have mitochondrial gene expression at or below 20%
- Every retained cell has non-missing manual cell-type information (cells without labels were deleted)
- Downstream method runs (e.g., eagersupervised(train,test,label_train,method='Seurat')) execute without errors on the filtered matrices

## Limitations

- Thresholds (200 genes, 2x SD, 20% mitochondrial) were tuned for the Tabula Muris mouse tissue datasets; other tissues or platforms may need recalibration
- Deleting cells with no cell type information biases the benchmark toward methods needing labeled training data
- Normalization (e.g., NormalizeData) is a separate downstream step; filtering alone does not prepare the data for all methods

## Evidence

- [methods] genes not expressed in any cells were filtered: "genes not expressed in any cells were filtered"
- [methods] the number of expressed genes in the cells is less than 200: "the number of expressed genes in the cells is less than 200"
- [methods] the expression of mitochondrial genes in the cells is more than 20%: "the expression of mitochondrial genes in the cells is more than 20%"
- [methods] The count matrix and manually annotated labels are downloaded from public resources: "The count matrix and manually annotated labels are downloaded from public resources"
- [readme] The input of training and testing data is count matrix, the row is gene and the column is cell.: [evidence span withheld — anchor status: external_doc]
