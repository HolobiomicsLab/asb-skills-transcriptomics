---
name: binary-sensitivity-specificity-computation
description: Use when comparing automatic scRNA-seq cell-type classifiers on a binary
  malignant-vs-non-malignant detection task, where 'unknown'/'unassigned' predictions
  must be scored as malignant positives against held-out ground-truth labels.
when_to_use_negative:
- When the evaluation target is multi-class accuracy, F1-score, unlabeling rate, or
  running time across many cell types rather than a binary malignant/non-malignant
  call
- When the test data lack ground-truth labels for malignant vs non-malignant cells,
  since both metrics require annotated positives and negatives
- When evaluating marker-based methods (e.g., markersupervised interface) trained
  only on marker lists with no reference-trained predictions on a tumor test set
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper package used to train and apply the automatic methods (eagersupervised/lazysupervised/markersupervised)
    whose predictions feed the sensitivity/specificity computation
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: caret (R package)
  role: Provides 'createDataPartition' to downsample both lung datasets to ~10,000
    cells with an equal proportion of each cell type, ensuring balanced positives/negatives
    for the binary metrics
- name: CELLBLAST
  role: One of the lazysupervised methods whose unlabeling behavior ('unknown' predictions)
    is converted to malignant calls before scoring
  repo: https://github.com/gao-lab/Cell_BLAST
- name: clustifyr
  role: Reference method achieving nearly 100% specificity and sensitivity for malignant
    tumor-cell detection in the lung transfer benchmark
- name: scmap-cell
  role: Lazysupervised method applied to the normal-lung reference and scored on the
    tumor test set
  repo: https://github.com/hemberg-lab/scmap
- name: CaSTLe
  role: Eagerly supervised method evaluated in the normal-lung to lung-cancer transfer
    and scored for sensitivity/specificity
  repo: https://github.com/yuvallb/CaSTLe
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:72a28b701ad82e108d9a4b78e89cf059b0fc98bd4b9d8aea48c48ee321a1e290
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
  - caret (R package)
  - clustifyr
  - scmap-cell
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# binary-sensitivity-specificity-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes binary sensitivity (proportion of malignant tumor cells correctly identified) and specificity (proportion of non-malignant cells correctly identified) for automatic cell-type identification methods evaluated on a normal-lung → lung-cancer transfer task, as in the Xie et al. 2021 benchmark of 32 methods.

## When to use

Use when comparing automatic scRNA-seq cell-type classifiers on a binary malignant-vs-non-malignant detection task, where 'unknown'/'unassigned' predictions must be scored as malignant positives against held-out ground-truth labels.

## When NOT to use

- When the evaluation target is multi-class accuracy, F1-score, unlabeling rate, or running time across many cell types rather than a binary malignant/non-malignant call
- When the test data lack ground-truth labels for malignant vs non-malignant cells, since both metrics require annotated positives and negatives
- When evaluating marker-based methods (e.g., markersupervised interface) trained only on marker lists with no reference-trained predictions on a tumor test set

## Inputs

- Count matrix and manually annotated cell-type labels for human normal lung scRNA-seq data (training reference, GSE131907)
- Count matrix and manually annotated cell-type labels for late-stage lung tumor scRNA-seq data (test set, GSE131907)
- Per-method prediction vectors from AutomaticCellTypeIdentification wrappers, including 'unknown'/'unassigned' calls

## Outputs

- Table with one sensitivity value in [0, 1] per evaluated method
- Table with one specificity value in [0, 1] per evaluated method
- Method ranking (e.g., Fig. 3B ordering) by sensitivity and specificity

## How to apply

Split the labeled count matrices (e.g., human normal lung and lung tumor from GSE131907, downsampled to ~10,000 cells with an equal proportion of each cell type via caret's 'createDataPartition') into a normal-lung training set and a lung-tumor test set. Train each method (CELLBLAST, CellFishing.jl, scmap-cell, ACTINN, CaSTLe, CHETAH, clustifyr, etc.) via the AutomaticCellTypeIdentification R package wrappers (lazysupervised/eagersupervised) on the normal-lung training data, then apply each trained model to the lung tumor test data. Convert each method's predictions into binary calls: predicted 'unknown'/'unassigned' cells are treated as malignant; all other predicted cell types are non-malignant negatives. Compare predictions against the ground-truth labels of the tumor test set to compute sensitivity (true malignant cells correctly flagged) and specificity (non-malignant cells correctly kept), each in [0, 1], per method. Rank methods on both metrics and highlight those reaching near-100% on both (clustifyr in Fig. 3B).

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper package used to train and apply the automatic methods (eagersupervised/lazysupervised/markersupervised) whose predictions feed the sensitivity/specificity computation) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **caret (R package)** (Provides 'createDataPartition' to downsample both lung datasets to ~10,000 cells with an equal proportion of each cell type, ensuring balanced positives/negatives for the binary metrics)
- **CELLBLAST** (One of the lazysupervised methods whose unlabeling behavior ('unknown' predictions) is converted to malignant calls before scoring) — https://github.com/gao-lab/Cell_BLAST
- **clustifyr** (Reference method achieving nearly 100% specificity and sensitivity for malignant tumor-cell detection in the lung transfer benchmark)
- **scmap-cell** (Lazysupervised method applied to the normal-lung reference and scored on the tumor test set) — https://github.com/hemberg-lab/scmap
- **CaSTLe** (Eagerly supervised method evaluated in the normal-lung to lung-cancer transfer and scored for sensitivity/specificity) — https://github.com/yuvallb/CaSTLe

## Examples

```
lazysupervised(train,test,label_train,method='CELLBLAST')
```

## Evaluation signals

- The output table contains exactly one sensitivity and one specificity value, each in [0, 1], for every evaluated method
- The per-method values reproduce the ordering of Fig. 3B, with clustifyr showing near-100% specificity and sensitivity
- Cells predicted 'unknown'/'unassigned' by lazy methods are counted as malignant positives, not excluded from scoring
- Sensitivity is computed only over ground-truth malignant cells and specificity only over ground-truth non-malignant cells in the downsampled (~10,000 cell) tumor test set
- Both lung datasets were downsampled with equal per-cell-type proportions (caret 'createDataPartition') before training, so class balance matches the paper's protocol

## Limitations

- Treating 'unknown'/'unassigned' predictions as malignant conflates a method's unlabeling rate with malignant detection, so methods with high unlabeling rates may appear to score well for the wrong reason
- Metrics are computed on ~10,000-cell downsampled datasets with equal cell-type proportions, so absolute values may not transfer to full unbalanced datasets
- Results depend on cross-dataset transferability (normal lung as reference for late-stage lung cancer); performance can differ when reference and target come from the same tissue condition

## Evidence

- [methods] Overall performance of automatic methods consists of accuracy, F1-score, speed, unlabeling rate, sensitivity and specificity: "Overall performance of automatic methods consists of accuracy, F1-score, speed, unlabeling rate, sensitivity and specificity"
- [results] clustifyr outperformed the other automatic methods in detecting malignant tumor cells, achieving nearly 100% specificity and sensitivity when predicting on late-stage lung cancer data using normal lung data as reference.: [evidence span withheld — anchor status: not_found]
- [methods] The human normal lung and lung tumor datasets are downsampled (∼10,000 cells) with an equal proportion of each cell type: "The human normal lung and lung tumor datasets are downsampled (∼10,000 cells) with an equal proportion of each cell type"
- [readme] The input of training and testing data is count matrix, the row is gene and the column is cell.: [evidence span withheld — anchor status: external_doc]
