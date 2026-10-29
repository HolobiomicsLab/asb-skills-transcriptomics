---
name: rna-seq-cancer-normal-classification
description: Use when you have cancer and normal RNA-seq expression matrices (bulk
  or single-cell) plus a fixed gene set and need to benchmark its generalization accuracy
  on independent GEO datasets with a neural-network classifier.
when_to_use_negative:
- When the goal is to select genes de novo rather than validate a fixed 14-gene signature
  — feature selection is done by the Wx DI-score algorithm (10,000 iterations), not
  by this validation skill
- When the task requires experimental (wet-lab) verification of biomarker usefulness
  — the paper notes validation was computational only and 'Further experimental validation
  will be necessary'
- When the comparison classifier should differ — this skill specifically uses a Keras
  neural network; SVM/XGBoost comparisons are separate evaluations
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3474
tools:
- name: Keras
  role: Neural-network framework used to train the cancer/normal classifier on each
    14-gene subset
  repo: https://github.com/deargen/DearWXpub
- name: TensorFlow GPU 1.4.0
  role: 'Backend learning framework for the Keras classifier (note: backend choice
    can change selected features)'
- name: Python 3.4
  role: Runtime environment in which the validation pipeline is executed
- name: Wx
  role: Algorithm whose 14-gene UGCB set (Wx-14-UGCB) is being validated against the
    Peng-14-UGCB baseline
  repo: https://github.com/deargen/DearWXpub
- name: edgeR
  role: Source of the comparison DEG-based 14-gene set (Peng-14-UGCB) used as the
    baseline signature
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:29a7363a9068f2416b185ed4cfb0f75a3787dede301c759083dedecea45d9899
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Keras
  - Python
  - Python 3.4
  - TensorFlow
  - TensorFlow GPU 1.4.0
  - Wx
  - edgeR
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# rna-seq-cancer-normal-classification

## Summary

Validates the Wx-14-UGCB 14-gene cancer/normal signature against the Peng-14-UGCB (edgeR-derived) set on three independent GEO RNA-seq datasets (GSE40419 lung adenocarcinoma, GSE72056 melanoma, GSE103322 head and neck single-cell) by training a Keras neural-network classifier and reporting LOOCV / 5-fold cross-validation test accuracy.

## When to use

Use when you have cancer and normal RNA-seq expression matrices (bulk or single-cell) plus a fixed gene set and need to benchmark its generalization accuracy on independent GEO datasets with a neural-network classifier.

## When NOT to use

- When the goal is to select genes de novo rather than validate a fixed 14-gene signature — feature selection is done by the Wx DI-score algorithm (10,000 iterations), not by this validation skill
- When the task requires experimental (wet-lab) verification of biomarker usefulness — the paper notes validation was computational only and 'Further experimental validation will be necessary'
- When the comparison classifier should differ — this skill specifically uses a Keras neural network; SVM/XGBoost comparisons are separate evaluations

## Inputs

- GEO RNA-seq expression matrices: GSE40419 (lung adenocarcinoma, 164 samples), GSE72056 (melanoma, 4513 samples), GSE103322 (HNSC single cell, 5902 samples)
- Wx-14-UGCB gene list: EEF1A1, FN1, GAPDH, SFTPC, AHNAK, KLK3, UMOD, CTSB, COL1A1, GPX3, GNAS, ATP1A1, SFTPB, ACTB
- Peng-14-UGCB (edgeR DEG) gene list: KIF4A, NUSAP1, HJURP, NEK2, FANCI, DTL, UHRF1, FEN1, IQGAP3, KIF20A, TRIM59, CENPL, C16ORF59, UBE2C

## Outputs

- Per-dataset, per-gene-set classification accuracies (expected: GSE40419 80.00% vs 56.87%, GSE72056 90.71% vs 70.22%, GSE103322 81.10% vs 68.28%)
- A results table (e.g., geo_validation_accuracies.csv) with dataset and gene-set labels

## How to apply

