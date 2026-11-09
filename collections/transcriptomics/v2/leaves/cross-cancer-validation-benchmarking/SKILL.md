---
name: cross-cancer-validation-benchmarking
description: Use when you have normalized scRNA-seq expression matrices from cancer
  types disjoint from the training data (e.g., P1_BC through P11_Melanoma) and need
  to reproduce or extend reported malignant vs non-malignant prediction performance
  metrics.
when_to_use_negative:
- The goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial)
  — the article explicitly states the algorithm 'is not designed for identifying subpopulations
  of non-cancer cells'
- The test data overlap the training sets used for feature selection, which would
  overestimate performance ('the reported predictive performance in the training set
  could be overestimated due to the predicted sets being involved in feature selection')
- The input is raw (unnormalized) counts — PreCanCell_data() requires a normalized
  expression matrix with rownames being genes and colnames being cells
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: PreCanCell
  role: R package providing PreCanCell_data() for gene matching and [0,1] scaling
    and PreCanCell_classifier() for the five-model k-NN ensemble with majority vote
  repo: https://github.com/WangX-Lab/PreCanCell
- name: fastknn
  role: Backend implementing the k-NN (k=5) classifiers inside PreCanCell_classifier()
  repo: https://github.com/davpinto/fastknn
- name: R
  role: Language environment in which the benchmarking workflow and metric computation
    are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:10a9238b69f69edc54da2d44428603f21ca05dd59724c8c8182d7781d89d92bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - PreCanCell
  - R
  - fastknn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-cancer-validation-benchmarking

## Summary

Benchmark the PreCanCell ensemble k-NN classifier on independent single-cell transcriptome test sets spanning multiple cancer types, computing accuracy, sensitivity, specificity, balanced accuracy, and AUROC against published cell annotations.

## When to use

Use when you have normalized scRNA-seq expression matrices from cancer types disjoint from the training data (e.g., P1_BC through P11_Melanoma) and need to reproduce or extend reported malignant vs non-malignant prediction performance metrics.

## When NOT to use

- The goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial) — the article explicitly states the algorithm 'is not designed for identifying subpopulations of non-cancer cells'
- The test data overlap the training sets used for feature selection, which would overestimate performance ('the reported predictive performance in the training set could be overestimated due to the predicted sets being involved in feature selection')
- The input is raw (unnormalized) counts — PreCanCell_data() requires a normalized expression matrix with rownames being genes and colnames being cells

## Inputs

- 11 normalized scRNA-seq expression matrices (genes as rows, cells as columns; TPM/CPM or any library-depth normalization) for test sets P1_BC, P2_BCC, P3_HCC, P4_SyS, P5_SyS, P6_HCC, P7_GBM, P8_PDAC, P9_mCRPC, P10_GBM, P11_Melanoma
- Original publication's cell-level malignant/non-malignant annotations as ground truth for each test set
- PreCanCell R package with its bundled five training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) and 259-gene DEG panel

## Outputs

- Per-dataset metrics table (e.g., test_set_metrics.csv) with accuracy, sensitivity, specificity, balanced accuracy, and AUROC for each of the 11 test sets
- Per-cell classification results (Sample, freq_cancer, freq_non_cancer, pred_label) from PreCanCell_classifier() for each test set

## How to apply

Load each of the 11 test-set normalized expression matrices (genes as rows, cells as columns; TPM or CPM normalization is acceptable per the README). Preprocess each with PreCanCell_data(), which intersects genes with the 259-gene DEG feature panel (73 TMGs + 186 NMGs) and min-max scales expression to [0,1]. Run PreCanCell_classifier() on each preprocessed matrix: it classifies every cell as malignant or non-malignant by k-NN (k=5) using fastknn, with each of the five training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) in turn, then assigns the final label by majority vote of the five results (recording freq_cancer and freq_non_cancer per cell). Score each test set against the original publication's cell annotations using accuracy, sensitivity, specificity, balanced accuracy (mean of sensitivity and specificity), and AUROC, then assemble the per-dataset metrics into a single table (e.g., test_set_metrics.csv).

## Related tools

- **PreCanCell** (R package providing PreCanCell_data() for gene matching and [0,1] scaling and PreCanCell_classifier() for the five-model k-NN ensemble with majority vote) — https://github.com/WangX-Lab/PreCanCell
- **fastknn** (Backend implementing the k-NN (k=5) classifiers inside PreCanCell_classifier()) — https://github.com/davpinto/fastknn
- **R** (Language environment in which the benchmarking workflow and metric computation are executed)

## Examples

```
library(PreCanCell); testdata <- PreCanCell_data(input); results <- PreCanCell_classifier(testdata, cores = 2)
```

## Evaluation signals

- Accuracy exceeds 0.90 in at least 7 of the 11 test sets and falls between 0.80 and 0.90 in three
- Balanced accuracy exceeds 0.80 in all 11 test sets
- AUROC exceeds 0.80 in at least 8 test sets; sensitivity exceeds 0.80 in at least 9 sets
- Specificity exceeds 0.80 in all 11 test sets
- Per-cell results table has the expected schema (Sample, freq_cancer, freq_non_cancer, pred_label) with frequencies in [0,1] and pred_label in {cancer, non_cancer}
- Preprocessed test matrix contains only the 259 DEG features (73 TMGs + 186 NMGs) with expression values scaled to [0,1]

## Limitations

- The algorithm only annotates cancer vs non-cancer cells and does not resolve non-cancer subpopulations
- No accession IDs or repository links are provided for several test datasets (P1_BC, P4_SyS, P5_SyS, P7_GBM, P8_PDAC, P10_GBM), hindering exact reproduction
- No version number is stated for the PreCanCell R package, and no versions are given for the compared algorithms
- PreCanCell_data() requires any library-depth normalized input (e.g., TPM, CPM); raw counts are not accepted directly

## Evidence

- [abstract] In all these datasets, PreCanCell achieved above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC: "In all these datasets, PreCanCell achieved above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC"
- [methods] We then using the `PreCanCell_data()` function for data preprocessing that will select matched genes and scale gene expression values.: "We then using the `PreCanCell_data()` function for data preprocessing that will select matched genes and scale gene expression values."
- [methods] running `PreCanCell_classifier()` function to predict malignant and non-malignant cells: "running `PreCanCell_classifier()` function to predict malignant and non-malignant cells"
- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [readme] The input data can be any library-depth normalization (e.g. TPM, CPM).: "Its input should be normalized expression matrix with rownames being genes and colnames being cells. The input data can be any library-depth normalization (e.g. TPM, CPM)."
