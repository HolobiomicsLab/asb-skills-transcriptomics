---
name: machine-learning-model-selection-and-hyperparameter-application
description: Use when building clinical outcome prediction models from cross-platform
  transcriptome datasets (RNA-seq trained, microarray transferred), when comparing
  multiple ML model types, or when needing quantile-normalized transfer learning between
  platforms.
when_to_use_negative:
- The training and validation sets share no common genes — the program fails silently,
  producing ./training csv splits with only sample-name and label columns and no gene
  expression columns
- Only SHAP biomarker analysis is needed with a regression model (gpr or lr) — SHAP
  is implemented only for LightGBM, XGBoost, and Random Forest
- The data are already normalized or the analysis goal does not involve cross-platform
  transfer, since quantile normalization and platform transfer are core, default behaviors
  of this pipeline
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: Ciclops
  role: Pipeline that trains clinical outcome prediction models with 10-fold cross-validation
    and transfers them to a second transcriptomic platform, writing performance metrics
    to ./performance/
  repo: https://github.com/GuanLab/ciclops.git
- name: LightGBM
  role: Default model type (lgb) selectable via the -m flag for cross-validated training
    and transfer testing
- name: XGBoost
  role: Alternative model type (xgb) selectable via the -m flag for cross-validated
    training and transfer testing
- name: Python
  role: Runtime for the ciclops CLI (python ciclops) used to execute the pipeline
    per model type
- name: pip
  role: Installation route for the ciclops package (pip install ciclops)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.101583
    title: Protocol for using Ciclops to build models trained on cross-platform transcriptome
      data for clinical outcome prediction
  source_content_hash: sha256:e401cf2317f96863c42a8aad38783e988bdb1424ea8e11ecd8c2d7df1a70202b
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# machine-learning-model-selection-and-hyperparameter-application

## Summary

Runs the Ciclops pipeline to train and compare five machine learning models (LightGBM, XGBoost, Random Forest, Gaussian Process Regression, Linear Regression) on cross-platform transcriptome data, using 10-fold cross-validation on a training set (e.g., GSE151189) and transfer testing on a validation set (e.g., GSE59098).

## When to use

Use when building clinical outcome prediction models from cross-platform transcriptome datasets (RNA-seq trained, microarray transferred), when comparing multiple ML model types, or when needing quantile-normalized transfer learning between platforms.

## When NOT to use

- The training and validation sets share no common genes — the program fails silently, producing ./training csv splits with only sample-name and label columns and no gene expression columns
- Only SHAP biomarker analysis is needed with a regression model (gpr or lr) — SHAP is implemented only for LightGBM, XGBoost, and Random Forest
- The data are already normalized or the analysis goal does not involve cross-platform transfer, since quantile normalization and platform transfer are core, default behaviors of this pipeline

## Inputs

- Training data .csv (sample names in first column, labels in last column), e.g., GSE151189 RNA-seq
- Transfer-validation data .csv (sample names in first column, labels in last column), e.g., GSE59098 microarray
- Choice of model type via -m flag: lgb, xgb, rf, gpr, or lr (default lgb)

## Outputs

- ./training/ — preprocessed training datasets for model training and 10-fold cross validation
- ./validation/ — validation dataset for transferring test
- ./params/ — trained machine learning model parameters
- ./performance/ — model performance in 10-fold cross validation and transferring test, reported as 95% bootstrap confidence intervals

## How to apply

Format the training and transfer-validation data as .csv files with sample names in the first column and labels in the last column (e.g., GSE151189 as training, GSE59098 as validation). Run the Ciclops Python pipeline once per model type via the -m flag — lgb (default), xgb, rf, gpr, lr — keeping quantile normalization enabled by omitting --no_quantile. The pipeline preprocesses the training data and performs 10-fold cross validation (writing preprocessed splits to ./training/), then transfers the trained models to the validation set (saving parameters to ./params/). Collect metrics from ./performance/, which reports cross-validation and transferring-test results as 95% confidence intervals obtained by bootstrapping. Hyperparameters are not fixed by the tool; the protocol advises tuning them for a better fit, and users needing other algorithms must modify the source code.

## Related tools

- **Ciclops** (Pipeline that trains clinical outcome prediction models with 10-fold cross-validation and transfers them to a second transcriptomic platform, writing performance metrics to ./performance/) — https://github.com/GuanLab/ciclops.git
- **LightGBM** (Default model type (lgb) selectable via the -m flag for cross-validated training and transfer testing)
- **XGBoost** (Alternative model type (xgb) selectable via the -m flag for cross-validated training and transfer testing)
- **Python** (Runtime for the ciclops CLI (python ciclops) used to execute the pipeline per model type)
- **pip** (Installation route for the ciclops package (pip install ciclops))

## Examples

```
python ciclops --train_path GSE151189.csv --valid_path GSE59098.csv -m lgb
```

## Evaluation signals

- ./performance/ contains results for all five model types (lgb, xgb, rf, gpr, lr) with valid (non-NaN) evaluation metrics for both the 10-fold cross-validation and the transfer test on GSE59098
- No anomalous results such as confidence intervals spanning [0, 1] for AUPRC are present in the performance outputs
- 95% confidence intervals in ./performance/ are produced by bootstrapping and exclude the trivial bounds (0 and 1) for AUPRC/AUROC
- ./training/ contains preprocessed cross-validation splits with gene expression columns, confirming the training and validation sets shared common genes
- ./params/ contains saved model parameters for each of the five model types after the transfer step

## Limitations

- The program fails silently when training and test sets share no common genes, producing csv splits in ./training with only sample names and labels and no gene expression columns
- Results can be unreliable without automatic safeguards — e.g., confidence intervals spanning [0, 1] for AUPRC, or NaN values for any of the evaluation metrics
- Users are limited in their choice of machine learning algorithms and must modify the source code to use their own; SHAP analysis is not implemented for the regression models (Gaussian Process Regression and Linear Regression)
- No fixed hyperparameters are guaranteed; users must tune hyperparameters themselves for a better fit

## Evidence

- [readme] optional arguments include model types and flags: "-m MODEL_TYPE, --model_type MODEL_TYPE Machine learning models to use: lgb: LightGBM; xgb: XGBoost; rf: Random Forest;"
- [readme] performance folder contents: "`./performance/`: model performance in 10-fold cross validation and transferring test"
- [methods] training folder contents: "preprocessed training datasets for model training and 10-fold cross validation"
- [discussion] anomalous results safeguard discussion: "confidence intervals spanning [0, 1] for AUPRC, or NaN values for any of the evaluation metrics"
- [abstract] cross-platform purpose: "freely available software that can build explainable models to deliver across cross-platform datasets for predicting clinical outcomes"
