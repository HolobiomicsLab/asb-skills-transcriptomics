---
name: confusion-matrix-accuracy-auc-evaluation
description: Use when comparing two or more supervised classifiers on the same held-out
  test split and you need comparable metrics (confusion matrix, accuracy, AUC) to
  rank them, e.g. malignant vs nonMalignant cell prediction.
when_to_use_negative:
- When only a single trained model exists and there is no second classifier to rank
  against — a plain per-model metric report suffices and no comparison table is needed.
- When the training and validation data share samples (no sample-source split), because
  leakage would inflate accuracy/AUC and invalidate the comparison.
- For unsupervised cell clustering or annotation outputs with no ground-truth labels,
  since confusion matrix, accuracy, and AUC all require reference labels.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: xgboost
  role: Gradient-boosted classifier trained on the training split to predict malignant
    vs nonMalignant labels; its predictions are thresholded at 0.5 before metric computation
  repo: https://github.com/czythu/scCancer
- name: scikit-learn
  role: Provides the ElasticNet logistic regression and Neural Network classifiers
    trained on the same split, plus metric computation for the comparison
- name: scCancer2
  role: R package whose malignancy module supplies the reference dataset, gene list
    (genes-scRNA-tcga-sorted.txt), and pretrained xgboost model being evaluated
  repo: https://doi.org/10.5281/zenodo.10477296
- name: Seurat
  role: R framework holding the expression matrix (RNA assay data) that supplies the
    classifier feature matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:a66d5a824740d9c99bab549095387b007aa488d1926b0df2e3b3502aa0825184
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# confusion-matrix-accuracy-auc-evaluation

## Summary

Evaluates trained cell-state classifiers (XGBoost, ElasticNet, Neural Network) on a held-out validation split by computing confusion matrices, accuracy, and AUC, enabling head-to-head comparison of malignancy predictors as done in scCancer2.

## When to use

Use when comparing two or more supervised classifiers on the same held-out test split and you need comparable metrics (confusion matrix, accuracy, AUC) to rank them, e.g. malignant vs nonMalignant cell prediction.

## When NOT to use

- When only a single trained model exists and there is no second classifier to rank against — a plain per-model metric report suffices and no comparison table is needed.
- When the training and validation data share samples (no sample-source split), because leakage would inflate accuracy/AUC and invalidate the comparison.
- For unsupervised cell clustering or annotation outputs with no ground-truth labels, since confusion matrix, accuracy, and AUC all require reference labels.

## Inputs

- Constructed malignancy reference expression matrix (e.g. 466,468 cells x 2,756 genes) with malignant/nonMalignant labels
- Training/validation split partitioned by sample source
- Feature gene list genes-scRNA-tcga-sorted.txt (2,756 TCGA-sorted genes)
- Trained XGBoost, ElasticNet, and Neural Network models

## Outputs

- Per-classifier confusion matrix on the validation split
- Accuracy and AUC values per classifier (all in [0, 1])
- Summary comparison table, e.g. classifier_comparison.csv

## How to apply

Train each candidate classifier (e.g. xgboost, scikit-learn ElasticNet logistic regression, and a Neural Network) on the same training split, with features restricted to the 2,756 TCGA-sorted genes from genes-scRNA-tcga-sorted.txt and missing genes zero-filled in a zero-initialized matrix over the feature list. Split data by sample source so validation samples are distinct from training samples, preventing cross-sample leakage. Predict binary labels (malignant vs nonMalignant; xgboost probabilities thresholded at 0.5) on the held-out validation split for every model. For each model compute the confusion matrix, accuracy, and AUC, and assemble the metrics into a summary table such as classifier_comparison.csv. Confirm all metric values fall in [0, 1] and that the relative ranking matches the reported comparison (XGBoost highest, Fig. 4A / Supplementary Fig. S7).

## Related tools

- **xgboost** (Gradient-boosted classifier trained on the training split to predict malignant vs nonMalignant labels; its predictions are thresholded at 0.5 before metric computation) — https://github.com/czythu/scCancer
- **scikit-learn** (Provides the ElasticNet logistic regression and Neural Network classifiers trained on the same split, plus metric computation for the comparison)
- **scCancer2** (R package whose malignancy module supplies the reference dataset, gene list (genes-scRNA-tcga-sorted.txt), and pretrained xgboost model being evaluated) — https://doi.org/10.5281/zenodo.10477296
- **Seurat** (R framework holding the expression matrix (RNA assay data) that supplies the classifier feature matrix)

## Examples

```
predict.label <- predict(model.ref, testdata); predict.label[which(predict.label > 0.5)] <- "malignant"; caret::confusionMatrix(factor(predict.label), factor(labels)); auc <- pROC::auc(labels, prob)
```

## Evaluation signals

- classifier_comparison.csv contains accuracy and AUC rows for all three classifiers (XGBoost, ElasticNet, Neural Network).
- XGBoost achieves the highest accuracy and AUC, matching the relative ranking reported in Fig. 4A / Supplementary Fig. S7.
- All metric values fall within [0, 1].
- Confusion matrices for each classifier are computed on a validation split whose samples were not used in training (split by sample source).
- Feature matrix has exactly the 2,756 columns of genes-scRNA-tcga-sorted.txt, with genes absent from the expression matrix zero-filled.

## Limitations

- No software version numbers were stated for R, scCancer, Seurat, or xgboost, so exact reproducibility of metric values may vary across library versions.
- Metrics depend on the 0.5 probability threshold for xgboost label assignment; changing the threshold changes confusion-matrix-based accuracy (AUC is threshold-independent).
- The comparison was made on a reference dataset of 10 cancer types / 175 TCGA samples / 663,857 cells; ranking may differ for datasets far outside this pan-cancer training distribution.

## Evidence

- [methods] xgboost threshold and label assignment: "predict.label <- predict(model.ref, testdata)
predict.label[which(predict.label > 0.5)] <- "malignant""
- [methods] gene feature list from package: "genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")"
- [results] XGBoost superiority finding: [evidence span withheld — anchor status: not_found]
- [abstract] robust classifier without internal reference: "This classifier shows robust performances when no internal confidential reference cells are available."
- [readme] malignancy reference dataset construction: [evidence span withheld — anchor status: external_doc]
