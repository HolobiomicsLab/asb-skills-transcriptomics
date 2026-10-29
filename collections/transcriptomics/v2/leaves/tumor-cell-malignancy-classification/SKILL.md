---
name: tumor-cell-malignancy-classification
description: Use when you have normal-tissue scRNA-seq reference data with cell-type
  labels and tumor scRNA-seq test data, and need to identify malignant cells as those
  left unassigned by cell-type classifiers.
when_to_use_negative:
- The goal is ordinary cell-type assignment between matched tissues, not new (malignant)
  cell-type detection — in that setting unassigned cells are not positives but errors.
- There is no labeled normal-tissue reference available for the tumor's tissue of
  origin, so classifier training is impossible.
- The dataset is the self-projection cross-validation benchmark (e.g. Tabula Muris
  or PBMC SCP424), where labels exist in both train and test and malignant-cell detection
  is not defined.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper package used to train each automatic cell-type identification method
    on normal lung data and predict on lung tumor data
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: caret (R package)
  role: provides the 'createDataPartition' function used to downsample both lung datasets
    to ~10,000 cells with an equal proportion of each cell type
- name: Seurat
  role: preprocessing pipeline and marker identification ('NormalizeData', 'FindAllMarkers')
    used in the cell-type identification workflow
- name: CELLBLAST
  role: lazy-supervised method evaluated for malignant-cell detection via unknown
    predictions
  repo: https://github.com/gao-lab/Cell_BLAST
- name: CellFishing.jl
  role: lazy-supervised method evaluated for malignant-cell detection via unknown
    predictions
  repo: https://github.com/bicycle1885/CellFishing.jl
- name: scmap-cell
  role: lazy-supervised method evaluated for malignant-cell detection via unknown
    predictions
  repo: https://github.com/hemberg-lab/scmap
- name: ACTINN
  role: eager-supervised method evaluated for malignant-cell detection via unknown
    predictions
  repo: https://github.com/mafeiyang/ACTINN
- name: CaSTLe
  role: eager-supervised method evaluated for malignant-cell detection via unknown
    predictions
  repo: https://github.com/yuvallb/CaSTLe
- name: CHETAH
  role: eager-supervised method evaluated for malignant-cell detection via unknown
    predictions
  repo: https://github.com/jdekanter/CHETAH
- name: clustifyr
  role: method that outperformed others in detecting malignant tumor cells, achieving
    nearly 100% specificity and sensitivity (Fig. 3B)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:13e85f3fedd466a7e76092dfb342dee1ab3b8766509e4cc4fdd2fcd4c5acd1c1
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
  - ACTINN
  - AutomaticCellTypeIdentification
  - CELLBLAST
  - CHETAH
  - CaSTLe
  - CellFishing.jl
  - R
  - Seurat
  - caret (R package)
  - clustifyr
  - scmap-cell
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# tumor-cell-malignancy-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects malignant tumor cells in late-stage lung cancer scRNA-seq data by applying automatic cell-type identification methods trained on matched normal lung reference data, treating 'unknown'/'unassigned' predictions as malignant. Validated by sensitivity/specificity per method, with clustifyr achieving nearly 100% of both.

## When to use

Use when you have normal-tissue scRNA-seq reference data with cell-type labels and tumor scRNA-seq test data, and need to identify malignant cells as those left unassigned by cell-type classifiers.

## When NOT to use

- The goal is ordinary cell-type assignment between matched tissues, not new (malignant) cell-type detection — in that setting unassigned cells are not positives but errors.
- There is no labeled normal-tissue reference available for the tumor's tissue of origin, so classifier training is impossible.
- The dataset is the self-projection cross-validation benchmark (e.g. Tabula Muris or PBMC SCP424), where labels exist in both train and test and malignant-cell detection is not defined.

## Inputs

- human normal lung scRNA-seq count matrix with manually annotated cell-type labels (GSE131907)
- late-stage lung tumor scRNA-seq count matrix with manually annotated cell-type labels (GSE131907)

## Outputs

