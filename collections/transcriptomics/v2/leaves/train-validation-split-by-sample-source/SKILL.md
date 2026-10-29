---
name: train-validation-split-by-sample-source
description: Use when training a supervised malignancy or cell-type classifier on
  multi-sample scRNA-seq data and held-out performance must reflect generalization
  to unseen patients/samples rather than unseen cells from already-seen samples.
when_to_use_negative:
- The dataset contains cells from a single sample or lacks sample-source metadata,
  so a source-disjoint split is impossible and leakage-avoidance by sample cannot
  be implemented
- The goal is to annotate cells with an already-validated pretrained model (e.g. scCancer2's
  predCellType/runCellSubtypeClassify on new tumor data) rather than to benchmark
  classifier generalization
- The task is cross-dataset label similarity quantification (similarityCalculation
  between reference annotations), which does not involve a supervised malignant/nonMalignant
  training split
tools:
- name: xgboost
  role: Trains the top-performing malignant vs nonMalignant classifier on the training
    sample split and predicts held-out samples
  repo: https://github.com/czythu/scCancer
- name: scikit-learn
  role: Trains the ElasticNet logistic regression and Neural Network comparison classifiers
    on the same training split
- name: scCancer2
  role: Provides the malignancy module, reference construction from TCGA bulk + scRNA-seq
    data, and the pretrained model pipeline being benchmarked
  repo: http://lifeome.net/software/sccancer2/
- name: Seurat
  role: Loads and formats the scRNA-seq expression matrix used as classifier input
    alongside the xgboost library in R
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:ec4a4bd1d4b550cf9d0201f8fea64cf5070382ac149984c3f65ffd0c85127383
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# train-validation-split-by-sample-source

## Summary

Partitions a scRNA-seq malignancy reference dataset into training and validation sets by sample source (not by cell) before training classifiers such as XGBoost, ElasticNet, and Neural Networks, preventing cross-sample leakage when evaluating malignant vs nonMalignant prediction.

## When to use

Use when training a supervised malignancy or cell-type classifier on multi-sample scRNA-seq data and held-out performance must reflect generalization to unseen patients/samples rather than unseen cells from already-seen samples.

## When NOT to use

- The dataset contains cells from a single sample or lacks sample-source metadata, so a source-disjoint split is impossible and leakage-avoidance by sample cannot be implemented
- The goal is to annotate cells with an already-validated pretrained model (e.g. scCancer2's predCellType/runCellSubtypeClassify on new tumor data) rather than to benchmark classifier generalization
- The task is cross-dataset label similarity quantification (similarityCalculation between reference annotations), which does not involve a supervised malignant/nonMalignant training split

## Inputs

- Constructed malignancy reference matrix (466,468 cells x 2,756 genes) with malignant/nonMalignant labels and per-cell sample-source identifiers
- TCGA-sorted gene list file genes-scRNA-tcga-sorted.txt (2,756 genes)
- Trained classifier objects: xgboost model, scikit-learn ElasticNet logistic regression, and Neural Network

## Outputs

- Held-out (validation-split) confusion matrix, accuracy, and AUC for XGBoost, ElasticNet, and Neural Network
- classifier_comparison.csv summarizing accuracy and AUC (values in [0,1]) for the three classifiers

## How to apply

Load the constructed malignancy reference dataset (466,468 cells x 2,756 genes, built by combining pan-cancer TCGA bulk data and scRNA-seq data across 10 cancer types, 175 samples, 663,857 cells) and assign each cell to a split based on its sample-source identifier rather than randomly at the cell level. Restrict features to the 2,756 TCGA-sorted genes from genes-scRNA-tcga-sorted.txt, zero-filling any gene absent from the expression matrix so all models see a consistent feature space. Train XGBoost (via the R xgboost library alongside Seurat) plus scikit-learn ElasticNet logistic regression and a Neural Network on the training split to predict malignant vs nonMalignant labels, then predict on the entirely sample-disjoint validation split. Compute confusion matrix, accuracy, and AUC per classifier, apply the 0.5 probability threshold for the xgboost malignant calls, and confirm XGBoost attains the highest accuracy/AUC as reported in Fig. 4A. The rationale is that splitting by cell alone lets near-duplicate profiles from the same tumor appear in both splits, inflating metrics and masking true cross-patient generalization.

## Related tools

- **xgboost** (Trains the top-performing malignant vs nonMalignant classifier on the training sample split and predicts held-out samples) — https://github.com/czythu/scCancer
- **scikit-learn** (Trains the ElasticNet logistic regression and Neural Network comparison classifiers on the same training split)
- **scCancer2** (Provides the malignancy module, reference construction from TCGA bulk + scRNA-seq data, and the pretrained model pipeline being benchmarked) — http://lifeome.net/software/sccancer2/
- **Seurat** (Loads and formats the scRNA-seq expression matrix used as classifier input alongside the xgboost library in R)

## Examples

```
suppressMessages(library(Seurat)); suppressMessages(library(xgboost)); predict.label <- predict(model.ref, testdata); predict.label[which(predict.label > 0.5)] <- "malignant"
```

## Evaluation signals

- classifier_comparison.csv contains accuracy and AUC for all three classifiers (XGBoost, ElasticNet, Neural Network), each metric falling in [0, 1]
- XGBoost achieves the highest accuracy and AUC among the three classifiers, matching the relative ranking reported in Fig. 4A / Section 3.4
- No sample source appears in both the training and validation splits (verifiable from per-cell sample-source labels), confirming absence of cross-sample leakage
- Every cell in the validation predictions has a feature vector of exactly 2,756 TCGA-sorted genes, with zero-filling applied to genes absent from the expression matrix
- xgboost malignant calls reproduce the threshold rule: predictions > 0.5 relabeled as 'malignant', all others nonMalignant

## Limitations

- No software version numbers are stated for R, scCancer, Seurat, or xgboost, so exact reproducibility of the classifier comparison is not guaranteed
- Robust performance was demonstrated when no internal confidential reference cells are available; when high-quality matched internal reference cells exist, a malignancy-estimation approach within the same sample may be preferable
- The feature space is fixed at the 2,756 TCGA-sorted genes; datasets with poor overlap against this gene list will accumulate many zero-filled features, degrading classifier signal

## Evidence

- [methods] TCGA-sorted gene list as feature space: "genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")"
- [methods] xgboost used with Seurat for classifier training: "suppressMessages(library(Seurat))
suppressMessages(library(xgboost))"
- [readme] README confirms trained malignant-identification model: [evidence span withheld — anchor status: external_doc]
- [abstract] Robustness claim motivating held-out evaluation: "This classifier shows robust performances when no internal confidential reference cells are available."
