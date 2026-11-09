---
name: classifier-accuracy-comparison
description: Use when a previously selected feature set must be stress-tested for
  classifier dependence — i.e., compare accuracy of two algorithms (e.g., XGBoost
  vs SVM) on the exact same features and splits.
when_to_use_negative:
- The goal is feature selection itself — use the Wx DI-score ranking algorithm rather
  than comparing classifiers on a fixed set
- The datasets differ between classifiers (features or folds not held identical),
  since the comparison is only meaningful with the same UGCB set
- Only a single classifier's performance is needed for reporting, not a cross-algorithm
  robustness check
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: XGBoost
  role: Trained on the UGCB features to obtain LOOCV classification accuracy for comparison
    against SVM
- name: SVM
  role: Alternative classifier trained on the identical UGCB feature set and folds
    to benchmark XGBoost
- name: Python 3.4
  role: Environment in which feature selection and classification accuracy evaluation
    are run
  repo: https://github.com/deargen/DearWXpub
- name: Wx
  role: Upstream neural-network feature selection algorithm that produced the UGCB
    set used by both classifiers
  repo: https://github.com/deargen/DearWXpub
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:2aa139fefcb38c0c2655962f065d595e5eaaf1e79b43c2344e1766441bc2afa6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Python
  - Python 3.4
  - SVM
  - TensorFlow GPU
  - Wx
  - XGBoost
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# classifier-accuracy-comparison

## Summary

Evaluates whether a fixed gene feature set (e.g., the Wx-14-UGCB biomarker candidates) yields consistent classification accuracy across classifiers by training XGBoost and SVM on identical features and folds, as done on the TCGA pan-cancer Cancer/Normal task.

## When to use

Use when a previously selected feature set must be stress-tested for classifier dependence — i.e., compare accuracy of two algorithms (e.g., XGBoost vs SVM) on the exact same features and splits.

## When NOT to use

- The goal is feature selection itself — use the Wx DI-score ranking algorithm rather than comparing classifiers on a fixed set
- The datasets differ between classifiers (features or folds not held identical), since the comparison is only meaningful with the same UGCB set
- Only a single classifier's performance is needed for reporting, not a cross-algorithm robustness check

## Inputs

- Preprocessed TCGA pan-cancer gene-expression matrix (TCGA_DATAS folder) with Cancer/Normal labels for 12 cancer types
- Fixed UGCB gene feature set (e.g., Wx-14-UGCB: EEF1A1, FN1, GAPDH, SFTPC, AHNAK, KLK3, UMOD, CTSB, COL1A1, GPX3, GNAS, ATP1A1, SFTPB, ACTB)
- Train/validation sample splits per cancer type

## Outputs

- Per-cancer-type LOOCV classification accuracy table for both classifiers
- Average/total accuracy values for SVM and XGBoost on the same UGCB set
- Comparison confirming whether XGBoost accuracy exceeds SVM accuracy

## How to apply

Subset the preprocessed TCGA pan-cancer expression matrix (TCGA_DATAS, 12 cancer types with Cancer/Normal labels) to the identical UGCB gene set so both classifiers see the same features. Randomly divide each cancer and its corresponding control group into training and validation sets, then apply leave-one-out cross validation (LOOCV), which trains on n−1 samples and validates on the remaining one. Train an SVM on the UGCB features and record per-cancer-type and average LOOCV accuracy; then train XGBoost on the same features and folds and record its accuracies. Tabulate both classifiers' accuracies side by side; the article's rationale is that algorithm-independence of a biomarker set is demonstrated when the overall trend of accuracy is almost the same across algorithms even if one (XGBoost) is higher.

## Related tools

- **XGBoost** (Trained on the UGCB features to obtain LOOCV classification accuracy for comparison against SVM)
- **SVM** (Alternative classifier trained on the identical UGCB feature set and folds to benchmark XGBoost)
- **Python 3.4** (Environment in which feature selection and classification accuracy evaluation are run) — https://github.com/deargen/DearWXpub
- **Wx** (Upstream neural-network feature selection algorithm that produced the UGCB set used by both classifiers) — https://github.com/deargen/DearWXpub

## Examples

```
$ python src/wx_tcga.py
```

## Evaluation signals

- Output table contains accuracy values for both classifiers on every evaluated cancer type with identical sample counts
- XGBoost accuracy exceeds SVM accuracy on the same UGCB feature set, matching the reported result
- Overall trend of classification accuracy is almost the same across both algorithms (robustness, not divergence)
- Both classifiers were evaluated on identical UGCB features and identical LOOCV folds (train on n−1, validate on 1)
- Accuracy values fall within plausible per-cancer ranges consistent with the reported TCGA results (roughly 90–100% for 14-gene sets)

## Limitations

- Selected features vary depending on the algorithm used, so the UGCB set must be fixed before classifier comparison
- It is difficult to establish which feature-selection algorithm is better without experimental verification
- A large number of samples are needed to achieve good classification or regression performance with neural-network-based approaches
- Keras and TensorFlow backends can select different features, and updated TCGA samples change results relative to the paper

## Evidence

- [results] XGBoost comparison finding: "The result showed that XGBoost achieved higher classification accuracy compared to SVM with the same UGCB set."
- [results] SVM re-evaluation step: "We repeated the evaluation with a different algorithm called support-vector machine (SVM)"
- [results] LOOCV protocol: "the LOOCV method, which estimates the generalization performance of a given model trained on n – 1 samples and validates this with the remaining sample"
- [results] Data split for validation: "The samples in each cancer and their corresponding control group were randomly divided into two sets, a training set and validation set"
- [readme] README run command and accuracy goal: "## Step 4. Do the Feature selection and Get Classification Accuracy"
