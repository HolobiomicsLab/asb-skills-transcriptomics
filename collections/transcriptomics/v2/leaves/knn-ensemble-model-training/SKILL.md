---
name: knn-ensemble-model-training
description: Use when a normalized single-cell expression matrix must be classified
  as malignant vs non-malignant using PreCanCell's five-dataset k-NN (k=5) ensemble
  with majority vote over common DEG features.
when_to_use_negative:
- The goal is to identify subpopulations of non-cancer cells (immune, stromal, epithelial)
  — PreCanCell is limited to annotating cancer vs non-cancer cells
- You want to train custom models on your own labeled data — use the package's Custom_preprocessing()/Custom_classifier()
  path with a CustomDataList instead of the five-dataset ensemble
- The input matrix is raw counts, not library-depth normalized (TPM/CPM) — PreCanCell_data()
  expects a normalized expression matrix
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: PreCanCell
  role: R package providing PreCanCell_data() preprocessing and PreCanCell_classifier()
    that trains the five k-NN (k=5) base classifiers and aggregates them by majority
    vote
  repo: https://github.com/WangX-Lab/PreCanCell
- name: fastknn
  role: Backend implementing the k-NN (k=5) base classifiers used inside PreCanCell_classifier();
    installed via devtools::install_github("davpinto/fastknn")
  repo: https://github.com/davpinto/fastknn
- name: foreach / doParallel
  role: Parallel machinery used to execute the five base classifiers across multiple
    workers (cores argument of PreCanCell_classifier())
- name: Seurat
  role: Used for data preprocessing (version >= 4.3.0) of the input expression matrix
    before training
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:bc7b8b6a1d45bd1ace869d2e553e1065b11ecc20bbe339e55183b5f4569cbb16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - PreCanCell
  - R
  - Seurat
  - doParallel
  - fastknn
  - foreach
  - foreach / doParallel
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# knn-ensemble-model-training

## Summary

Reconstructs PreCanCell's ensemble classifier: train one k-NN (k=5) base classifier on each of five cancer single-cell transcriptome training datasets (RCC, HNSCC, melanoma, LUAD, BC) using the common DEGs (73 TMGs + 186 NMGs) as features, then assign a malignant/non-malignant label by majority vote of the five base results.

## When to use

Use when a normalized single-cell expression matrix must be classified as malignant vs non-malignant using PreCanCell's five-dataset k-NN (k=5) ensemble with majority vote over common DEG features.

## When NOT to use

- The goal is to identify subpopulations of non-cancer cells (immune, stromal, epithelial) — PreCanCell is limited to annotating cancer vs non-cancer cells
- You want to train custom models on your own labeled data — use the package's Custom_preprocessing()/Custom_classifier() path with a CustomDataList instead of the five-dataset ensemble
- The input matrix is raw counts, not library-depth normalized (TPM/CPM) — PreCanCell_data() expects a normalized expression matrix

## Inputs

- Normalized expression matrix with rownames being genes and colnames being cells (e.g. TPM or CPM), e.g. a tab-separated .txt file like the package example.txt
- Preprocessed testdata matrix from PreCanCell_data() with matched genes selected and values scaled to [0,1]
- Number of cores for parallel execution (cores argument)

## Outputs

- Results data frame with columns Sample, freq_cancer, freq_non_cancer, and pred_label for every cell, where freq_cancer + freq_non_cancer = 1 (ratios out of 5 base classifiers)
- Per-cell malignant/non-malignant labels from majority vote of five k-NN classifiers

## How to apply

First run PreCanCell_data() on a normalized expression matrix (rownames = genes, colnames = cells; TPM/CPM-style library-depth normalization acceptable) to select matched genes and min-max scale expression values to [0,1]. Then call PreCanCell_classifier(testdata, cores), which trains a fastknn k-NN (k=5) base classifier on each of the five cancer training datasets using the tumor marker genes (TMGs) and non-tumor marker genes (NMGs) as features, executing the five base classifiers in parallel via foreach/doParallel with the specified number of cores. The rationale for the ensemble is that each cancer type's dataset contributes an independent vote, and the final label is the majority of the five k-NN results. The output table reports freq_cancer and freq_non_cancer as the fraction of the five classifiers voting each way (ratios of x/5), plus the winning pred_label per cell.

## Related tools

- **PreCanCell** (R package providing PreCanCell_data() preprocessing and PreCanCell_classifier() that trains the five k-NN (k=5) base classifiers and aggregates them by majority vote) — https://github.com/WangX-Lab/PreCanCell
- **fastknn** (Backend implementing the k-NN (k=5) base classifiers used inside PreCanCell_classifier(); installed via devtools::install_github("davpinto/fastknn")) — https://github.com/davpinto/fastknn
- **foreach / doParallel** (Parallel machinery used to execute the five base classifiers across multiple workers (cores argument of PreCanCell_classifier()))
- **Seurat** (Used for data preprocessing (version >= 4.3.0) of the input expression matrix before training)

## Examples

```
library(PreCanCell); testdata <- PreCanCell_data(input); results <- PreCanCell_classifier(testdata, cores = 2)
```

## Evaluation signals

- The results table contains exactly the four columns Sample, freq_cancer, freq_non_cancer, pred_label for every input cell
- For every row, freq_cancer + freq_non_cancer = 1, and each frequency is a ratio of x/5 (e.g. 0, 0.2, 0.4, 0.6, 0.8, 1)
- Every pred_label value is one of 'cancer' or 'non_cancer' (malignant or non-malignant)
- The predicted label matches the majority of the five k-NN votes, i.e. freq_cancer > 0.5 implies 'cancer'
- Downstream benchmarking reproduces the paper's claim of accuracy, sensitivity, specificity, balanced accuracy and AUROC above 0.8 across the 19 tested single-cell datasets

## Limitations

- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection (DEG identification)
- The algorithm is limited to annotating cancer vs non-cancer cells; it is not designed for identifying subpopulations of non-cancer cells
- No version number is stated for the PreCanCell R package, complicating reproducibility

## Evidence

- [abstract] Five k-NN base classifiers + majority vote: "a single cell is classified as malignant or non-malignant by k-NN (k = 5)"
- [abstract] Majority vote aggregation: "the single cell is determined as malignant or non-malignant by the majority vote of the five k-NN classification results"
- [methods] Preprocessing via PreCanCell_data(): "We then using the `PreCanCell_data()` function for data preprocessing that will select matched genes and scale gene expression values."
- [readme] README usage of classifier function: "The function `PreCanCell_classifier()` is used to identify malignant and non-malignant cells from single-cell transcriptomes, containing 2 parameters: testdata and cores."
- [results] DEG feature composition: "The DEGs included 73 TMGs and 186 NMGs."
