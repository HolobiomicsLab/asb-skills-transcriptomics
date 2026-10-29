---
name: scrna-seq-count-matrix-handling
description: Use when raw scRNA-seq count matrices (e.g. from GEO, ArrayExpress, GSA,
  Tabula Muris, PBMC SCP424, GSE131907, GSE116470) must be filtered, normalized, and
  split into train/test matrices with labels for cell-type identification.
when_to_use_negative:
- Input is already a normalized/processed feature table or Seurat object rather than
  a raw count matrix
- The goal is marker-based prediction only and canonical markers from PanglaoDB/CellMarker/CancerSEA
  are available, so no training matrix split is needed
- Data are not single-cell RNA-seq (e.g. bulk RNA-seq or spatial transcriptomics)
  and the 200-gene/mitochondrial-percentage QC filters do not apply
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutomaticCellTypeIdentification
  role: R package whose wrappers (eagersupervised, lazysupervised, markersupervised)
    consume the prepared count matrices and labels
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: Seurat
  role: Provides NormalizeData for count-matrix normalization and FindAllMarkers for
    marker derivation from the training matrix
- name: caret (R package)
  role: createDataPartition and createFolds split filtered cells into training/testing
    subsets
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:f9d7d1f6a2aa0c94b3ecf7375a736e3932c10a50a2d55d11c8a6e15dedb2e2f4
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
  - Seurat
  - caret (R package)
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# scrna-seq-count-matrix-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare scRNA-seq count matrices (genes as rows, cells as columns) with manual cell-type labels into the shared training/testing input format consumed by the AutomaticCellTypeIdentification wrappers, applying the paper's QC filters and normalization.

## When to use

Use when raw scRNA-seq count matrices (e.g. from GEO, ArrayExpress, GSA, Tabula Muris, PBMC SCP424, GSE131907, GSE116470) must be filtered, normalized, and split into train/test matrices with labels for cell-type identification.

## When NOT to use

- Input is already a normalized/processed feature table or Seurat object rather than a raw count matrix
- The goal is marker-based prediction only and canonical markers from PanglaoDB/CellMarker/CancerSEA are available, so no training matrix split is needed
- Data are not single-cell RNA-seq (e.g. bulk RNA-seq or spatial transcriptomics) and the 200-gene/mitochondrial-percentage QC filters do not apply

## Inputs

- Raw scRNA-seq count matrix (rows = genes, columns = cells), e.g. from GEO/ArrayExpress/GSA
- Manually annotated per-cell cell-type labels
- Optional: mitochondrial gene annotation for QC filtering

## Outputs

- Filtered and normalized count matrix split into train and test matrices
- label_train vector of manually annotated cell types aligned to training cells
- Marker table (e.g. marker_5.csv) with cluster, gene, avg logFC and statistics per cell type

## How to apply

Download the count matrix and manually annotated labels from public resources, orienting the matrix so rows are genes and columns are cells. Apply the paper's QC filters: drop genes not expressed in any cells, drop cells with fewer than 200 expressed genes, drop cells exceeding twice the standard deviation of the mean expressed genes, drop cells with mitochondrial gene expression above 20%, and delete cells lacking cell-type information. Normalize the count matrix with the Seurat NormalizeData function (as wrapped by diffmarker). Partition cells into training and testing sets using caret's createDataPartition (or createFolds for five-fold cross-validation), keeping labels aligned with cells so the matrices can be passed to eagersupervised/lazysupervised as (train, test, label_train) and derived marker lists (e.g. top 5 per type via FindAllMarkers with only.pos=TRUE) to markersupervised.

## Related tools

- **AutomaticCellTypeIdentification** (R package whose wrappers (eagersupervised, lazysupervised, markersupervised) consume the prepared count matrices and labels) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **Seurat** (Provides NormalizeData for count-matrix normalization and FindAllMarkers for marker derivation from the training matrix)
- **caret (R package)** (createDataPartition and createFolds split filtered cells into training/testing subsets)

## Examples

```
eagersupervised(train,test,label_train,method='Seurat')
```

## Evaluation signals

- Matrix orientation check: rows are genes and columns are cells as required by the wrappers
- No zero-expression genes and no cells failing the 200-gene, 2x-SD, or >20% mitochondrial filters remain
- label_train length equals the number of columns (cells) in the training matrix
- Each wrapper call returns a prediction vector for the test cells without argument errors
- Marker table is non-empty and contains cluster, gene, average log fold-change, and statistics for every cell type in the training labels

## Limitations

- Cells with no cell type information must be deleted beforehand, since supervised wrappers require labels for training cells
- Python-based wrapped methods require the corresponding conda environment loaded at the beginning in R
- Lung datasets were downsampled to ~10,000 cells with equal cell-type proportions in the paper; very imbalanced raw matrices may need analogous treatment

## Evidence

- [readme] The input of training and testing data is count matrix, the row is gene and the column is cell.: [evidence span withheld — anchor status: external_doc]
