---
name: knn-cross-validation-classification
description: Use when you must benchmark the k-NN (k = 5) DEG-feature classifier's
  malignant vs non-malignant performance (accuracy, sensitivity, specificity, balanced
  accuracy, AUROC) within the five cancer scRNA-seq training datasets under 10-fold
  cross-validation.
when_to_use_negative:
- Predicting labels for a new (test) single-cell dataset rather than measuring training-set
  performance — use the standard majority-vote PreCanCell_classifier() prediction
  workflow instead.
- Input is raw unnormalized counts or a matrix without gene rownames — PreCanCell_data()
  requires a normalized expression matrix (any library-depth normalization such as
  TPM or CPM) with rownames being genes and colnames being cells.
- The goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial)
  — the algorithm is limited to annotating cancer vs non-cancer cells.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: PreCanCell
  role: R package whose PreCanCell_data() preprocessing (matched DEG selection plus
    [0,1] min-max scaling) and PreCanCell_classifier() k-NN (k = 5) pipeline with
    parallel backends execute the 10-fold cross-validation on each training dataset
  repo: https://github.com/WangX-Lab/PreCanCell
- name: fastknn
  role: k-NN implementation underlying the PreCanCell classifier (k = 5), installed
    via devtools::install_github before PreCanCell
  repo: davpinto/fastknn
- name: Seurat
  role: Required (version >= 4.3.0) for data preprocessing of the normalized expression
    matrices prior to cross-validated classification
- name: parallel
  role: R parallel backend (with doParallel/foreach) engaged by setting cores > 1
    to speed up cross-validation rounds on each training dataset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:a045b0c85789f6ca0df7f47a48d4aabde3e170e41b7b25205d4b1ee63698a0e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - PreCanCell
  - R
  - Seurat
  - fastknn
  - parallel
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# knn-cross-validation-classification

## Summary

Cross-validates PreCanCell's k-NN (k = 5) malignant vs non-malignant classifier within each of the five cancer training scRNA-seq datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC), using the shared DEGs (73 TMGs and 186 NMGs) as [0,1]-min-max-scaled features. Reports 10-fold cross-validated accuracy, sensitivity, specificity, balanced accuracy, and AUROC, all expected to exceed 0.90 per training dataset.

## When to use

Use when you must benchmark the k-NN (k = 5) DEG-feature classifier's malignant vs non-malignant performance (accuracy, sensitivity, specificity, balanced accuracy, AUROC) within the five cancer scRNA-seq training datasets under 10-fold cross-validation.

## When NOT to use

- Predicting labels for a new (test) single-cell dataset rather than measuring training-set performance — use the standard majority-vote PreCanCell_classifier() prediction workflow instead.
- Input is raw unnormalized counts or a matrix without gene rownames — PreCanCell_data() requires a normalized expression matrix (any library-depth normalization such as TPM or CPM) with rownames being genes and colnames being cells.
- The goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial) — the algorithm is limited to annotating cancer vs non-cancer cells.

## Inputs

- Normalized scRNA-seq expression matrices (TPM/CPM, genes as rownames, cells as colnames) for the five training datasets: T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC
- Shared DEG feature set (73 TMGs and 186 NMGs) identified between malignant and non-malignant cells across the five cancer datasets
- Ground-truth malignant/non-malignant cell labels for each training dataset, used to score held-out folds

## Outputs

- cv_metrics_training_sets.csv: one row per training dataset (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) and one column per metric (accuracy, sensitivity, specificity, balanced accuracy, AUROC)
- Per-fold held-out k-NN (k = 5) predictions and labels for each cross-validation round of each training dataset

## How to apply

