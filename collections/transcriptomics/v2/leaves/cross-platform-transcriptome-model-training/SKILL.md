---
name: cross-platform-transcriptome-model-training
description: Use when a training set from one transcriptomic platform (e.g., RNA-seq
  GSE151189 .csv with sample names in the first column and labels in the last) must
  yield a model that predicts clinical outcomes on samples from a different platform
  (e.g., GSE59098 microarray).
when_to_use_negative:
- The training and validation datasets share no common genes — Ciclops fails silently,
  producing ./training/ csv splits with only sample-name and label columns and no
  gene expression columns
- The goal is SHAP-based biomarker analysis with a regression model (gpr or lr) —
  SHAP is implemented only for LightGBM, XGBoost, and Random Forest
- The datasets were already jointly normalized during preprocessing, in which case
  quantile normalization should be skipped via --no_quantile
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Ciclops
  role: Core pipeline that preprocesses the training data, performs 10-fold cross
    validation, trains the model, and transfers it to the validation platform dataset
  repo: https://github.com/GuanLab/ciclops.git
- name: LightGBM
  role: Default machine learning model type (-m lgb) used by Ciclops for cross-platform
    outcome prediction
  repo: https://github.com/GuanLab/ciclops.git
- name: XGBoost
  role: Alternative model type (-m xgb) selectable in the Ciclops pipeline
  repo: https://github.com/GuanLab/ciclops.git
- name: Python
  role: Runtime used to invoke the pipeline via 'python ciclops' after installing
    the package with pip
  repo: https://github.com/GuanLab/ciclops.git
- name: pip
  role: Installs the Ciclops package ('pip install ciclops')
  repo: https://github.com/GuanLab/ciclops.git
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.101583
    title: Protocol for using Ciclops to build models trained on cross-platform transcriptome
      data for clinical outcome prediction
  source_content_hash: sha256:da7acdfae1ed0ab927f5f56979a07e920b2e21a13f48156b929790134c36fdb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GuanLab/ciclops.git
  tools:
  - Ciclops
  - LightGBM
  - Python
  - XGBoost
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-platform-transcriptome-model-training

## Summary

Train and transfer clinical outcome prediction models across transcriptomic platforms using Ciclops, which preprocesses a training .csv, runs 10-fold cross-validation, and applies the fitted model to an external validation dataset (e.g., GSE151189 RNA-seq training transferred to the GSE59098 microarray validation set).

## When to use

Use when a training set from one transcriptomic platform (e.g., RNA-seq GSE151189 .csv with sample names in the first column and labels in the last) must yield a model that predicts clinical outcomes on samples from a different platform (e.g., GSE59098 microarray).

## When NOT to use

- The training and validation datasets share no common genes — Ciclops fails silently, producing ./training/ csv splits with only sample-name and label columns and no gene expression columns
- The goal is SHAP-based biomarker analysis with a regression model (gpr or lr) — SHAP is implemented only for LightGBM, XGBoost, and Random Forest
- The datasets were already jointly normalized during preprocessing, in which case quantile normalization should be skipped via --no_quantile

## Inputs

- Training dataset as .csv (sample names in first column, labels in last column), e.g., GSE151189
- Transfer validation dataset as .csv (sample names in first column, labels in last column), e.g., GSE59098
- Model type flag: lgb (LightGBM, default), xgb (XGBoost), rf (Random Forest), gpr (Gaussian Process Regression), or lr (Linear Regression)

## Outputs

- ./training/: preprocessed training dataset splits for model training and 10-fold cross validation
- ./validation/: validation dataset for the transferring test
- ./params/: trained machine learning model parameters
- ./performance/: model performance in 10-fold cross validation and transferring test, reported as 95% confidence intervals obtained by bootstrapping
- ./SHAP/: SHAP analysis results (only with --shap and tree-based models)

## How to apply

Format each GEO dataset as a .csv with sample names as the first column and labels as the last column, then run the Ciclops pipeline once per model type (-m lgb default, plus xgb, rf, gpr, lr) with --train_path pointing to the training csv and --valid_path pointing to the transfer-validation csv. Keep quantile normalization enabled (do not pass --no_quantile) unless the datasets were already jointly normalized, since quantile normalization was key to the successful cross-platform model in the original study. The pipeline preprocesses the training data for 10-fold cross-validation into ./training/, transfers trained models to the validation set saving parameters into ./params/, and writes cross-validation and transferring-test performance metrics to ./performance/. SHAP analysis (--shap with -n top genes, default 20) is available only for tree-based models (lgb, xgb, rf). Judge success by checking that ./performance/ contains valid (non-NaN) metrics for both the 10-fold cross-validation and the transfer test, and no anomalous results such as AUPRC confidence intervals spanning [0, 1].

## Related tools

- **Ciclops** (Core pipeline that preprocesses the training data, performs 10-fold cross validation, trains the model, and transfers it to the validation platform dataset) — https://github.com/GuanLab/ciclops.git
- **LightGBM** (Default machine learning model type (-m lgb) used by Ciclops for cross-platform outcome prediction) — https://github.com/GuanLab/ciclops.git
- **XGBoost** (Alternative model type (-m xgb) selectable in the Ciclops pipeline) — https://github.com/GuanLab/ciclops.git
- **Python** (Runtime used to invoke the pipeline via 'python ciclops' after installing the package with pip) — https://github.com/GuanLab/ciclops.git
- **pip** (Installs the Ciclops package ('pip install ciclops')) — https://github.com/GuanLab/ciclops.git

## Examples

```
python ciclops --train_path GSE151189.csv --valid_path GSE59098.csv -m lgb
```

## Evaluation signals

- ./performance/ contains results for all selected model types (e.g., lgb, xgb, rf, gpr, lr) for both the 10-fold cross-validation and the transferring test on GSE59098
- All evaluation metrics are valid (non-NaN) in the ./performance/ tables
- No anomalous results such as confidence intervals spanning [0, 1] for AUPRC
- ./training/ csv splits contain gene expression columns, not just sample-name and label columns (their absence indicates no common genes between datasets)
- With quantile-normalized vs --no_quantile paired runs, a comparison table (e.g., quantile_normalization_comparison.csv) contains matched metric rows for both runs

## Limitations

- If training and test sets share no common genes, the program fails silently and csv files in ./training have only two columns (sample names and labels)
- Results can be unreliable (e.g., confidence intervals spanning [0, 1] for AUPRC, or NaN evaluation metrics) without automatic safeguards
- SHAP analysis is not implemented for the regression models (Gaussian Process Regression and Linear Regression) because it works best and fastest with tree-based models
- Users are limited in their choice of machine learning algorithms and must modify the source code to use their own; hyperparameters are not fixed and must be tuned for a better fit
- Model performance may vary across datasets and conditions due to data collection methods and environmental factors impacting gene expression

## Evidence

- [readme] Pipeline for building clinical outcome prediction models on training dataset and transfer learning on validation datasets.: "Pipeline for building clinical outcome prediction models on training dataset and transfer learning on validation datasets."
- [readme] Quantile normalization is on by default: "--no_quantile         If specified, do not use quantile normalization."
- [readme] Model performance output directory: "`./performance/`: model performance in 10-fold cross validation and transferring test"
- [methods] 10-fold cross validation on preprocessed training data: "preprocessed training datasets for model training and 10-fold cross validation"
- [discussion] Validation failure mode: "confidence intervals spanning [0, 1] for AUPRC, or NaN values for any of the evaluation metrics"
