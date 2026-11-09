---
name: xgboost-classifier-training
description: Use when scRNA-seq tumor data lacks internal confidential reference malignant
  cells and cells must be labeled malignant vs nonMalignant, or when comparing XGBoost,
  ElasticNet, and neural network classifiers on gene-expression features.
when_to_use_negative:
- When internal confidential reference malignant cells are available in the dataset,
  since the classifier is designed for the no-reference scenario and other scCancer2
  annotation modules may apply
- When the input is not gene expression aligned to the 2,756-gene TCGA-sorted feature
  list (e.g., already-reduced embeddings or non-transcriptomic features)
- For non-cancer scRNA-seq cell-type annotation tasks outside malignant vs nonMalignant
  labeling — use predCellType or runCellSubtypeClassify instead
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: xgboost
  role: Trains the malignancy classifier and runs predict(model.ref, testdata) on
    xgb.DMatrix test data with a 0.5 probability threshold
- name: scCancer
  role: Provides the pretrained model template sc_xgboost.model and the genes-scRNA-tcga-sorted.txt
    feature list via system.file("txt", package = "scCancer")
  repo: https://github.com/czythu/scCancer
- name: Seurat
  role: Preprocesses held-out scRNA-seq samples (LogNormalize 10000, vst up to 5000
    features, ScaleData regressing nCount_RNA and mito.percent) before classification
- name: scikit-learn
  role: Trains the ElasticNet logistic regression and Neural Network baselines on
    the same training split for the three-classifier comparison
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:a9d16bb26e4fe3f93b913069ef078552b57c52b9b184c410a521a259be2d77c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/czythu/scCancer
  tools:
  - R
  - Seurat
  - scCancer
  - scCancer2
  - scikit-learn
  - xgboost
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# xgboost-classifier-training

## Summary

Trains an XGBoost malignancy classifier that integrates pan-cancer TCGA bulk gene expression with scRNA-seq data (10 cancer types, 175 samples, 663,857 cells) to label tumor microenvironment cells as malignant vs nonMalignant, and benchmarks it against ElasticNet logistic regression and neural network alternatives.

## When to use

Use when scRNA-seq tumor data lacks internal confidential reference malignant cells and cells must be labeled malignant vs nonMalignant, or when comparing XGBoost, ElasticNet, and neural network classifiers on gene-expression features.

## When NOT to use

- When internal confidential reference malignant cells are available in the dataset, since the classifier is designed for the no-reference scenario and other scCancer2 annotation modules may apply
- When the input is not gene expression aligned to the 2,756-gene TCGA-sorted feature list (e.g., already-reduced embeddings or non-transcriptomic features)
- For non-cancer scRNA-seq cell-type annotation tasks outside malignant vs nonMalignant labeling — use predCellType or runCellSubtypeClassify instead

## Inputs

- Pan-cancer training matrix integrating TCGA bulk gene expression and scRNA-seq data (10 cancer types, 175 samples, 663,857 cells)
- Feature list genes-scRNA-tcga-sorted.txt (2,756 TCGA-sorted genes) from the scCancer package txt directory
- Seurat-preprocessed scRNA-seq expression matrices (LogNormalize, scale factor 10000; vst variable features up to 5000; ScaleData regressing nCount_RNA and mito.percent)
- Malignant vs nonMalignant cell labels for the training split
- Optional pretrained model template sc_xgboost.model from the scCancer package txt directory

## Outputs

- Trained XGBoost malignancy classifier (xgboost model object)
- Per-cell predicted labels: 'malignant' (> 0.5) or 'nonMalignant' (<= 0.5)
- Confusion matrix, accuracy, and AUC per classifier
- classifier_comparison.csv summary table ranking XGBoost vs ElasticNet vs Neural Network

## How to apply

Build the training matrix by integrating TCGA bulk gene expression with scRNA-seq datasets across the pan-cancer reference, and restrict features to the 2,756 TCGA-sorted genes listed in genes-scRNA-tcga-sorted.txt, zero-filling any gene absent from a sample's expression matrix (as done by the zero-initialized matrix built over the feature list). Split training and validation sets by sample source to prevent cross-sample leakage, then train the xgboost classifier on the training split alongside a scikit-learn ElasticNet logistic regression and neural network for comparison. Predict on the held-out split with predict(model.ref, testdata) after converting the aligned matrix to an xgb.DMatrix, thresholding probabilities at 0.5 so values > 0.5 are 'malignant' and <= 0.5 are 'nonMalignant'. Compute confusion matrix, accuracy, and AUC per model — XGBoost achieved the best performance (Fig. 4A, Supplementary Fig. S7) — and optionally retrain with specific cancer types (e.g., lung cancer) dropped to test held-out generalization (Fig. 4G, Supplementary Fig. S12).

## Related tools

- **xgboost** (Trains the malignancy classifier and runs predict(model.ref, testdata) on xgb.DMatrix test data with a 0.5 probability threshold)
- **scCancer** (Provides the pretrained model template sc_xgboost.model and the genes-scRNA-tcga-sorted.txt feature list via system.file("txt", package = "scCancer")) — https://github.com/czythu/scCancer
- **Seurat** (Preprocesses held-out scRNA-seq samples (LogNormalize 10000, vst up to 5000 features, ScaleData regressing nCount_RNA and mito.percent) before classification)
- **scikit-learn** (Trains the ElasticNet logistic regression and Neural Network baselines on the same training split for the three-classifier comparison)

## Examples

```
suppressMessages(library(Seurat)); suppressMessages(library(xgboost)); predict.label <- predict(model.ref, testdata); predict.label[which(predict.label > 0.5)] <- "malignant"
```

## Evaluation signals

- classifier_comparison.csv must contain accuracy and AUC for all three classifiers with XGBoost achieving the highest values, matching the ranking in Fig. 4A
- All accuracy/AUC metric values must fall in [0, 1]
- Predicted labels must be strictly 'malignant' or 'nonMalignant', with the 0.5 threshold applied to xgboost probabilities
- Held-out cancer-type performance (e.g., lung cancer and gastric cancer samples dropped from training) should remain good, matching Fig. 4G / Supplementary Fig. S12
- Every row of the prediction matrix must align to the 2,756 genes of genes-scRNA-tcga-sorted.txt, with genes absent from a sample zero-filled

## Limitations

- No software version numbers are stated for R, scCancer, Seurat, or xgboost in the methods, so exact reproducibility requires package-version pinning by the practitioner
- No repository URL or accession IDs are provided for the HCC scRNA-seq training datasets (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021), complicating data acquisition
- Generalization to unseen cancer types was validated only on held-out lung cancer and gastric cancer; performance on other excluded cancer types is not established here

## Evidence

- [abstract] Pan-cancer reference data integration: "integrating large-scale pan-cancer TCGA bulk gene expression datasets and scRNA-seq datasets (10 cancer types, 175 samples, 663 857 cells)"
- [abstract] Classifier robustness without internal reference: "This classifier shows robust performances when no internal confidential reference cells are available."
- [methods] Threshold-based label assignment: "predict.label <- predict(model.ref, testdata)
predict.label[which(predict.label > 0.5)] <- "malignant""
- [methods] Feature list from package txt directory: "genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")"
- [methods] Preprocessing parameters for held-out samples: "object <- NormalizeData(object = object, normalization.method = "LogNormalize", scale.factor = 10000,"
