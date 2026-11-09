---
name: cross-validation-performance-evaluation
description: Use when validating an m5C site prediction model on a balanced training
  set (e.g., 5,298 pos/5,298 neg A. thaliana or 4,563 pos/4,563 neg M. musculus 41-bp
  sequences) and reporting 10-fold cross-validation metrics.
when_to_use_negative:
- Evaluating on the independent A. thaliana test set, where Staem5 was reported to
  be inferior to iRNA-m5C_SVM and m5CPred-SVM — this skill covers training-set cross-validation
  only
- Input is unbalanced or species data other than A. thaliana / M. musculus; the article
  notes it is difficult to develop a general cross-species model and species-specific
  models are required
- The input is already a table of encoded features and cross-validated predictions
  rather than raw FASTA sequences to be encoded and classified
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: Staem5
  role: Stacked ensemble m5C prediction pipeline providing training data, feature
    fusion, and the stacked model
  repo: https://github.com/Cxd-626/Staem5.git
- name: scikit-learn
  role: Implementation of SVM (RBF kernel), XGBoost, LightGBM base classifiers and
    10-fold cross-validation
- name: mlxtend
  role: Model stacking with logistic regression as the meta-classifier
- name: iLearn
  role: Computes the PCPseDNC feature encoding of the 41-bp sequence segments
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:150579fbce7807c769290b3f2948d42423608fcb792550bd76e4b46f2af3d5bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - LightGBM
  - Python
  - Staem5
  - iLearn
  - mlxtend
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cross-validation-performance-evaluation

## Summary

Reproduce Staem5's stacked m5C prediction model (SVM, XGBoost, LightGBM base classifiers stacked with logistic regression) and evaluate it with 10-fold cross-validation using Sn, Sp, Acc, MCC, and AUC metrics on the A. thaliana and M. musculus training sets.

## When to use

Use when validating an m5C site prediction model on a balanced training set (e.g., 5,298 pos/5,298 neg A. thaliana or 4,563 pos/4,563 neg M. musculus 41-bp sequences) and reporting 10-fold cross-validation metrics.

## When NOT to use

- Evaluating on the independent A. thaliana test set, where Staem5 was reported to be inferior to iRNA-m5C_SVM and m5CPred-SVM — this skill covers training-set cross-validation only
- Input is unbalanced or species data other than A. thaliana / M. musculus; the article notes it is difficult to develop a general cross-species model and species-specific models are required
- The input is already a table of encoded features and cross-validated predictions rather than raw FASTA sequences to be encoded and classified

## Inputs

- Balanced A. thaliana m5C training dataset (5,298 positive and 5,298 negative 41-bp sequence segments) from the Staem5 repository
- Balanced M. musculus m5C training set (4,563 positive and 4,563 negative sequences) from the Staem5 repository data
- F-score-selected 185-feature subset derived from fused PCPseDNC, PSP, k-mer (k=1,2,3), and PseEIIP encodings

## Outputs

- cv_metrics report / mouse_cv_metrics.csv with 10-fold cross-validation Sn, Sp, Acc, MCC, and AUC
- cv_predictions.csv with per-segment cross-validation predictions
- base_classifier_accuracies.csv tabulating before/after Bayesian-optimization accuracies for SVM, XGBoost, LightGBM, ExtraTree, GBDT

## How to apply

Load the balanced training dataset shipped with the Staem5 repository and encode each 41-bp sequence segment by fusing four feature schemes — PCPseDNC (computed with iLearn), PSP, k-mer (k=1,2,3), and PseEIIP — applying F-score-based feature selection with a reduction step of 5. Train the base classifiers (SVM with radial basis kernel whose C and gamma are tuned via Bayesian optimization, XGBoost, LightGBM) in scikit-learn, then stack them with a logistic regression meta-classifier implemented via mlxtend. Run 10-fold cross-validation on the training set and compute Sn, Sp, Acc, MCC, and AUC per the paper's Equations 1–4, saving the metrics report and per-segment predictions. For A. thaliana the reproduced stacked model should reach approximately 73.89% accuracy, 0.479 MCC, and 0.810 AUC; for M. musculus approximately 77.42% accuracy, 0.549 MCC, and 0.855 AUC.

## Related tools

- **Staem5** (Stacked ensemble m5C prediction pipeline providing training data, feature fusion, and the stacked model) — https://github.com/Cxd-626/Staem5.git
- **scikit-learn** (Implementation of SVM (RBF kernel), XGBoost, LightGBM base classifiers and 10-fold cross-validation)
- **mlxtend** (Model stacking with logistic regression as the meta-classifier)
- **iLearn** (Computes the PCPseDNC feature encoding of the 41-bp sequence segments)

## Examples

```
python m5C_predict.py --input query.fa --species Arabidopsis --output predictions.csv
```

## Evaluation signals

- Reproduced A. thaliana 10-fold cross-validation accuracy is approximately 73.89%, MCC approximately 0.479, AUC approximately 0.810
- Reproduced M. musculus stacked-model metrics match accuracy 77.42%, MCC 0.549, AUC 0.855
- Base-classifier comparison reproduces SVM improving from 62.40% to 73.62% and GBDT from 65.69% to 71.77% after Bayesian optimization, with SVM showing the largest gain
- Metrics report contains all five defined measures (Sn, Sp, Acc, MCC, AUC) computed per Equations 1–4
- Per-segment predictions file (cv_predictions.csv) has one prediction per training sequence segment

## Limitations

- No general cross-species model: it could be difficult to develop a general model for cross-species prediction, and species-specific models are necessary
- Hyperparameters were pre-set according to previous experience rather than systematically derived; selected parameters are only listed in supplementary Table S2
- Detailed performance metrics, stacking comparison results, and feature selection results are only in supplementary Tables S1, S3, and S5–S7
- Staem5 was inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent test set of A. thaliana
- Requires pinned versions per README (python 3.7, sklearn 0.21.3, xgboost 0.90, lightgbm 2.3.0, mlxtend 0.17.3, numpy 1.19.5)

## Evidence