Load each training-set scRNA-seq expression matrix (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) and pass it through PreCanCell_data(), which restricts the matrix to the matched DEG features (73 TMGs and 186 NMGs) and min-max scales gene expression values to [0,1]; the input must be a normalized expression matrix (e.g. TPM or CPM) with rownames as genes and colnames as cells. In R, run the PreCanCell_classifier() workflow (built on the fastknn k-NN implementation) with k = 5 on each training dataset under 10-fold cross-validation, setting cores > 1 to engage the parallel/doParallel/foreach backends for execution speed. On the held-out fold of each cross-validation round, compute accuracy, sensitivity, specificity, and AUROC, with balanced accuracy defined as the average of sensitivity and specificity. Aggregate the per-dataset cross-validated metrics into a single table with one row per training dataset and one column per metric (cv_metrics_training_sets.csv). Apply the acceptance criterion: all five metrics must be greater than 0.90 in each of the five training datasets; flag any row where a metric falls at or below 0.90 as failing before writing the output. Interpret results with caution, since the training sets were involved in DEG feature selection and training-set performance may be overestimated.

## Related tools

- **PreCanCell** (R package whose PreCanCell_data() preprocessing (matched DEG selection plus [0,1] min-max scaling) and PreCanCell_classifier() k-NN (k = 5) pipeline with parallel backends execute the 10-fold cross-validation on each training dataset) — https://github.com/WangX-Lab/PreCanCell
- **fastknn** (k-NN implementation underlying the PreCanCell classifier (k = 5), installed via devtools::install_github before PreCanCell) — davpinto/fastknn
- **Seurat** (Required (version >= 4.3.0) for data preprocessing of the normalized expression matrices prior to cross-validated classification)
- **parallel** (R parallel backend (with doParallel/foreach) engaged by setting cores > 1 to speed up cross-validation rounds on each training dataset)

## Examples

```
testdata <- PreCanCell_data(input); results <- PreCanCell_classifier(testdata, cores = 2)
```

## Evaluation signals

- Every training-dataset row (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) must show accuracy, sensitivity, specificity, balanced accuracy, and AUROC all > 0.90; flag any row where a metric falls at or below 0.90 as failing the acceptance criterion.
- Balanced accuracy must equal the average of sensitivity and specificity for each row of the metrics table.
- The output table must contain exactly one row per training dataset and one column per metric before writing cv_metrics_training_sets.csv.
- Features must be restricted to the DEG set (73 TMGs + 186 NMGs) and the preprocessed matrix output by PreCanCell_data() must have values min-max scaled to the range [0,1].
- Metrics must be computed on the held-out fold of each of the 10 cross-validation rounds per dataset, not on the full training matrix.

## Limitations

- Training-set cross-validated performance may be overestimated because the predicted (training) sets were involved in DEG feature selection.
- The algorithm is limited to annotating cancer vs non-cancer cells; it does not identify subpopulations of non-cancer cells (immune, stromal, epithelial).
- No version number is stated for the PreCanCell R package.
- PreCanCell_classifier() exposes only testdata and cores parameters, so per-fold CV metric aggregation must be arranged around the default prediction workflow.

## Evidence

- [readme] With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5).: "With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5)."
- [readme] The function `PreCanCell_data()` is used to data preprocessing. Its input should be normalized expression matrix with rownames being genes and colnames being cells.: "The function `PreCanCell_data()` is used to data preprocessing. Its input should be normalized expression matrix with rownames being genes and colnames being cells."
- [readme] `cores` represents the number of cores to use for parallel execution.: "`cores` represents the number of cores to use for parallel execution."
- [methods] We then using the `PreCanCell_data()` function for data preprocessing that will select matched genes and scale gene expression values.: "We then using the `PreCanCell_data()` function for data preprocessing that will select matched genes and scale gene expression values."
- [methods] running `PreCanCell_classifier()` function to predict malignant and non-malignant cells: "running `PreCanCell_classifier()` function to predict malignant and non-malignant cells"
- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [abstract] In all these datasets, PreCanCell achieved above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC: "In all these datasets, PreCanCell achieved above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC"
- [discussion] the reported predictive performance in the training set could be overestimated due to the predicted sets being involved in feature selection: "the reported predictive performance in the training set could be overestimated due to the predicted sets being involved in feature selection"