- per-method table of sensitivity and specificity values in [0, 1] for malignant-cell detection
- per-cell predictions on the tumor test data, with 'unknown'/'unassigned' calls mapped to malignant

## How to apply

Download human normal lung and lung tumor count matrices with annotations from GEO accession GSE131907. Preprocess by filtering genes not expressed in any cells, cells with fewer than 200 expressed genes, cells exceeding twice the standard deviation of the mean expressed-gene count, cells with mitochondrial expression above 20%, and cells lacking cell-type labels. Downsample both datasets to ~10,000 cells with an equal proportion of each cell type using 'createDataPartition' in the caret R package. Train automatic cell-type identification methods (via the AutomaticCellTypeIdentification R package, e.g. CELLBLAST, CellFishing.jl, scmap-cell, ACTINN, CaSTLe, CHETAH) on the normal-lung training data, then apply each trained method to the lung tumor test data. Because malignant cells have no counterpart cell type in the normal reference, treat 'unknown'/'unassigned' predictions as malignant cells, and compute sensitivity (malignant positives correctly identified) and specificity (non-malignant negatives correctly identified) for each method.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper package used to train each automatic cell-type identification method on normal lung data and predict on lung tumor data) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **caret (R package)** (provides the 'createDataPartition' function used to downsample both lung datasets to ~10,000 cells with an equal proportion of each cell type)
- **Seurat** (preprocessing pipeline and marker identification ('NormalizeData', 'FindAllMarkers') used in the cell-type identification workflow)
- **CELLBLAST** (lazy-supervised method evaluated for malignant-cell detection via unknown predictions) — https://github.com/gao-lab/Cell_BLAST
- **CellFishing.jl** (lazy-supervised method evaluated for malignant-cell detection via unknown predictions) — https://github.com/bicycle1885/CellFishing.jl
- **scmap-cell** (lazy-supervised method evaluated for malignant-cell detection via unknown predictions) — https://github.com/hemberg-lab/scmap
- **ACTINN** (eager-supervised method evaluated for malignant-cell detection via unknown predictions) — https://github.com/mafeiyang/ACTINN
- **CaSTLe** (eager-supervised method evaluated for malignant-cell detection via unknown predictions) — https://github.com/yuvallb/CaSTLe
- **CHETAH** (eager-supervised method evaluated for malignant-cell detection via unknown predictions) — https://github.com/jdekanter/CHETAH
- **clustifyr** (method that outperformed others in detecting malignant tumor cells, achieving nearly 100% specificity and sensitivity (Fig. 3B))

## Examples

```
eagersupervised(train,test,label_train,method='clustifyr')
```

## Evaluation signals

- Every evaluated method yields a sensitivity and a specificity value, each within [0, 1].
- Per-method values reproduce the ordering of Fig. 3B, with clustifyr showing near-100% specificity and sensitivity.
- Both lung datasets are downsampled to ~10,000 cells with an equal proportion of each cell type before training/testing.
- Preprocessing removes genes not expressed in any cells, cells with <200 expressed genes, cells >2 SD above the mean expressed-gene count, cells with >20% mitochondrial expression, and cells without cell-type labels.
- All 'unknown'/'unassigned' predictions on the tumor test data are consistently counted as malignant positives when computing sensitivity.

## Limitations

- The 'unknown = malignant' assumption only holds when tumor test data contains cell types absent from the normal reference; unassigned cells in matched-tissue settings indicate method failure, not malignancy.
- Results depend on downsampling to ~10,000 cells with equal per-type proportions; imbalanced sampling may shift sensitivity/specificity.
- Python-based methods in the AutomaticCellTypeIdentification package require loading a conda environment in R before running.
- Filtering thresholds (200 genes, 2 SD, 20% mitochondrial) may remove stressed or malignant subpopulations from the reference or test data.

## Evidence

- [other] clustifyr outperformed the other automatic methods in detecting malignant tumor cells, achieving nearly 100% specificity and sensitivity when predicting on late-stage lung cancer data using normal lung data as reference.: [evidence span withheld — anchor status: not_found]
- [readme] README interface examples for running methods in the wrapper package: [evidence span withheld — anchor status: not_found]
