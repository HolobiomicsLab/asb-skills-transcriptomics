---
name: classification-accuracy-evaluation
description: Use when you must benchmark competing fixed gene-signature (biomarker
  candidate) sets for cancer-vs-normal classification on a labeled TCGA-style expression
  matrix, and check each signature's LOOCV accuracy against published values.
when_to_use_negative:
- When the goal is feature selection itself (ranking genes by Wx discriminative index
  score) rather than evaluating the accuracy of already-fixed gene signatures
- When evaluating multi-class tumor typing rather than binary cancer-vs-normal classification,
  since the paper's UGCB evaluation distinguishes cancer from normal samples
- 'When the Martinez-7 set is being scored: it has no cancer-type-agnostic gene list
  (genes differ per cancer type, and the README reports no TOTAL row for it), so a
  naive single gene-set subset will be wrong'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: XGBoost
  role: Classifier trained in each LOOCV fold to score each UGCB set; preferred over
    SVM for higher classification accuracy
- name: Wx
  role: Source of the Wx-14/Wx-7 UGCB gene sets being evaluated; runs in Keras/TensorFlow
    on the TCGA data
  repo: https://github.com/deargen/DearWXpub
- name: TCGA-Assembler
  role: Module_A.R tool used to download the TCGA RNA-seq data that form the evaluation
    cohort
  repo: http://www.compgenome.org/TCGA-Assembler/index.php
- name: edgeR
  role: Source of the competing DEG-14-UGCB set (top 14 DEGs sorted by ascending adjusted
    p value)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:d03482f55039f7496cd2bbcdfbff410ec74f1210ff074137887183de54e269a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Python
  - TCGA-Assembler
  - Wx
  - XGBoost
  - edgeR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# classification-accuracy-evaluation

## Summary

Evaluates the generalization performance of competing universal gene-expression cancer biomarker (UGCB) sets (e.g., Wx-14 vs Peng-14, DEG-14, Martinez-7, Wx-7) by leave-one-out cross-validation (LOOCV) with XGBoost on the TCGA pan-cancer cohort, comparing total accuracy against paper-reported values.

## When to use

Use when you must benchmark competing fixed gene-signature (biomarker candidate) sets for cancer-vs-normal classification on a labeled TCGA-style expression matrix, and check each signature's LOOCV accuracy against published values.

## When NOT to use

- When the goal is feature selection itself (ranking genes by Wx discriminative index score) rather than evaluating the accuracy of already-fixed gene signatures
- When evaluating multi-class tumor typing rather than binary cancer-vs-normal classification, since the paper's UGCB evaluation distinguishes cancer from normal samples
- When the Martinez-7 set is being scored: it has no cancer-type-agnostic gene list (genes differ per cancer type, and the README reports no TOTAL row for it), so a naive single gene-set subset will be wrong

## Inputs

- Preprocessed TCGA pan-cancer expression matrix (TCGA_DATAS folder; 12 cancer types plus normal controls, e.g., 6226 samples with 5609 tumor and 617 normal in the repo data snapshot)
- Gene-set lists: Wx-14-UGCB, Wx-7-UGCB, Peng-14-UGCB, DEG-14-UGCB (edgeR top-14 DEGs), MartinezL-7-UGCB (cancer-type-specific 7-gene sets)
- Paper-reported total accuracy values for validation (Wx-14 96.72, Wx-7 95.74, Peng-14 94.59, DEG-14 94.81, Martinez-7 92.20)

## Outputs

- loocv_accuracy_comparison.csv with one total LOOCV accuracy per gene set
- Per-fold (held-out sample) hit/miss predictions and per-cancer-type accuracy breakdowns

## How to apply

