---
name: malignant-cell-classification
description: Use when annotating a tumor microenvironment scRNA-seq dataset where
  malignant cells must be identified but no confident internal reference malignant
  cells exist within the sample, or when testing whether the classifier generalizes
  to a cancer type (e.g. lung, gastric) held out of training.
when_to_use_negative:
- When the sample already contains an internal confidential reference population of
  malignant cells (e.g. inferCNV-style evidence within the same tumor), since the
  classifier is designed for the scenario where 'no internal confidential reference
  cells are available'
- When the input is bulk RNA-seq alone and the goal is not single-cell-level malignancy
  annotation — the classifier is built for scRNA-seq TME annotation within scCancer2's
  scAnnotation module
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scCancer2 (scCancer R package)
  role: Hosts the pretrained XGBoost malignancy model, the sorted TCGA gene list,
    and the prediction workflow within the scAnnotation module
  repo: czythu/scCancer
- name: Seurat
  role: Preprocesses scRNA-seq counts (LogNormalize with scale factor 10000, vst variable-feature
    selection up to 5000 features, ScaleData regressing nCount_RNA and mito.percent)
    before prediction
- name: xgboost
  role: R library used to run the pretrained model (predict on xgb.DMatrix at threshold
    0.5) and to retrain the classifier with held-out cancer types
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:66c254a75013e164868a94f097a7f1a1055c1978bc2d5d2e94e4a7d71e84d2e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/czythu/scCancer
  tools:
  - R
  - Seurat
  - scCancer
  - scCancer2
  - scCancer2 (scCancer R package)
  - xgboost
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# malignant-cell-classification

## Summary

scCancer2's XGBoost-based malignancy classifier labels single cells in tumor scRNA-seq data as 'malignant' or 'nonMalignant' using a reference model trained on pan-cancer TCGA bulk gene expression plus scRNA-seq data (10 cancer types, 175 samples, 663,857 cells). It applies when no internal confidential reference malignant cells are available in the sample.

## When to use

Use when annotating a tumor microenvironment scRNA-seq dataset where malignant cells must be identified but no confident internal reference malignant cells exist within the sample, or when testing whether the classifier generalizes to a cancer type (e.g. lung, gastric) held out of training.

## When NOT to use

- When the sample already contains an internal confidential reference population of malignant cells (e.g. inferCNV-style evidence within the same tumor), since the classifier is designed for the scenario where 'no internal confidential reference cells are available'
- When the input is bulk RNA-seq alone and the goal is not single-cell-level malignancy annotation — the classifier is built for scRNA-seq TME annotation within scCancer2's scAnnotation module

## Inputs

- scRNA-seq count matrix or Seurat object for the tumor sample (with per-cell nCount_RNA and mitochondrial percentage metadata)
- Pretrained XGBoost model template 'sc_xgboost.model' from the scCancer package txt directory
- Sorted gene list 'genes-scRNA-tcga-sorted.txt' from the scCancer package txt directory
- Optional: pan-cancer training matrix combining TCGA bulk gene expression and scRNA-seq data (10 cancer types, 175 samples, 663,857 cells) for retraining with held-out cancer types

## Outputs

- Per-cell probability scores from the XGBoost model
- Per-cell binary labels: 'malignant' (> 0.5) or 'nonMalignant' (<= 0.5)
- Optionally, a retrained XGBoost malignancy classifier trained on nine cancer types after holding out one cancer type (e.g. cond_lc_heldout, cond_gc_heldout)

## How to apply

Load the pretrained model template 'sc_xgboost.model' and the sorted gene list 'genes-scRNA-tcga-sorted.txt' from the scCancer package txt directory (system.file('txt', package='scCancer')). Preprocess the query counts in Seurat: NormalizeData with LogNormalize and scale.factor 10000, FindVariableFeatures with vst and nfeatures = min(5000, ...), then ScaleData with vars.to.regress = c('nCount_RNA', 'mito.percent'). Align the scaled expression matrix to the sorted gene list, filling genes absent from the sample with zeros, convert the aligned matrix to an xgb.DMatrix, and run predict(model.ref, testdata). Threshold the predicted probabilities at 0.5 so that values > 0.5 become 'malignant' and values <= 0.5 become 'nonMalignant'. To test generalization to a new cancer type, drop that cancer type's samples (e.g. lung cancer, LC) from the pan-cancer training matrix (TCGA bulk + scRNA-seq, 10 cancer types, 175 samples), retrain the XGBoost model with the xgboost R library, and rerun prediction on the held-out scRNA-seq data.

## Related tools

- **scCancer2 (scCancer R package)** (Hosts the pretrained XGBoost malignancy model, the sorted TCGA gene list, and the prediction workflow within the scAnnotation module) — czythu/scCancer
- **Seurat** (Preprocesses scRNA-seq counts (LogNormalize with scale factor 10000, vst variable-feature selection up to 5000 features, ScaleData regressing nCount_RNA and mito.percent) before prediction)
- **xgboost** (R library used to run the pretrained model (predict on xgb.DMatrix at threshold 0.5) and to retrain the classifier with held-out cancer types)

## Examples

```
testdata <- xgb.DMatrix(scale.expr[, genes.path]); predict.label <- predict(model.ref, testdata); predict.label[which(predict.label > 0.5)] <- "malignant"; predict.label[which(predict.label <= 0.5)] <- "nonMalignant"
```

## Evaluation signals

- Predicted labels must be exactly the two classes 'malignant' and 'nonMalignant', assigned by thresholding predicted probabilities at 0.5
- The query expression matrix must be dimensionally aligned to 'genes-scRNA-tcga-sorted.txt', with genes absent from a sample filled with zeros before xgb.DMatrix conversion
- Preprocessing must reproduce the article's parameters: LogNormalize with scale factor 10000, vst variable-feature selection up to 5000 features, and ScaleData regressing nCount_RNA and mito.percent
- For held-out generalization tests (lung cancer, gastric cancer), per-condition label tabulations should match the good performance reported in Fig. 4G / Supplementary Fig. S12 (cond_lc_heldout, cond_gc_heldout)
- Robustness check: prediction should remain reliable on samples with no internal confidential reference cells, per the paper's stated robustness finding

## Limitations

- No software version numbers are stated for R, scCancer, Seurat, or xgboost, so exact reproducibility of the environment is not documented
- Generalization to a cancer type depends on it being represented (or well-approximated) in the pan-cancer training set; performance on held-out LC and GC was verified empirically, but other cancer types have not been validated in this context
- The classifier combines TCGA bulk gene expression with scRNA-seq data for training, so its transferability depends on how well bulk-derived signal maps onto single-cell expression profiles

## Evidence

- [abstract] integrating large-scale pan-cancer TCGA bulk gene expression datasets and scRNA-seq datasets (10 cancer types, 175 samples, 663 857 cells): "integrating large-scale pan-cancer TCGA bulk gene expression datasets and scRNA-seq datasets (10 cancer types, 175 samples, 663 857 cells)"
- [abstract] This classifier shows robust performances when no internal confidential reference cells are available.: "This classifier shows robust performances when no internal confidential reference cells are available."
- [methods] genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt"): "genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")"
- [methods] object <- ScaleData(object, vars.to.regress = c("nCount_RNA", "mito.percent"), verbose = FALSE): "object <- ScaleData(object, vars.to.regress = c("nCount_RNA", "mito.percent"), verbose = FALSE)"
- [methods] predict.label[which(predict.label > 0.5)] <- "malignant": "predict.label[which(predict.label > 0.5)] <- "malignant""
