---
name: single-cell-malignancy-classification
description: Use when a tumor scRNA-seq expression matrix needs malignant/non-malignant
  cell labels but no confidential internal reference cells exist, or when comparing
  malignancy classifiers (XGBoost vs ElasticNet vs Neural Network).
when_to_use_negative:
- An internal confidential malignant/non-malignant reference from the same study is
  available and can be used directly for supervised annotation
- Cells have already been assigned malignancy labels by copy-number-based tools (e.g.
  inferCNV/CopyKAT) and you only need cross-validation, not classification
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: xgboost
  role: Trains the malignant-cell classifier on the TCGA-sorted gene features; the
    best-performing model (> 0.5 probability threshold) in the three-way comparison
- name: scCancer2
  role: Package that ships the pretrained malignancy model (model.ref), the TCGA gene
    list, and the malignancy annotation module
  repo: http://lifeome.net/software/sccancer2/
- name: Seurat
  role: Provides the Seurat object expression matrix (expr@assays[["RNA"]]@data) that
    is fed to the classifier for per-cell prediction
- name: scikit-learn
  role: Trains the ElasticNet logistic regression and Neural Network benchmarks used
    to compare against XGBoost (Fig. 4A)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:fab8394ed5adb8b43a3d58d2872b3f4f2d172bf1df8511cd84ed523de5a4223e
schema_version: 0.2.0
metadata:
  license_tier: open
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-malignancy-classification

## Summary

Supervised XGBoost-based classification of malignant vs non-malignant cells in tumor scRNA-seq data, using a reference built by integrating pan-cancer TCGA bulk RNA-seq and scRNA-seq data (10 cancer types, 175 samples, 663,857 cells) so malignant cells can be identified even when no internal reference cells are available.

## When to use

Use when a tumor scRNA-seq expression matrix needs malignant/non-malignant cell labels but no confidential internal reference cells exist, or when comparing malignancy classifiers (XGBoost vs ElasticNet vs Neural Network).

## When NOT to use

- An internal confidential malignant/non-malignant reference from the same study is available and can be used directly for supervised annotation
- Cells have already been assigned malignancy labels by copy-number-based tools (e.g. inferCNV/CopyKAT) and you only need cross-validation, not classification

## Inputs

- Tumor scRNA-seq expression matrix (e.g. Seurat object with RNA assay log-normalized data)
- Malignancy reference dataset (466,468 cells x 2,756 genes) combining pan-cancer TCGA bulk RNA-seq and scRNA-seq data
- Gene feature list file genes-scRNA-tcga-sorted.txt shipped with the scCancer package (system.file("txt", package = "scCancer"))

## Outputs

- Per-cell malignant vs nonMalignant labels (predict.label, threshold 0.5)
- classifier_comparison.csv summary table with accuracy and AUC for XGBoost, ElasticNet, and Neural Network
- Confusion matrices for each classifier on the held-out validation split

## How to apply

Build or load the malignancy reference dataset (466,468 cells x 2,756 genes) and partition it into training and validation sets split by sample source to prevent cross-sample leakage. Restrict features to the 2,756 TCGA-sorted genes listed in genes-scRNA-tcga-sorted.txt, zero-filling any gene absent from the expression matrix (build a zero-initialized matrix over the feature list). Train an XGBoost classifier on the training split, with ElasticNet logistic regression and a Neural Network (scikit-learn) trained on the same split for benchmarking. Predict on the held-out split and binarize the malignant probability at threshold 0.5 (predictions > 0.5 labeled 'malignant'). Compute confusion matrix, accuracy, and AUC per model; XGBoost should rank highest, as reported in Fig. 4A / Supplementary Fig. S7.

## Related tools

- **xgboost** (Trains the malignant-cell classifier on the TCGA-sorted gene features; the best-performing model (> 0.5 probability threshold) in the three-way comparison)
- **scCancer2** (Package that ships the pretrained malignancy model (model.ref), the TCGA gene list, and the malignancy annotation module) — http://lifeome.net/software/sccancer2/
- **Seurat** (Provides the Seurat object expression matrix (expr@assays[["RNA"]]@data) that is fed to the classifier for per-cell prediction)
- **scikit-learn** (Trains the ElasticNet logistic regression and Neural Network benchmarks used to compare against XGBoost (Fig. 4A))

## Examples

```
predict.label <- predict(model.ref, testdata); predict.label[which(predict.label > 0.5)] <- "malignant"
```

## Evaluation signals

- The output classifier_comparison.csv contains accuracy and AUC for all three classifiers (XGBoost, ElasticNet, Neural Network), each value in [0, 1]
- XGBoost achieves the highest accuracy and AUC, matching the relative ranking reported in Fig. 4A / Section 3.4
- Predicted labels contain only two classes ('malignant' and 'nonMalignant') after applying the 0.5 threshold
- Every gene in the 2,756-gene TCGA feature list is represented in the prediction matrix (missing genes zero-filled), so the feature dimension is exactly 2,756

## Limitations

- Performance depends on the pan-cancer reference (TCGA bulk + scRNA-seq, 10 cancer types, 175 samples); robustness is claimed specifically when no internal confidential reference cells are available
- No software version numbers are stated for R, scCancer, Seurat, or xgboost in the methods, so exact reproducibility may vary
- The probability threshold is fixed at 0.5; datasets with extreme malignant/non-malignant class imbalance may need recalibration

## Evidence

- [abstract] An XGBoost-style classifier integrating pan-cancer TCGA bulk and scRNA-seq data: [evidence span withheld — anchor status: not_found]
- [methods] Malignant vs non-malignant prediction with pretrained xgboost model: "predict.label <- predict(model.ref, testdata)
predict.label[which(predict.label > 0.5)] <- "malignant""
- [methods] TCGA gene feature list path: "genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")"
- [readme] README: reference dataset construction: [evidence span withheld — anchor status: external_doc]
- [results] Fig. 4A classifier ranking: [evidence span withheld — anchor status: not_found]
