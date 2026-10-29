---
name: ten-fold-cross-validation-assessment
description: Use when building clinical outcome prediction models on a cross-platform
  transcriptome training set and you need 10-fold cross-validation metrics plus transfer-test
  evaluation on an external validation dataset.
when_to_use_negative:
- The training and validation .csv files share no common genes — Ciclops fails silently,
  producing ./training splits with only sample-name and label columns and missing
  gene expression columns
- SHAP analysis is the goal with regression models (gpr, lr) — SHAP is only implemented
  for LightGBM, XGBoost, and Random Forest
- You need a guaranteed-default hyperparameter configuration — hyperparameters are
  not fixed and must be tuned by the user for a better fit
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: Ciclops
  role: Pipeline that preprocesses training data, performs 10-fold cross validation,
    transfers trained models to the validation set, and writes performance metrics
  repo: https://github.com/GuanLab/ciclops.git
- name: LightGBM
  role: Default model type (lgb) trained via cross-validation and transferred to the
    test set
- name: XGBoost
  role: Alternative model type (xgb) selectable via -m for cross-validated training
    and transfer evaluation
- name: pip
  role: Installation channel for the ciclops package (pip install ciclops)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.101583
    title: Protocol for using Ciclops to build models trained on cross-platform transcriptome
      data for clinical outcome prediction
  source_content_hash: sha256:1c8d30f95901c1ad304b70af7af1b75bfc466cd7cd585d2220f208f6dca3b9fe
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# ten-fold-cross-validation-assessment

## Summary

Run Ciclops' 10-fold cross-validation on a training transcriptome dataset (e.g., GSE151189 RNA-seq) and evaluate the trained models on an external cross-platform test set (e.g., GSE59098 microarray), collecting per-fold and bootstrapped 95% CI performance metrics from ./performance/.

## When to use

Use when building clinical outcome prediction models on a cross-platform transcriptome training set and you need 10-fold cross-validation metrics plus transfer-test evaluation on an external validation dataset.

## When NOT to use

- The training and validation .csv files share no common genes — Ciclops fails silently, producing ./training splits with only sample-name and label columns and missing gene expression columns
- SHAP analysis is the goal with regression models (gpr, lr) — SHAP is only implemented for LightGBM, XGBoost, and Random Forest
- You need a guaranteed-default hyperparameter configuration — hyperparameters are not fixed and must be tuned by the user for a better fit

## Inputs

- Training dataset as .csv (e.g., GSE151189) with sample names in the first column and labels in the last column
- Transfer validation dataset as .csv (e.g., GSE59098) with sample names in the first column and labels in the last column
- Model type flag: lgb, xgb, rf, gpr, or lr (default lgb)

## Outputs

- ./training/: preprocessed training dataset splits for model training and 10-fold cross validation
- ./validation/: validation dataset for transferring test
- ./params/: trained machine learning model parameters
- ./performance/: model performance metrics for 10-fold cross validation and transferring test, reported as 95% bootstrap confidence intervals

## How to apply

Format the training data (e.g., GSE151189) and transfer-validation data (e.g., GSE59098) as .csv files with sample names in the first column and labels in the last column. Run the Ciclops pipeline with --train_path and --valid_path, selecting one model type per run via -m: lgb (default), xgb, rf, gpr, or lr; keep quantile normalization enabled unless there is a specific reason not to (it is on by default; pass --no_quantile to disable). The pipeline preprocesses the training set, performs 10-fold cross validation writing preprocessed splits to ./training/, transfers the trained models to the validation set saving model parameters to ./params/, and writes cross-validation and transfer-test metrics to ./performance/. Results are reported as 95% confidence intervals obtained by bootstrapping. Verify that ./performance/ contains results for all five model types with valid (non-NaN) metrics for both the 10-fold cross-validation and the transfer test.

## Related tools

- **Ciclops** (Pipeline that preprocesses training data, performs 10-fold cross validation, transfers trained models to the validation set, and writes performance metrics) — https://github.com/GuanLab/ciclops.git
- **LightGBM** (Default model type (lgb) trained via cross-validation and transferred to the test set)
- **XGBoost** (Alternative model type (xgb) selectable via -m for cross-validated training and transfer evaluation)
- **pip** (Installation channel for the ciclops package (pip install ciclops))

## Examples

```
python ciclops --train_path GSE151189.csv --valid_path GSE59098.csv -m lgb
```

## Evaluation signals

- ./performance/ contains results for all five model types (lgb, xgb, rf, gpr, lr) after running each once
- Evaluation metrics for both the 10-fold cross-validation and the transfer test on GSE59098 are valid (non-NaN)
- No anomalous results such as confidence intervals spanning [0, 1] for AUPRC
- 95% confidence intervals are derived from bootstrapping, not point estimates alone
- Preprocessed cross-validation splits in ./training/ contain gene expression columns (not just sample names and labels), confirming common genes exist between platforms

## Limitations

- Program fails silently when training and test sets share no common genes, producing incomplete csv splits in ./training with only sample names and labels
- Results can be unreliable (e.g., confidence intervals spanning [0, 1] for AUPRC, or NaN values for evaluation metrics) without automatic safeguards
- Model performance varies across datasets and conditions for unexplained reasons (data collection methods, environmental factors impacting gene expression), requiring careful research design
- No fixed hyperparameters are guaranteed; users must tune hyperparameters themselves for a better fit
- Limited choice of machine learning algorithms; users must modify the source code to use their own algorithms

## Evidence

- [readme] It will generate the following folders: "`./performance/`: model performance in 10-fold cross validation and transferring test"
- [readme] README usage arguments: "Path to your training data, in .csv format; includes sample names as first column and labels as last column"
- [readme] quantile normalization default: "--no_quantile         If specified, do not use quantile normalization."
- [results] evaluation outputs finding: [evidence span withheld — anchor status: not_found]
- [discussion] silent failure mode: "the csv files containing the train/test splits in the ./training directory only have two columns: sample names and labels, missing any gene expression columns"
