---
name: rna-m5c-site-prediction
description: Use when query RNA sequences in FASTA format need m5C site prediction
  for A. thaliana or M. musculus, or when reproducing/benchmarking stacked-ensemble
  m5C predictors against SVM/RF baselines like m5CPred-SVM or iRNA-m5C.
when_to_use_negative:
- The input species is neither A. thaliana nor M. musculus — the paper notes it is
  difficult to develop a general cross-species model and species-specific models are
  necessary
- The goal is prediction on an independent A. thaliana test set where maximum sensitivity
  is required — Staem5 was inferior to iRNA-m5C_SVM and m5CPred-SVM on that independent
  test set despite superior cross-validation results
- Input is already an encoded feature table — this skill's feature-fusion step (PCPseDNC,
  PSP, k-mer, PseEIIP encoding of raw 41-bp sequences) would be redundant
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: Staem5
  role: Stacked ensemble m5C prediction model (SVM, XGBoost, LightGBM base classifiers
    + logistic regression meta-classifier); run via m5C_predict.py
  repo: https://github.com/Cxd-626/Staem5.git
- name: scikit-learn
  role: Trains the base classifiers (SVM with RBF kernel, XGBoost, LightGBM) and evaluates
    10-fold cross-validation
- name: mlxtend
  role: Implements model stacking with logistic regression as the meta-classifier
- name: iLearn
  role: Computes the PCPseDNC feature encoding used in the fused feature set
- name: PEA-m5C
  role: Prior RF-based plant m5C predictor used as a comparison baseline
  repo: https://github.com/cma2015/PEA-m5C
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:9d2d86bcc23143602fa60f3b794c7adcfeebfdcd0adbc30558584b0ceda47ca8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - LightGBM
  - PEA-m5C
  - Python
  - Staem5
  - iLearn
  - iRNA-m5C
  - iRNA-m5C_SVM
  - m5CPred-SVM
  - mlxtend
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# rna-m5c-site-prediction

## Summary

Predict m5C RNA modification sites at single-nucleotide resolution using Staem5, a stacked ensemble of SVM, XGBoost, and LightGBM base classifiers with a logistic regression meta-classifier, built on fused sequence features (PCPseDNC, PSP, k-mer, PseEIIP) with F-score feature selection.

## When to use

Use when query RNA sequences in FASTA format need m5C site prediction for A. thaliana or M. musculus, or when reproducing/benchmarking stacked-ensemble m5C predictors against SVM/RF baselines like m5CPred-SVM or iRNA-m5C.

## When NOT to use

- The input species is neither A. thaliana nor M. musculus — the paper notes it is difficult to develop a general cross-species model and species-specific models are necessary
- The goal is prediction on an independent A. thaliana test set where maximum sensitivity is required — Staem5 was inferior to iRNA-m5C_SVM and m5CPred-SVM on that independent test set despite superior cross-validation results
- Input is already an encoded feature table — this skill's feature-fusion step (PCPseDNC, PSP, k-mer, PseEIIP encoding of raw 41-bp sequences) would be redundant

## Inputs

- Query RNA sequences in FASTA format (41-bp segments centered on candidate cytosines)
- Species identifier: 'Arabidopsis' or 'Mouse' (currently the only supported species)
- Balanced training datasets: A. thaliana (5,298 positive / 5,298 negative) and M. musculus (4,563 positive / 4,563 negative) from the Staem5 repository

## Outputs

- Per-sequence m5C prediction results written to a user-specified output file
- 10-fold cross-validation metrics (Sn, Sp, Acc, MCC, AUC) for model evaluation
- Stacked-model performance for A. thaliana: Acc 73.89%, MCC 0.479, AUC ~0.810; for M. musculus: Acc 77.42%, MCC 0.549, AUC 0.855

## How to apply

Encode each 41-bp sequence segment by fusing four feature schemes — PCPseDNC (computed with iLearn), position-specific propensity (PSP), k-mer (k=1,2,3), and PseEIIP — then reduce dimensionality with F-score feature selection using a reduction step of 5. Train base classifiers (SVM with radial basis kernel whose C and gamma are tuned by Bayesian optimization, XGBoost, and LightGBM) in scikit-learn (sklearn 0.21.3, xgboost 0.90, lightgbm 2.3.0 on Python 3.7), and stack them with a logistic regression meta-classifier implemented via the mlxtend package (0.17.3). Base classifiers were selected from five candidates (SVM, XGBoost, LightGBM, ExtraTree, GBDT) by evaluating combinations with stacked tactics and adding classifiers until accuracy decreased; SVM, XGBoost, and LightGBM proved optimal. Evaluate with 10-fold cross-validation computing Sn, Sp, Acc, MCC, and AUC; for A. thaliana the stacked model reaches 73.89% accuracy, 0.479 MCC, and ~0.810 AUC, and for M. musculus 77.42% accuracy, 0.549 MCC, and 0.855 AUC. Species-specific models are required — run with --species set to 'Arabidopsis' or 'Mouse' since no general cross-species model is supported.

## Related tools

- **Staem5** (Stacked ensemble m5C prediction model (SVM, XGBoost, LightGBM base classifiers + logistic regression meta-classifier); run via m5C_predict.py) — https://github.com/Cxd-626/Staem5.git
- **scikit-learn** (Trains the base classifiers (SVM with RBF kernel, XGBoost, LightGBM) and evaluates 10-fold cross-validation)
- **mlxtend** (Implements model stacking with logistic regression as the meta-classifier)
- **iLearn** (Computes the PCPseDNC feature encoding used in the fused feature set)
- **PEA-m5C** (Prior RF-based plant m5C predictor used as a comparison baseline) — https://github.com/cma2015/PEA-m5C

## Examples

```
python m5C_predict.py --input query.fa --output predictions.txt --species Arabidopsis
```

## Evaluation signals

- Reproduced A. thaliana 10-fold cross-validation accuracy is approximately 73.89% with MCC approximately 0.479 and AUC approximately 0.810
- Reproduced M. musculus cross-validation metrics match Acc 77.42%, MCC 0.549, AUC 0.855
- Stacking accuracy slightly exceeds voting accuracy for each base-classifier combination (e.g., A. thaliana 73.89% stacking vs 73.68% voting)
- F-score-selected feature subsets (e.g., 185 features for M. musculus) are recovered with reduction step of 5
- Environment pins match the README: numpy 1.19.5, pandas 1.2.4, sklearn 0.21.3, xgboost 0.90, lightgbm 2.3.0, mlxtend 0.17.3, Python 3.7

## Limitations

- No general cross-species model; species-specific models are required for A. thaliana and M. musculus
- Staem5 was inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent test set of A. thaliana, so cross-validation superiority does not guarantee test-set superiority
- Selected hyperparameters are only available in supplementary Table S2 and detailed metric/stacking comparisons in supplementary Tables S1 and S3, complicating exact reproduction
- Strict dependency versions in the README (e.g., tensorflow 1.14, keras 2.30, Python 3.7) may conflict with modern environments

## Evidence
