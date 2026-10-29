---
name: leave-one-out-cross-validation
description: Use when comparing cancer-vs-normal classification accuracy of alternative
  gene sets (e.g., Wx-14 vs Peng-14 UGCBs) on a small-to-moderate TCGA expression
  matrix, where every sample must serve as a validation case to estimate generalization
  performance.
when_to_use_negative:
- The dataset is large enough for k-fold splits (e.g., GEO validation sets GSE72056/GSE40419/GSE103322,
  where the authors used 5-fold cross validation on train/dev/test splits instead
  of LOOCV)
- The goal is feature selection rather than evaluation — use the Wx discriminative-index
  (DI) ranking (iterated 10,000 times) to pick genes; LOOCV only scores a fixed, already-selected
  gene set
- Input is raw TCGA downloads — the samples must first be preprocessed into the TCGA_DATAS
  expression matrix before LOOCV can be applied
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: XGBoost
  role: Classifier trained on n-1 samples per LOOCV fold to score each UGCB gene set;
    also benchmarked against SVM on the same UGCB features
  repo: https://github.com/deargen/DearWXpub
- name: Keras
  role: Neural-network backend used to train and validate classifiers under the paper's
    Python 3.4 / TensorFlow GPU 1.4.0 / Keras 2.1.2 environment
  repo: https://github.com/deargen/DearWXpub
- name: Wx
  role: Feature-selection algorithm whose output Wx-14-UGCB gene set is the primary
    input evaluated via LOOCV
  repo: https://github.com/deargen/DearWXpub
- name: Python
  role: Language for loading the TCGA_DATAS expression matrix (pandas) and running
    the LOOCV loops (e.g., src/wx_tcga.py)
  repo: https://github.com/deargen/DearWXpub
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:d74463619260543f068e2f9a5da7d10e1372225e82103d46342eb6accbb29a28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Keras
  - Python
  - TensorFlow
  - TensorFlow GPU
  - Wx
  - XGBoost
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# leave-one-out-cross-validation

## Summary

Leave-one-out cross validation (LOOCV) estimates the generalization performance of a classifier by training on n-1 samples and validating on the single held-out sample, repeated over every sample. In this article it is the benchmark protocol used to compare UGCB gene sets (Wx-14-UGCB, Peng-14-UGCB, DEG-14-UGCB, MartinezL-7-UGCB) on the TCGA pan-cancer cohort with 12 cancer types.

## When to use

Use when comparing cancer-vs-normal classification accuracy of alternative gene sets (e.g., Wx-14 vs Peng-14 UGCBs) on a small-to-moderate TCGA expression matrix, where every sample must serve as a validation case to estimate generalization performance.

## When NOT to use

- The dataset is large enough for k-fold splits (e.g., GEO validation sets GSE72056/GSE40419/GSE103322, where the authors used 5-fold cross validation on train/dev/test splits instead of LOOCV)
- The goal is feature selection rather than evaluation — use the Wx discriminative-index (DI) ranking (iterated 10,000 times) to pick genes; LOOCV only scores a fixed, already-selected gene set
- Input is raw TCGA downloads — the samples must first be preprocessed into the TCGA_DATAS expression matrix before LOOCV can be applied

## Inputs

- Preprocessed TCGA pan-cancer gene-expression matrix (TCGA_DATAS folder; 6226 samples: 5609 tumor, 617 normal across BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, THCA)
- Cancer/Normal sample labels per sample
- Candidate UGCB gene-set lists (Wx-14-UGCB, Wx-7-UGCB, Peng-14-UGCB, DEG-14-UGCB, MartinezL-7-UGCB)

## Outputs

- Per-fold LOOCV prediction (hit/miss) for each sample under each gene set
- Total and per-cancer-type classification accuracy values (e.g., loocv_accuracy_comparison.csv)

## How to apply

