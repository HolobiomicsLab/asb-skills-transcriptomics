---
name: deg-feature-based-supervised-classification
description: Use when a normalized single-cell expression matrix must be labeled cancer
  vs non-cancer by a supervised model pre-trained on the five cancer transcriptome
  datasets with DEG features.
when_to_use_negative:
- The goal is to annotate non-cancer cell subpopulations (immune, stromal, epithelial)
  — the algorithm is limited to annotating cancer vs non-cancer cells only
- The expression matrix is raw counts and has not been depth-normalized — PreCanCell_data()
  requires a normalized expression matrix (e.g. TPM, CPM) as input
- No well-annotated training data or DEG feature set is available and only a custom
  dataset is being used without the Custom_preprocessing()/Custom_classifier() variant
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: PreCanCell
  role: R package providing PreCanCell_data() for gene matching/[0,1] scaling and
    PreCanCell_classifier() for the ensemble k-NN (k=5) majority-vote classification
  repo: https://github.com/WangX-Lab/PreCanCell
- name: fastknn
  role: k-NN backend used to train each of the five base classifiers with k = 5 on
    the DEG features
  repo: davpinto/fastknn
- name: Seurat
  role: Used for data preprocessing, including NormalizeData() and FindMarkers() to
    derive the DEG features (version >= 4.3.0 required)
- name: foreach
  role: Parallel machinery (with doParallel) used to execute the five base k-NN classifiers
    concurrently, controlled by the cores argument
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:8e3d7b0d6def90d34f4c89efec0d9d0ee13e9c5c20a195fdc6357cff459dcea3
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
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# deg-feature-based-supervised-classification

## Summary

Supervised ensemble classification of single cells as malignant or non-malignant using k-NN (k=5) base classifiers trained on five cancer scRNA-seq datasets with common DEGs (73 TMGs and 186 NMGs) as features, combined by majority vote.

## When to use

Use when a normalized single-cell expression matrix must be labeled cancer vs non-cancer by a supervised model pre-trained on the five cancer transcriptome datasets with DEG features.

## When NOT to use

- The goal is to annotate non-cancer cell subpopulations (immune, stromal, epithelial) — the algorithm is limited to annotating cancer vs non-cancer cells only
- The expression matrix is raw counts and has not been depth-normalized — PreCanCell_data() requires a normalized expression matrix (e.g. TPM, CPM) as input
- No well-annotated training data or DEG feature set is available and only a custom dataset is being used without the Custom_preprocessing()/Custom_classifier() variant

## Inputs

- Normalized single-cell expression matrix (rownames = genes, colnames = cells; e.g. TPM or CPM values)
- Preprocessed output of PreCanCell_data(): matched genes with expression min-max scaled to [0,1]

## Outputs

- Results table/data frame with columns Sample, freq_cancer, freq_non_cancer, and pred_label per cell (pred_label is cancer or non_cancer)

## How to apply

First run PreCanCell_data() on a normalized expression matrix (rownames = genes, colnames = cells; TPM/CPM or any depth normalization) so matched genes are selected and expression is min-max scaled to [0,1]. Then call PreCanCell_classifier(testdata, cores = N), which trains one k-NN classifier with k = 5 on each of the five training datasets (RCC, HNSCC, melanoma, LUAD, BC) using the DEGs — 73 tumor marker genes and 186 non-tumor marker genes — as features via the fastknn backend, executing the five base classifiers in parallel with foreach/doParallel. For each cell, aggregate the five k-NN votes: compute freq_cancer and freq_non_cancer as fractions out of 5 and assign the final label by majority vote. Rationale: using DEGs shared across five cancer types and an ensemble vote improves robustness of malignant/non-malignant calls over single classifiers.

## Related tools

- **PreCanCell** (R package providing PreCanCell_data() for gene matching/[0,1] scaling and PreCanCell_classifier() for the ensemble k-NN (k=5) majority-vote classification) — https://github.com/WangX-Lab/PreCanCell
- **fastknn** (k-NN backend used to train each of the five base classifiers with k = 5 on the DEG features) — davpinto/fastknn
- **Seurat** (Used for data preprocessing, including NormalizeData() and FindMarkers() to derive the DEG features (version >= 4.3.0 required))
- **foreach** (Parallel machinery (with doParallel) used to execute the five base k-NN classifiers concurrently, controlled by the cores argument)

## Examples

```
library(PreCanCell); testdata <- PreCanCell_data(input); results <- PreCanCell_classifier(testdata, cores = 2)
```

## Evaluation signals

- The results table contains exactly the columns Sample, freq_cancer, freq_non_cancer, and pred_label for every cell
- freq_cancer + freq_non_cancer = 1 for each row
- Every pred_label value is one of the two allowed labels: cancer or non_cancer
- freq_cancer and freq_non_cancer take values in {0, 0.2, 0.4, 0.6, 0.8, 1.0} since they are ratios out of five base classifiers
- Input to PreCanCell_classifier() is the output of PreCanCell_data() with expression values bounded in [0,1] and restricted to matched genes

## Limitations

- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection
- The algorithm only annotates cancer vs non-cancer cells; it does not identify subpopulations of non-cancer cells (immune, stromal, epithelial)
- No version number is stated for the PreCanCell R package

## Evidence

- [intro] With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5): "With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5)"
- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [readme] The function `PreCanCell_data()` is used to data preprocessing. Its input should be normalized expression matrix with rownames being genes and colnames being cells.: "The function `PreCanCell_data()` is used to data preprocessing. Its input should be normalized expression matrix with rownames being genes and colnames being cells."
- [readme] The function `PreCanCell_classifier()` is used to identify malignant and non-malignant cells from single-cell transcriptomes, containing 2 parameters: testdata and cores.: "The function `PreCanCell_classifier()` is used to identify malignant and non-malignant cells from single-cell transcriptomes, containing 2 parameters: testdata and cores."
- [discussion] the reported predictive performance in the training set could be overestimated due to the predicted sets being involved in feature selection: "the reported predictive performance in the training set could be overestimated due to the predicted sets being involved in feature selection"