Download the external evaluation datasets from GEO — GSE40419 (lung adenocarcinoma, 87 malignant/77 normal, total 164), GSE72056 (melanoma, 1257 malignant/3256 normal, total 4513), and GSE103322 (head and neck squamous cell carcinoma single cells, 2215 malignant/3687 normal, total 5902) — and load each expression matrix. Subset features to the 14 genes of the Wx-14-UGCB set (EEF1A1, FN1, GAPDH, SFTPC, AHNAK, KLK3, UMOD, CTSB, COL1A1, GPX3, GNAS, ATP1A1, SFTPB, ACTB) and, separately, the 14 genes of the Peng-14-UGCB (edgeR-derived) set. Train a neural-network classifier in Keras 2.1.2 with a TensorFlow (GPU 1.4.0) backend under Python 3.4. Estimate generalization with leave-one-out cross validation (train on n−1 samples, validate on the remaining one) or the README's 5-fold cross validation on the defined train/dev/test splits (e.g., GSE103322 trn+dev=4722, tst=1180). Record the six accuracy values: GSE40419 80.00% (Wx) vs 56.87% (Peng), GSE72056 90.71% vs 70.22%, GSE103322 81.10% vs 68.28%, verifying that Wx-14-UGCB outperforms Peng-14-UGCB on all three datasets.

## Related tools

- **Keras** (Neural-network framework used to train the cancer/normal classifier on each 14-gene subset) — https://github.com/deargen/DearWXpub
- **TensorFlow GPU 1.4.0** (Backend learning framework for the Keras classifier (note: backend choice can change selected features))
- **Python 3.4** (Runtime environment in which the validation pipeline is executed)
- **Wx** (Algorithm whose 14-gene UGCB set (Wx-14-UGCB) is being validated against the Peng-14-UGCB baseline) — https://github.com/deargen/DearWXpub
- **edgeR** (Source of the comparison DEG-based 14-gene set (Peng-14-UGCB) used as the baseline signature)

## Examples

```
$ pip install tensorflow-gpu && pip install keras && python src/wx_tcga.py
```

## Evaluation signals

- Reproduced GSE40419 accuracy is 80.00% for Wx-14-UGCB and 56.87% for Peng-14-UGCB on the test set (132 train/32 test split)
- Reproduced GSE72056 accuracy is 90.71% (Wx) vs 70.22% (Peng) under 5-fold cross validation on the 3611/902 train-test split
- Reproduced GSE103322 accuracy is 81.10% (Wx) vs 68.28% (Peng) on the 1180-sample test set
- Invariant: Wx-14-UGCB accuracy exceeds Peng-14-UGCB accuracy on all three independent GEO datasets
- Feature table for each run contains exactly the 14 genes of the respective signature (no overlap between the two sets per the paper)

## Limitations

- The 14-gene Wx-14-UGCB signature 'lacks extensive experimental validation' — evaluation here is purely computational; the discussion states further experimental validation will be necessary
- Selected features and results can differ with the backend learning framework — the README notes 'Selected Features can be different as which backend learning framework used' and that the Keras version differs from the paper's naive TensorFlow implementation
- The original environment is pinned (Python 3.4, TensorFlow GPU 1.4.0, Keras 2.1.2); results on modern stacks are not guaranteed
- Neural-network-based approaches require a large number of samples for good classification performance; small datasets (e.g., GSE40419 with 164 samples) are the weakest cases

## Evidence

- [results] GSE40419 accuracy: "the LOOCV method, which estimates the generalization performance of a given model trained on n – 1 samples and validates this with the remaining sample, was applied to each UGCB set"
- [intro] three GEO datasets: "three different cancer RNA-seq data from gene expression omnibus (GEO) were used to evaluate the performance of the identified biomarker candidates"
- [readme] README GSE40419 result: "Accuracy Result(test set)
WX14 : 80.00%
Peng14 : 56.87%"
- [readme] README GSE72056 result: "Accuracy Result (test set)
WX14 : 90.71%
Peng14 : 70.22%"
- [readme] backend caveat: "Selected Features can be different as which backend learning framework used"