Load the preprocessed TCGA pan-cancer expression matrix (TCGA_DATAS, 12 cancer types plus normal controls) with pandas and subset it to the genes of each candidate UGCB set (e.g., the 14 genes EEF1A1, FN1, GAPDH, SFTPC, AHNAK, KLK3, UMOD, CTSB, COL1A1, GPX3, GNAS, ATP1A1, SFTPB, ACTB for Wx-14). For each gene set, run LOOCV: train the classifier (XGBoost, or a Keras neural network on Python 3.4 / TensorFlow GPU 1.4.0 / Keras 2.1.2) on n-1 samples and predict the remaining sample, iterating across all samples. Record per-fold and total classification accuracy (e.g., expected totals of 96.72% for Wx-14, 94.59% for Peng-14, 94.81% for DEG-14, 92.20% for MartinezL-7). The rationale is that LOOCV maximizes training data per fold, giving a fair, low-variance comparison when gene sets differ in size and provenance; the same folds must be reused across gene sets and classifiers so any accuracy difference is attributable to the feature set or algorithm.

## Related tools

- **XGBoost** (Classifier trained on n-1 samples per LOOCV fold to score each UGCB gene set; also benchmarked against SVM on the same UGCB features) — https://github.com/deargen/DearWXpub
- **Keras** (Neural-network backend used to train and validate classifiers under the paper's Python 3.4 / TensorFlow GPU 1.4.0 / Keras 2.1.2 environment) — https://github.com/deargen/DearWXpub
- **Wx** (Feature-selection algorithm whose output Wx-14-UGCB gene set is the primary input evaluated via LOOCV) — https://github.com/deargen/DearWXpub
- **Python** (Language for loading the TCGA_DATAS expression matrix (pandas) and running the LOOCV loops (e.g., src/wx_tcga.py)) — https://github.com/deargen/DearWXpub

## Examples

```
git clone https://github.com/deargen/DearWXpub.git && python src/wx_tcga.py
```

## Evaluation signals

- Each gene set's LOOCV total accuracy must fall within 1 percentage point of the paper-reported value: Wx-14 96.72%, Wx-7 95.74%, Peng-14 94.59%, DEG-14 94.81%, MartinezL-7 92.20%
- The number of LOOCV folds must equal the sample count (each sample is held out exactly once); per-cancer hit counts must sum to the total (e.g., 3017/3119 hits for Wx-14)
- Per-type accuracies must match expected patterns, e.g., Wx-14 99.67% on KIRC and 90.57% on LIHC, with Wx-14 exceeding Peng-14 on five of seven cancer types
- All gene sets and classifiers must be evaluated on identical LOOCV folds and identical feature matrices; any accuracy difference must be traceable to the gene set or algorithm, not data splits
- Output table must contain a labeled accuracy value for every gene-set/classifier pair, with Wx-14-UGCB comparable to or better than the previously reported biomarker sets

## Limitations

- The Wx-14-UGCB 14-gene signature lacks extensive experimental validation, so high LOOCV accuracy does not itself establish clinical biomarker utility
- Selected features vary depending on the algorithm and backend framework used (the Keras implementation can yield different selected features than the paper's naive TensorFlow version, and some TCGA samples were updated), so reproduced accuracies may deviate slightly
- Neural-network-based approaches require a large number of samples to achieve good performance; LOOCV accuracy can be unstable on very small cohorts

## Evidence

- [results] LOOCV estimates generalization performance: "the LOOCV method, which estimates the generalization performance of a given model trained on n – 1 samples and validates this with the remaining sample, was applied to each UGCB set"
- [abstract] Wx-14 comparable to or outperforming prior biomarkers: "The 14 gene-expression biomarker candidates identified by Wx were comparable to or outperformed previously reported universal gene expression biomarkers"
- [results] XGBoost vs SVM on same UGCB set: "The result showed that XGBoost achieved higher classification accuracy compared to SVM with the same UGCB set."
- [readme] Environment for running the classifiers: "Experienment Environments

$ python 3.4
$ tensorflow gpu 1.4.0
$ keras 2.1.2"
- [readme] GEO validation used 5-fold CV instead: "Dataset splits(trn,dev,tst) = (3611, 902)
We perfomred the 5 fold cross validation"