Load the preprocessed TCGA pan-cancer expression matrix from the TCGA_DATAS folder (12 cancer types plus normal controls) and subset it to each gene set: Wx-14-UGCB (['EEF1A1','FN1','GAPDH','SFTPC','AHNAK','KLK3','UMOD','CTSB','COL1A1','GPX3','GNAS','ATP1A1','SFTPB','ACTB']), Wx-7, Peng-14, DEG-14, and Martinez-7 (which is cancer-type-specific, e.g., SMAD2/RUNX2/ABTB1/ST5/CEBPB/SETDB1/CEBPG for BLCA). For each set, run LOOCV: train an XGBoost classifier on n-1 samples and validate on the held-out sample, then accumulate hits over all folds to obtain total accuracy (expected: Wx-14 96.72%, Wx-7 95.74%, Peng-14 94.59%, DEG-14 94.81%, Martinez-7 92.20%). XGBoost is chosen because the paper found it achieved higher classification accuracy than SVM on the same UGCB set, but repeat the evaluation with SVM as a robustness check. Write all five total accuracies to loocv_accuracy_comparison.csv and verify each is within 1 percentage point of the reported value.

## Related tools

- **XGBoost** (Classifier trained in each LOOCV fold to score each UGCB set; preferred over SVM for higher classification accuracy)
- **Wx** (Source of the Wx-14/Wx-7 UGCB gene sets being evaluated; runs in Keras/TensorFlow on the TCGA data) — https://github.com/deargen/DearWXpub
- **TCGA-Assembler** (Module_A.R tool used to download the TCGA RNA-seq data that form the evaluation cohort) — http://www.compgenome.org/TCGA-Assembler/index.php
- **edgeR** (Source of the competing DEG-14-UGCB set (top 14 DEGs sorted by ascending adjusted p value))

## Examples

```
git clone https://github.com/deargen/DearWXpub.git && cd DearWXpub && python src/wx_tcga.py
```

## Evaluation signals

- Each gene set's LOOCV total accuracy must fall within 1 percentage point of the paper-reported value (Wx-14 96.72, Wx-7 95.74, Peng-14 94.59, DEG-14 94.81, Martinez-7 92.20)
- Wx-14-UGCB should show higher accuracy than Peng-14-UGCB for 5 out of 7 cancer types, matching the paper's reported comparison
- The per-cancer-type accuracy table should roughly reproduce the README values (e.g., BLCA 95.79, BRCA 98.19, KIRC 99.67 for Wx-14) on the same validation split
- SVM reruns on the same UGCB sets should not exceed the XGBoost accuracies, consistent with the paper's finding that XGBoost outperformed SVM
- The classified sample count must match the validation split used (e.g., TOTAL 3119 samples, half of the 6226-sample download, when splitting half for feature selection / half for validation)

## Limitations

- Selected features can differ depending on the neural-network backend: the README notes Keras (vs the paper's naive TensorFlow) 'Selected Features can be different as which backend learning framework used', and some TCGA samples were updated so the repo has more samples than the paper (6226 vs the paper's 4,513-sample split)
- The 14-gene Wx-14-UGCB signature lacks extensive experimental validation, so accuracy figures alone do not establish biomarker usefulness
- Since selected features vary by algorithm and none of the Wx UGCBs overlapped with Peng or Martinez-Ledesma sets, accuracy comparisons alone cannot establish which feature-selection algorithm is better without experimental verification

## Evidence

- [results] LOOCV applied per UGCB set: "the LOOCV method, which estimates the generalization performance of a given model trained on n – 1 samples and validates this with the remaining sample, was applied to each UGCB set"
- [results] XGBoost chosen over SVM: "The result showed that XGBoost achieved higher classification accuracy compared to SVM with the same UGCB set."
- [readme] README accuracy benchmark for Wx-14: "**TOTAL** | **3119** | **3017** | **96.72** | **2961** | **94.93** | **2957** | **94.81**"
- [readme] README data split for accuracy evaluation: "TCGA data( Downloaded at Dec. 26th. 2017 ), Half of data for feature selection / Half of data for validation"
- [readme] README gene marker lists used in the comparison: "`['EEF1A1','FN1','GAPDH','SFTPC','AHNAK','KLK3','UMOD','CTSB','COL1A1','GPX3','GNAS','ATP1A1','SFTPB','ACTB']`"
