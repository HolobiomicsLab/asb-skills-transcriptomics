---
name: tumor-marker-gene-feature-selection
description: 'Use when preparing malignant vs non-malignant cell classification input:
  you have a normalized scRNA-seq expression matrix (genes x cells, TPM/CPM) and need
  DEG features (TMGs/NMGs) selected and [0,1]-scaled.'
when_to_use_negative:
- Your input is raw UMI counts that have not undergone library-depth normalization
  (PreCanCell_data() expects normalized values such as TPM or CPM)
- Your goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial)
  rather than a binary cancer vs non-cancer call
- You need a feature set for cell-type classification beyond malignant vs non-malignant
  — the DEG set was selected only for this binary distinction
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: PreCanCell
  role: R package whose PreCanCell_data() function performs the DEG feature selection
    and [0,1] min-max scaling of gene expression values
  repo: https://github.com/WangX-Lab/PreCanCell
- name: Seurat
  role: Used for data preprocessing and, upstream, to identify the DEGs via NormalizeData()
    and FindMarkers()
- name: fastknn
  role: Provides the k-NN classifier (k = 5) that consumes the DEG-restricted, scaled
    feature matrix during PreCanCell_classifier()
  repo: https://github.com/davpinto/fastknn
- name: R
  role: Implementation language of the PreCanCell package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:601c948bb9455bbe59cb18e32dc8876825426b1303f8ba3b9c271830272555a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - PreCanCell
  - R
  - Seurat
  - fastknn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# tumor-marker-gene-feature-selection

## Summary

Restrict scRNA-seq expression matrices to the differentially expressed genes (73 TMGs and 186 NMGs) identified between malignant and non-malignant cells across five cancer training datasets, and min-max scale them to [0,1] via PreCanCell_data() before k-NN classification.

## When to use

Use when preparing malignant vs non-malignant cell classification input: you have a normalized scRNA-seq expression matrix (genes x cells, TPM/CPM) and need DEG features (TMGs/NMGs) selected and [0,1]-scaled.

## When NOT to use

- Your input is raw UMI counts that have not undergone library-depth normalization (PreCanCell_data() expects normalized values such as TPM or CPM)
- Your goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial) rather than a binary cancer vs non-cancer call
- You need a feature set for cell-type classification beyond malignant vs non-malignant — the DEG set was selected only for this binary distinction

## Inputs

- Normalized scRNA-seq expression matrix (rownames = genes, colnames = cells) from any library-depth normalization (e.g. TPM, CPM), readable via read.table from a tab-separated file

## Outputs

- Preprocessed expression matrix restricted to the DEG features (TMGs and NMGs) with gene expression values min-max scaled to the range [0,1], suitable as the 'testdata' argument of PreCanCell_classifier()

## How to apply

Take a normalized expression matrix (rownames = genes, colnames = cells; any library-depth normalization such as TPM or CPM is acceptable) and run the PreCanCell_data() function, which restricts genes to the matched DEG feature set — the 73 tumor marker genes (TMGs) and 186 normal marker genes (NMGs) identified via Seurat FindMarkers() with Bonferroni-adjusted Wilcoxon rank-sum P < 0.05 — and scales all retained gene expression values to [0,1] by min-max normalization. The rationale is that the DEGs, being genes consistently differentially expressed between malignant and non-malignant cells across the five cancer training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC), carry the discriminative signal for the downstream k-NN (k = 5) classifier executed by PreCanCell_classifier(). The output matrix has DEG rows and cell columns with [0,1]-scaled values and feeds directly into the classifier as the testdata parameter.

## Related tools

- **PreCanCell** (R package whose PreCanCell_data() function performs the DEG feature selection and [0,1] min-max scaling of gene expression values) — https://github.com/WangX-Lab/PreCanCell
- **Seurat** (Used for data preprocessing and, upstream, to identify the DEGs via NormalizeData() and FindMarkers())
- **fastknn** (Provides the k-NN classifier (k = 5) that consumes the DEG-restricted, scaled feature matrix during PreCanCell_classifier()) — https://github.com/davpinto/fastknn
- **R** (Implementation language of the PreCanCell package)

## Examples

```
library(PreCanCell); input <- read.table("example.txt", header=TRUE, sep="\t", row.names=1); testdata <- PreCanCell_data(input)
```

## Evaluation signals

- The output matrix's rownames are a subset of the DEG set (73 TMGs + 186 NMGs); genes outside this set must not be present
- All retained expression values fall within [0,1] after min-max scaling
- Input matrix must have rownames = genes and colnames = cells; a mismatch breaks PreCanCell_data()
- Downstream sanity check: feeding the scaled matrix into PreCanCell_classifier() with cores > 1 should yield per-cell freq_cancer/freq_non_cancer vote fractions and a pred_label of 'cancer' or 'non_cancer'
- On the five training datasets, classification with these features should achieve accuracy, sensitivity, specificity, balanced accuracy, and AUROC all greater than 0.90 under 10-fold cross-validation

## Limitations

- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection
- The algorithm is limited to annotating cancer vs non-cancer cells; it does not identify subpopulations of non-cancer cells
- The input to PreCanCell_data() must already be a normalized expression matrix; raw counts are not accepted

## Evidence

- [readme] The function `PreCanCell_data()` is used to data preprocessing. Its input should be normalized expression matrix with rownames being genes and colnames being cells.: "The function `PreCanCell_data()` is used to data preprocessing. Its input should be normalized expression matrix with rownames being genes and colnames being cells."
- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [methods] We then using the `PreCanCell_data()` function for data preprocessing that will select matched genes and scale gene expression values.: "We then using the `PreCanCell_data()` function for data preprocessing that will select matched genes and scale gene expression values."
- [readme] The input data can be any library-depth normalization (e.g. TPM, CPM).: "The input data can be any library-depth normalization (e.g. TPM, CPM)."
- [intro] With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by k-NN (k = 5): "With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5)"
