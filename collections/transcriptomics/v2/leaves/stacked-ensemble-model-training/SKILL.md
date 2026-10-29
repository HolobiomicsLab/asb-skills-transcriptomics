---
name: stacked-ensemble-model-training
description: Use when building a binary RNA-modification (e.g., m5C) predictor from
  balanced 41-bp sequence datasets where a single RF- or SVM-based model underperforms
  and ensemble stacking should be benchmarked.
when_to_use_negative:
- Input is already a numeric feature table unrelated to sequence encoding, or the
  task requires a general cross-species model — the paper notes species-specific models
  are necessary and cross-species prediction is difficult
- The query sequences are not 41-bp segments centered on a candidate cytosine, or
  the target species is neither A. thaliana nor M. musculus (the shipped m5C_predict.py
  accepts only 'Arabidopsis' or 'Mouse')
- A simple single-classifier baseline (RF or SVM) is sufficient and the added stacking
  complexity is not justified by the accuracy comparison
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3474
tools:
- name: Staem5
  role: Stacked ensemble pipeline and m5C prediction script providing training data
    and the stacking framework
  repo: https://github.com/Cxd-626/Staem5.git
- name: scikit-learn
  role: Implements the base classifiers (SVM with radial basis kernel, XGBoost, LightGBM
    wrappers) and 10-fold cross-validation
- name: mlxtend
  role: Implements the stacking ensemble with logistic regression as the meta-classifier
- name: iLearn
  role: Computes the PCPseDNC encoding (38 physicochemical properties) used in the
    fused feature set
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:a73822e80718ed3f31aeb3fcf44a85eab316600779ac4d97548a6f6f204c4171
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - LightGBM
  - PCC
  - Python
  - Staem5
  - iLearn
  - mlxtend
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# stacked-ensemble-model-training

## Summary

Trains a stacked ensemble classifier (Staem5-style) for m5C site prediction by combining base classifiers (SVM, XGBoost, LightGBM) with a logistic regression meta-classifier over fused sequence-encoding features, evaluated by 10-fold cross-validation.

## When to use

Use when building a binary RNA-modification (e.g., m5C) predictor from balanced 41-bp sequence datasets where a single RF- or SVM-based model underperforms and ensemble stacking should be benchmarked.

## When NOT to use

- Input is already a numeric feature table unrelated to sequence encoding, or the task requires a general cross-species model — the paper notes species-specific models are necessary and cross-species prediction is difficult
- The query sequences are not 41-bp segments centered on a candidate cytosine, or the target species is neither A. thaliana nor M. musculus (the shipped m5C_predict.py accepts only 'Arabidopsis' or 'Mouse')
- A simple single-classifier baseline (RF or SVM) is sufficient and the added stacking complexity is not justified by the accuracy comparison

## Inputs

- Balanced FASTA training set of 41-bp sequence segments centered on candidate m5C sites (e.g., 5,298 positive and 5,298 negative for A. thaliana; 4,563:4,563 for M. musculus)
- Fused feature matrix from PSP, k-mer (k=1,2,3), PCPseDNC, and PseEIIP encodings
- F-score-ranked feature list with progressive dimension reduction (step of 5)

## Outputs

- Trained stacked model (SVM + XGBoost + LightGBM base classifiers with logistic regression meta-classifier)
- 10-fold cross-validation metrics report (Sn, Sp, Acc, MCC, AUC), e.g., cv_metrics.csv / mouse_cv_metrics.csv
- Per-segment cross-validation predictions (cv_predictions.csv)
- Stacking-vs-voting comparison table (table2_stacking_vs_voting_cv_results.csv)

## How to apply

Encode each 41-bp sequence segment by fusing four schemes — PSP, k-mer (k=1,2,3), PCPseDNC (38 physicochemical properties computed with iLearn), and PseEIIP — then rank features with the F score and add top-ranked features in steps of 5 to reduce dimension and computing time. Train base classifiers (SVM with radial basis kernel and C/gamma tuned by Bayesian optimization, XGBoost, LightGBM) in scikit-learn, having first compared stacked combinations of candidate classifiers (SVM, XGBoost, LightGBM, ExtraTree, GBDT), adding classifiers until accuracy decreased. Stack the selected base classifiers with a logistic regression meta-classifier implemented via the mlxtend package. Run 10-fold cross-validation on the balanced training set (e.g., A. thaliana 5,298:5,298; M. musculus 4,563:4,563) and compute Sn, Sp, Acc, MCC, and AUC per the paper's Equations 1-4.

## Related tools

- **Staem5** (Stacked ensemble pipeline and m5C prediction script providing training data and the stacking framework) — https://github.com/Cxd-626/Staem5.git
- **scikit-learn** (Implements the base classifiers (SVM with radial basis kernel, XGBoost, LightGBM wrappers) and 10-fold cross-validation)
- **mlxtend** (Implements the stacking ensemble with logistic regression as the meta-classifier)
- **iLearn** (Computes the PCPseDNC encoding (38 physicochemical properties) used in the fused feature set)

## Examples

```
python m5C_predict.py --input query.fasta --output predictions.csv --species Arabidopsis
```

## Evaluation signals

- Reproduced 10-fold CV accuracy for the A. thaliana stacked model (SVM+XGBoost+LightGBM) is approximately 73.89% with MCC approximately 0.479 and AUC approximately 0.810
- Reproduced M. musculus stacked-model metrics match accuracy 77.42%, MCC 0.549, and AUC 0.855
- Stacking slightly outperforms majority voting for every base-classifier combination (e.g., A. thaliana stacking 73.89% vs voting 73.68%; M. musculus stacking 77.12% vs voting 76.83%)
- F-score feature selection at 185 features reaches the best M. musculus CV accuracy (77.42%), versus 77.21% for MRMD and PCC at 230 and 280 dimensions
- Output files (cv_metrics.csv, cv_predictions.csv, table2_stacking_vs_voting_cv_results.csv) exist and contain Sn, Sp, Acc, MCC, and AUC columns per fold

## Limitations

- No general cross-species model: it is necessary to set up species-specific models, and Staem5 can be inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent A. thaliana test set
- Selected hyperparameters (C, gamma) are only available in supplementary Table S2, pre-set according to previous experience rather than fully systematically derived
- Detailed stacking comparison results are only in supplementary Table S3; the README requires a pinned Python 3.7 environment (numpy 1.19.5, sklearn 0.21.3, xgboost 0.90, lightgbm 2.3.0, mlxtend 0.17.3, tensorflow 1.14)

## Evidence
