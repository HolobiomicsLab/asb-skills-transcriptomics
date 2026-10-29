---
name: cancer-vs-normal-classification
description: Use when a binary Cancer/Normal classification must be run on a pan-cancer
  transcriptomic matrix using a fixed biomarker gene set (e.g., Wx-14-UGCB, Peng-14,
  DEG-14), with LOOCV accuracy as the reported metric.
when_to_use_negative:
- The goal is gene ranking/feature selection itself rather than classification with
  a fixed gene set — use the Wx feature-selection workflow instead
- The task is a multi-class (>2) problem, e.g., the k>2 Wx example (wx_gse_multi_class.py)
  — this skill covers binary Cancer/Normal classification
- The input data are not cancer/normal-labeled transcriptomic samples (e.g., the non-cancer
  GSE105127 transcriptomic set)
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: XGBoost
  role: Primary classifier trained on UGCB gene-expression features under LOOCV and
    5-fold CV; achieved higher accuracy than SVM with the same UGCB set
- name: SVM
  role: Second classifier repeated on the identical UGCB features and folds to test
    whether accuracy depends on the algorithm
- name: Wx
  role: Neural network-based feature selection algorithm that produced the Wx-14-UGCB
    / Wx-7-UGCB gene sets used as classification features
  repo: https://github.com/deargen/DearWXpub
- name: TCGA-Assembler
  role: R tool (Module_A.R) used to download the TCGA RNA-seq data into the TCGA_DATAS
    folder before classification
  repo: http://www.compgenome.org/TCGA-Assembler/index.php
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:59859d6e4404e35a336b0b33f9ab616d81b3174f559d51abaa1ab85563564626
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Python
  - SVM
  - TCGA-Assembler
  - TensorFlow GPU
  - Wx
  - XGBoost
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cancer-vs-normal-classification

## Summary

Classifies tumor vs normal samples in the TCGA pan-cancer RNA-seq cohort (12 cancer types) using the Wx-14-UGCB gene-expression signature as features, evaluated by leave-one-out cross validation with XGBoost and benchmarked against SVM.

## When to use

Use when a binary Cancer/Normal classification must be run on a pan-cancer transcriptomic matrix using a fixed biomarker gene set (e.g., Wx-14-UGCB, Peng-14, DEG-14), with LOOCV accuracy as the reported metric.

## When NOT to use

- The goal is gene ranking/feature selection itself rather than classification with a fixed gene set — use the Wx feature-selection workflow instead
- The task is a multi-class (>2) problem, e.g., the k>2 Wx example (wx_gse_multi_class.py) — this skill covers binary Cancer/Normal classification
- The input data are not cancer/normal-labeled transcriptomic samples (e.g., the non-cancer GSE105127 transcriptomic set)

## Inputs

- Preprocessed TCGA pan-cancer gene-expression matrix (TCGA_DATAS folder) with Cancer/Normal labels for 12 cancer types (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, THCA)
- Gene-set lists (Wx-14-UGCB, Wx-7-UGCB, Peng-14-UGCB, MartinezL-7-UGCB, DEG-14-UGCB)
- Independent GEO RNA-seq validation sets (GSE72056 melanoma, GSE40419 lung adenocarcinoma, GSE103322 head & neck single-cell)

## Outputs

- Per-cancer-type and total classification accuracy table (hit counts and accuracy %) per gene set
- loocv_accuracy_comparison.csv with LOOCV total accuracy for each UGCB set
- ROC/AUC curves for selected cancer types (e.g., LUSC, LUAD, BRCA)
- Cross-classifier comparison showing XGBoost vs SVM accuracy on the same UGCB set

## How to apply

Load the preprocessed TCGA_DATAS expression matrix (12 cancer types plus normal controls; 6,226 samples total: 5,609 tumor, 617 normal) with pandas. Subset the matrix to the chosen universal gene-expression cancer biomarker (UGCB) set — for Wx-14-UGCB the genes are EEF1A1, FN1, GAPDH, SFTPC, AHNAK, KLK3, UMOD, CTSB, COL1A1, GPX3, GNAS, ATP1A1, SFTPB, ACTB — so all classifiers see identical features. Train an XGBoost classifier with leave-one-out cross validation (train on n-1 samples, validate on the held-out sample), and optionally repeat with SVM on the same folds to compare algorithms. Tabulate per-cancer-type and total accuracies; the reported Wx-14 total LOOCV accuracy is 96.72%, and validation requires each gene set's accuracy to fall within 1 percentage point of the paper value (Wx-7 95.74, Peng-14 94.59, DEG-14 94.81, Martinez-7 92.20). For independent GEO validation (GSE72056, GSE40419, GSE103322), use 5-fold cross validation on defined train/dev/test splits instead of LOOCV.

## Related tools

- **XGBoost** (Primary classifier trained on UGCB gene-expression features under LOOCV and 5-fold CV; achieved higher accuracy than SVM with the same UGCB set)
- **SVM** (Second classifier repeated on the identical UGCB features and folds to test whether accuracy depends on the algorithm)
- **Wx** (Neural network-based feature selection algorithm that produced the Wx-14-UGCB / Wx-7-UGCB gene sets used as classification features) — https://github.com/deargen/DearWXpub
- **TCGA-Assembler** (R tool (Module_A.R) used to download the TCGA RNA-seq data into the TCGA_DATAS folder before classification) — http://www.compgenome.org/TCGA-Assembler/index.php

## Examples

```
git clone https://github.com/deargen/DearWXpub.git; cd DearWXpub; python src/wx_tcga.py
```

## Evaluation signals

- Wx-14-UGCB total LOOCV accuracy reproduces at 96.72% (within 1 percentage point of the paper value)
- All five gene sets (Wx-14, Wx-7, Peng-14, DEG-14, Martinez-7) appear in loocv_accuracy_comparison.csv and each is within 1 pp of its reported accuracy
- XGBoost accuracy exceeds SVM accuracy when both are run on the same UGCB feature set, with an almost-same overall accuracy trend
- Per-type accuracies match the README table patterns, e.g., Wx-14 KIRP 99.38%, BRCA 98.19%, LIHC 90.57%
- Independent GEO validations land in expected ranges: WX14 90.71% (GSE72056), 80.00% (GSE40419), 81.10% (GSE103322), each well above Peng-14

## Limitations

- Selected features can differ depending on the backend learning framework (Keras vs naive TensorFlow), so reproduced gene sets may vary
- Some TCGA samples were updated, so the repository cohort has more samples than the paper (README: data downloaded Dec 26, 2017; 6,226 samples)
- The Wx-14-UGCB 14-gene signature lacks extensive experimental validation and further experimental validation is needed to prove the Wx algorithm's usefulness
- Sample splits differ between paper (2,888 training / 723 validation / 902 test, 64%/16%/20%) and README (half feature selection / half validation), affecting comparability

## Evidence

- [results] XGBoost vs SVM comparison: "The result showed that XGBoost achieved higher classification accuracy compared to SVM with the same UGCB set."
- [results] LOOCV protocol: "the LOOCV method, which estimates the generalization performance of a given model trained on n – 1 samples and validates this with the remaining sample, was applied to each UGCB set"
- [results] Cohort definition: "a pan-cancer cohort from TCGA RNA-seq data consisting of 12 different types of cancer and normal (control) samples"
- [readme] Train/validation split: "TCGA data( Downloaded at Dec. 26th. 2017 ), Half of data for feature selection / Half of data for validation"
- [abstract] Application to TCGA cohort: "We applied the Wx algorithm to a TCGA pan-cancer gene-expression cohort to identify an optimal set of gene-expression biomarker candidates"
