---
name: pan-cancer-bulk-scrna-integration
description: Use when annotating malignant cells in tumor scRNA-seq data lacking an
  internal confident reference, or when testing classifier generalization to cancer
  types held out of training (e.g., lung or gastric cancer).
when_to_use_negative:
- When an internal confidential reference of malignant and non-malignant cells is
  available, since scCancer2's supervised cell-subtype annotation modules (predCellType,
  runCellSubtypeClassify) can leverage those labels directly
- When the data are non-cancer scRNA-seq or spatial-only data with no tumor/TME context,
  since the classifier is trained on tumor scRNA-seq plus TCGA bulk references
- When input is already a normalized, gene-aligned feature table with model-ready
  features, since re-running Seurat normalization and zero-filling would be redundant
tools:
- name: scCancer2
  role: Framework providing the pretrained XGBoost malignancy model, sorted gene list,
    and prediction utilities
  repo: http://lifeome.net/software/sccancer2/
- name: scCancer
  role: R package whose txt directory supplies 'sc_xgboost.model' and 'genes-scRNA-tcga-sorted.txt';
    genes.path is resolved via system.file()
  repo: czythu/scCancer
- name: Seurat
  role: 'Preprocesses held-out scRNA-seq: NormalizeData (LogNormalize, 10000), FindVariableFeatures
    (vst, 5000), ScaleData with regression on nCount_RNA and mito.percent'
- name: xgboost
  role: Trains the retrained classifier on the nine remaining cancer types and runs
    predict() on xgb.DMatrix test data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:6719993fa299dc5def446fca756ed914f9892bdc74f5f050517d2cd8984a0d12
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - scCancer
  - scCancer2
  - xgboost
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# pan-cancer-bulk-scrna-integration

## Summary

Integrates large-scale pan-cancer TCGA bulk RNA-seq with multi-dataset scRNA-seq (10 cancer types, 175 samples, 663,857 cells) into a shared feature space to train an XGBoost malignancy classifier with high generalization to unseen cancer types. It enables malignant cell identification when no internal confidential reference cells are available.

## When to use

Use when annotating malignant cells in tumor scRNA-seq data lacking an internal confident reference, or when testing classifier generalization to cancer types held out of training (e.g., lung or gastric cancer).

## When NOT to use

- When an internal confidential reference of malignant and non-malignant cells is available, since scCancer2's supervised cell-subtype annotation modules (predCellType, runCellSubtypeClassify) can leverage those labels directly
- When the data are non-cancer scRNA-seq or spatial-only data with no tumor/TME context, since the classifier is trained on tumor scRNA-seq plus TCGA bulk references
- When input is already a normalized, gene-aligned feature table with model-ready features, since re-running Seurat normalization and zero-filling would be redundant

## Inputs

- TCGA pan-cancer bulk gene expression matrix
- Tumor scRNA-seq count matrices (e.g., 10 cancer types, 175 samples, 663,857 cells)
- Pretrained model file 'sc_xgboost.model' and gene list 'genes-scRNA-tcga-sorted.txt' from the scCancer package txt directory
- Held-out scRNA-seq samples (e.g., lung cancer and gastric cancer)

## Outputs

- Trained XGBoost malignancy classifier
- Per-cell malignant probability scores
- Per-cell binary labels: 'malignant' (> 0.5) or 'nonMalignant' (<= 0.5)
- Prediction tabulations per held-out condition (e.g., cond_lc_heldout, cond_gc_heldout)

## How to apply

Load the pretrained model template and gene list from the scCancer package txt directory ('sc_xgboost.model' and 'genes-scRNA-tcga-sorted.txt'). Assemble the pan-cancer training matrix from TCGA bulk gene expression and scRNA-seq datasets, optionally dropping an entire cancer type (e.g., all LC samples) to retrain on the remaining nine types with the xgboost R library. Preprocess held-out scRNA-seq in Seurat: LogNormalize with scale factor 10000, vst variable-feature selection up to 5000 features, and ScaleData regressing nCount_RNA and mito.percent; then align the scaled matrix to the sorted gene list, filling genes absent from a sample with zeros. Convert the aligned matrix to an xgb.DMatrix, run predict(model.ref, testdata), and threshold probabilities at 0.5 so >0.5 is 'malignant' and <=0.5 is 'nonMalignant'. Rationale: combining bulk and single-cell references across many cancer types gives the model breadth, so it still performs well on held-out LC and GC samples (Fig. 4G; Supplementary Fig. S12).

## Related tools

- **scCancer2** (Framework providing the pretrained XGBoost malignancy model, sorted gene list, and prediction utilities) — http://lifeome.net/software/sccancer2/
- **scCancer** (R package whose txt directory supplies 'sc_xgboost.model' and 'genes-scRNA-tcga-sorted.txt'; genes.path is resolved via system.file()) — czythu/scCancer
- **Seurat** (Preprocesses held-out scRNA-seq: NormalizeData (LogNormalize, 10000), FindVariableFeatures (vst, 5000), ScaleData with regression on nCount_RNA and mito.percent)
- **xgboost** (Trains the retrained classifier on the nine remaining cancer types and runs predict() on xgb.DMatrix test data)

## Examples

```
predict.label <- predict(model.ref, testdata); predict.label[which(predict.label > 0.5)] <- "malignant"
```

## Evaluation signals

- Held-out lung cancer (cond_lc_heldout) and gastric cancer (cond_gc_heldout) performance matches the good results reported in Fig. 4G / Section 3.4
- Predicted labels are strictly binary: cells with probability > 0.5 are 'malignant', <= 0.5 are 'nonMalignant', with no missing labels
- The scaled expression matrix aligns exactly to the sorted gene list from 'genes-scRNA-tcga-sorted.txt', with genes absent from a sample filled with zeros
- Retrained classifier (trained on nine cancer types without LC) retains predictive accuracy comparable to the full pan-cancer model
- Prediction inputs are valid xgb.DMatrix objects and predict(model.ref, testdata) returns probabilities in [0, 1]

## Limitations

- Held-out generalization was demonstrated for lung and gastric cancer; performance on other unseen cancer types is not established
- No software version numbers are stated for R, scCancer, Seurat, or xgboost, so exact reproducibility may vary across versions
- No repository URL or accession IDs are provided for the HCC scRNA-seq source datasets (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021), complicating reassembly of the training matrix
- Genes absent from a sample must be zero-filled, which can distort predictions for deeply subsetted or panel-sequenced scRNA-seq data

## Evidence

- [abstract] Pan-cancer training set size: "integrating large-scale pan-cancer TCGA bulk gene expression datasets and scRNA-seq datasets (10 cancer types, 175 samples, 663 857 cells)"
- [readme] Classifier motivation: [evidence span withheld — anchor status: external_doc]
- [methods] Gene list loading from package: "genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")"
- [methods] Seurat normalization parameters: "object <- NormalizeData(object = object, normalization.method = "LogNormalize", scale.factor = 10000,"
- [methods] Prediction and thresholding: "predict.label <- predict(model.ref, testdata) predict.label[which(predict.label > 0.5)] <- "malignant""
- [abstract] Robustness without internal reference: "This classifier shows robust performances when no internal confidential reference cells are available."
