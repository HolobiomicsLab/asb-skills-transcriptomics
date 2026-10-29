---
name: clinical-outcome-prediction-modeling
description: Use when training data and external validation data come from different
  transcriptomic platforms (e.g., RNA-seq vs. microarray GEO datasets) and the goal
  is a clinical outcome prediction model that transfers across platforms.
when_to_use_negative:
- The training and validation sets share no common genes — Ciclops fails silently,
  producing train/test split csvs in ./training with only two columns (sample names
  and labels) and no gene expression columns
- SHAP biomarker interpretation is needed for the regression models (Gaussian Process
  Regression or Linear Regression) — SHAP analysis is not implemented for those models
- Datasets were already jointly normalized during preprocessing — consider the --no_quantile
  flag rather than applying quantile normalization again
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: Ciclops
  role: Main pipeline for cross-platform training in clinical outcome predictions;
    installs via pip and runs model training, 10-fold cross validation, and transfer
    learning
  repo: https://github.com/GuanLab/ciclops.git
- name: LightGBM
  role: Default machine learning model type (lgb) used by Ciclops for outcome prediction
- name: XGBoost
  role: Alternative tree-based model type (xgb) selectable via -m in Ciclops
- name: SHAP
  role: Post-training analysis (via --shap, only for lgb/xgb/rf) to assess potential
    biomarkers and compare top genes between training and validation sets
- name: pip
  role: Package installer used to install the ciclops package
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.101583
    title: Protocol for using Ciclops to build models trained on cross-platform transcriptome
      data for clinical outcome prediction
  source_content_hash: sha256:287ad2d3a1aa7c3eeb10778a13e717df6c1c6cda9e0a55e2f5b9d8c74722519b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GuanLab/ciclops.git
  tools:
  - Ciclops
  - LightGBM
  - Python
  - SHAP
  - XGBoost
  - pip
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# clinical-outcome-prediction-modeling

## Summary

Build cross-platform clinical outcome prediction models with Ciclops, which performs transfer learning from one transcriptomic platform's samples to another (e.g., training on RNA-seq GSE151189 and testing on microarray GSE59098) using LightGBM as the default model with quantile normalization enabled.

## When to use

Use when training data and external validation data come from different transcriptomic platforms (e.g., RNA-seq vs. microarray GEO datasets) and the goal is a clinical outcome prediction model that transfers across platforms.

## When NOT to use

- The training and validation sets share no common genes — Ciclops fails silently, producing train/test split csvs in ./training with only two columns (sample names and labels) and no gene expression columns
- SHAP biomarker interpretation is needed for the regression models (Gaussian Process Regression or Linear Regression) — SHAP analysis is not implemented for those models
- Datasets were already jointly normalized during preprocessing — consider the --no_quantile flag rather than applying quantile normalization again

## Inputs

- Training dataset .csv with sample names as first column and labels as last column (e.g., GSE151189)
- Transfer validation dataset .csv with sample names as first column and labels as last column (e.g., GSE59098)
- Model type flag (-m): lgb, xgb, rf, gpr, or lr

## Outputs

- ./training/: preprocessed training datasets for model training and 10-fold cross validation
- ./validation/: validation dataset for transferring test
- ./params/: trained machine learning model parameters
- ./performance/: model performance in 10-fold cross validation and transferring test, reported as 95% confidence intervals by bootstrapping
- ./SHAP/: SHAP analysis results (when --shap is specified)

## How to apply

Format each dataset as a .csv with sample names in the first column and labels in the last column. Install via 'pip install ciclops' or clone GuanLab/ciclops, then run 'python ciclops' with --train_path pointing to the training csv and --valid_path to the transfer validation csv. Select the model with -m from {lgb (default), xgb, rf, gpr, lr} and keep quantile normalization enabled (do not pass --no_quantile) since it was key to cross-platform success in the original study. The pipeline preprocesses the training data, performs 10-fold cross validation (writing preprocessed splits to ./training/), transfers trained models to the validation set (saving parameters to ./params/), and reports performance in ./performance/ as metrics for both cross-validation and transferring test, with 95% confidence intervals obtained by bootstrapping. Optionally add --shap (only for lgb, xgb, rf) with -n TOP_GENES (default 20) to conduct SHAP analysis and compare top genes between training and validation sets.

## Related tools

- **Ciclops** (Main pipeline for cross-platform training in clinical outcome predictions; installs via pip and runs model training, 10-fold cross validation, and transfer learning) — https://github.com/GuanLab/ciclops.git
- **LightGBM** (Default machine learning model type (lgb) used by Ciclops for outcome prediction)
- **XGBoost** (Alternative tree-based model type (xgb) selectable via -m in Ciclops)
- **SHAP** (Post-training analysis (via --shap, only for lgb/xgb/rf) to assess potential biomarkers and compare top genes between training and validation sets)
- **pip** (Package installer used to install the ciclops package)

## Examples

```
python ciclops --train_path GSE151189.csv --valid_path GSE59098.csv -m lgb
```

## Evaluation signals

- ./performance/ contains results for all five model types (lgb, xgb, rf, gpr, lr) with valid (non-NaN) evaluation metrics for both 10-fold cross-validation and the transfer test
- No anomalous results such as confidence intervals spanning [0, 1] for AUPRC
- ./training/ contains preprocessed train/test split csvs with sample names, labels, AND gene expression columns (not just two columns, which signals silent failure)
- ./params/ contains trained model parameters for the transferred models
- Comparison runs with and without --no_quantile yield matched metric rows for paired delta computation; NaN metrics indicate an unreliable run

## Limitations

- Ciclops failed to install due to uninstalled dependencies (step 1 troubleshooting)
- SHAP analysis works best and fastest with tree-based models and is not implemented for the regression models (Gaussian Process Regression and Linear Regression)
- Users are limited in their choice of machine learning algorithms and must modify the source code to fit their needs
- Disparities in model performance may be due to the nature of the data collection methods and environmental factors drastically impacting gene expression
- Results can be unreliable without automatic safeguards, e.g., confidence intervals spanning [0, 1] for AUPRC or NaN values for evaluation metrics
- No changelog or fixed hyperparameters; users must tune hyperparameters themselves for a better fit

## Evidence

- [intro] Cross-platform training In CLinical Outcome PredictionS (Ciclops): "**Cross-platform training In CLinical Outcome PredictionS (Ciclops)** is the winning algorithm in [2019 Malaria DREAM Challenge SubChallenge 2]"
- [intro] Ciclops performs transfer learning: "Ciclops performs transfer learning from one transcriptomic platform's samples to another."
- [readme] README usage and CLI arguments: "python ciclops [-h] [--train_path TRAIN_PATH] [--valid_path VALID_PATH] [-m MODEL_TYPE] [--no_quantile] [--shap] [-n TOP_GENES]"
- [readme] Output folders: "`./performance/`: model performance in 10-fold cross validation and transferring test"
- [methods] Quantile normalization flag: "--no_quantile         If specified, do not use quantile normalization."
- [discussion] SHAP limitation: "because SHAP analysis works best and fastest with tree-based models, it is not implemented for use with the regression models"
- [discussion] Silent failure mode: "the csv files containing the train/test splits in the ./training directory only have two columns: sample names and labels, missing any gene expression columns"
- [discussion] Unreliable results without safeguards: "confidence intervals spanning [0, 1] for AUPRC, or NaN values for any of the evaluation metrics"
