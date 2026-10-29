---
name: transfer-learning-across-transcriptomic-platforms
description: Use when training data and validation data come from different transcriptomic
  platforms (e.g., RNA-seq vs. microarray) and you need clinical outcome models that
  generalize across platforms.
when_to_use_negative:
- Training and validation sets share no common genes — the program fails silently
  and the ./training csv splits contain only sample names and labels, missing gene
  expression columns.
- SHAP biomarker interpretation is required with the regression models — SHAP analysis
  is not implemented for Gaussian Process Regression (gpr) or Linear Regression (lr).
- Quantile normalization between platforms is inappropriate for the data — pass --no_quantile
  rather than using the default cross-platform pipeline.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3308
tools:
- name: Ciclops
  role: Core pipeline that trains models on the training platform, performs 10-fold
    cross validation, and transfers models to the validation platform
  repo: https://github.com/GuanLab/ciclops.git
- name: Python
  role: Runtime for invoking the ciclops CLI (python ciclops with --train_path, --valid_path,
    -m flags)
- name: LightGBM
  role: Default tree-based model type (lgb) usable for cross-validation and transfer
    test
- name: XGBoost
  role: Alternative tree-based model type (xgb) usable for cross-validation and transfer
    test
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.101583
    title: Protocol for using Ciclops to build models trained on cross-platform transcriptome
      data for clinical outcome prediction
  source_content_hash: sha256:37a3e3f47679173515dc724124ec8e2890bab6c1fb3bce3000ae6e5322fe35e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GuanLab/ciclops.git
  tools:
  - Ciclops
  - LightGBM
  - Python
  - XGBoost
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# transfer-learning-across-transcriptomic-platforms

## Summary

Train clinical outcome prediction models on one transcriptomic platform (e.g., RNA-seq GSE151189) with Ciclops and transfer them to another platform (e.g., microarray GSE59098) using quantile normalization and five model types, evaluating via 10-fold cross-validation and transfer test.

## When to use

Use when training data and validation data come from different transcriptomic platforms (e.g., RNA-seq vs. microarray) and you need clinical outcome models that generalize across platforms.

## When NOT to use

- Training and validation sets share no common genes — the program fails silently and the ./training csv splits contain only sample names and labels, missing gene expression columns.
- SHAP biomarker interpretation is required with the regression models — SHAP analysis is not implemented for Gaussian Process Regression (gpr) or Linear Regression (lr).
- Quantile normalization between platforms is inappropriate for the data — pass --no_quantile rather than using the default cross-platform pipeline.

## Inputs

- Training dataset as .csv (sample names in first column, labels in last column), e.g., GSE151189 RNA-seq
- Transfer validation dataset as .csv (sample names in first column, labels in last column), e.g., GSE59098 microarray
- Model type flag: lgb, xgb, rf, gpr, or lr (default lgb)

## Outputs

- ./training/: preprocessed training dataset splits for model training and 10-fold cross validation
- ./validation/: validation dataset prepared for the transferring test
- ./params/: trained machine learning model parameters
- ./performance/: model performance in 10-fold cross validation and transferring test, reported as 95% confidence intervals via bootstrapping
- ./SHAP/: SHAP analysis results (when --shap is used with lgb, xgb, or rf)

## How to apply

Format each dataset as a .csv with sample names in the first column and labels in the last column, then run the Ciclops pipeline once per model type (-m lgb, xgb, rf, gpr, lr; default lgb), pointing --train_path at the training csv and --valid_path at the transfer validation csv, keeping quantile normalization enabled (do not pass --no_quantile). The pipeline preprocesses the training data, performs 10-fold cross validation, writes preprocessed splits to ./training/, transfers trained model parameters to the validation set (./params/), and writes cross-validation and transfer-test performance metrics to ./performance/. Judge success by confirming ./performance/ contains valid (non-NaN) metrics for all five models on both cross-validation and transfer test, with no anomalous confidence intervals spanning [0, 1] for AUPRC.

## Related tools

- **Ciclops** (Core pipeline that trains models on the training platform, performs 10-fold cross validation, and transfers models to the validation platform) — https://github.com/GuanLab/ciclops.git
- **Python** (Runtime for invoking the ciclops CLI (python ciclops with --train_path, --valid_path, -m flags))
- **LightGBM** (Default tree-based model type (lgb) usable for cross-validation and transfer test)
- **XGBoost** (Alternative tree-based model type (xgb) usable for cross-validation and transfer test)

## Examples

```
python ciclops --train_path GSE151189.csv --valid_path GSE59098.csv -m lgb
```

## Evaluation signals

- ./performance/ contains evaluation results from each 10-fold cross-validation fold for both training and test sets, reported as 95% confidence intervals obtained by bootstrapping
- Results exist for all five model types (lgb, xgb, rf, gpr, lr) with valid, non-NaN evaluation metrics
- No anomalous confidence intervals spanning [0, 1] for AUPRC appear in the performance outputs
- csv files in ./training/ contain more than sample names and labels — i.e., gene expression columns are present (no silent gene-overlap failure)
- Trained model parameters are saved under ./params/ for the transferring test

## Limitations

- SHAP analysis is not implemented for the regression models (Gaussian Process Regression and Linear Regression) due to computational cost
- Users are limited in their choice of machine learning algorithms and must modify the source code to use their own
- The program fails silently when training and test sets share no common genes, producing csv splits with only sample names and labels
- Results can be unreliable without safeguards: confidence intervals spanning [0, 1] for AUPRC or NaN evaluation metrics can occur
- No fixed hyperparameters are guaranteed; users must tune hyperparameters themselves for a better fit
- No software version or changelog is provided for Ciclops

## Evidence

- [readme] Pipeline for building clinical outcome prediction models on training dataset and transfer learning on validation datasets.: "Pipeline for building clinical outcome prediction models on training dataset and transfer learning on validation datasets."
- [methods] 10-fold cross validation outputs: "preprocessed training datasets for model training and 10-fold cross validation"
- [intro] transfer learning from one platform to another: "Ciclops performs transfer learning from one transcriptomic platform's samples to another."
- [discussion] failure mode: missing gene columns: "the csv files containing the train/test splits in the ./training directory only have two columns: sample names and labels, missing any gene expression columns"
- [results] task card finding on performance outputs: [evidence span withheld — anchor status: not_found]
