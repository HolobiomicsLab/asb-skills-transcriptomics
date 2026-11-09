---
name: knn-ensemble-majority-voting
description: Use when classifying cells from a normalized scRNA-seq expression matrix
  (genes as rows, cells as columns) as cancer vs non-cancer, and per-cell voting frequencies
  (freq_cancer/freq_non_cancer) are needed for downstream accuracy/sensitivity/AUROC
  evaluation.
when_to_use_negative:
- The goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial)
  — the algorithm is limited to annotating cancer vs non-cancer cells only
- The input is raw counts rather than a normalized expression matrix (e.g. TPM/CPM)
  — PreCanCell_data() expects depth-normalized values
- Training-set performance is claimed as unbiased — the reported training-set accuracy
  is overestimated because the predicted sets were involved in feature selection (DEG
  identification)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PreCanCell
  role: R package providing PreCanCell_data() for DEG-panel selection and [0,1] scaling
    and PreCanCell_classifier() for the five-way k-NN ensemble with majority voting
  repo: https://github.com/WangX-Lab/PreCanCell
- name: fastknn
  role: R package providing the k-NN classifier (k=5) used for each of the five training-set
    classifications
  repo: https://github.com/davpinto/fastknn
- name: R
  role: Runtime environment for the PreCanCell and fastknn packages
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:5116ff500f39c55a399be3c240ba87fdae7761b3e7495fbd7c5cb4023da662b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - PreCanCell
  - R
  - fastknn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# knn-ensemble-majority-voting

## Summary

An ensemble classification skill (from PreCanCell) that labels single cells as malignant or non-malignant by running k-NN (k=5) five times — once per cancer training dataset (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) — over a 259-gene DEG panel (73 TMGs + 186 NMGs) and taking the majority vote.

## When to use

Use when classifying cells from a normalized scRNA-seq expression matrix (genes as rows, cells as columns) as cancer vs non-cancer, and per-cell voting frequencies (freq_cancer/freq_non_cancer) are needed for downstream accuracy/sensitivity/AUROC evaluation.

## When NOT to use

- The goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial) — the algorithm is limited to annotating cancer vs non-cancer cells only
- The input is raw counts rather than a normalized expression matrix (e.g. TPM/CPM) — PreCanCell_data() expects depth-normalized values
- Training-set performance is claimed as unbiased — the reported training-set accuracy is overestimated because the predicted sets were involved in feature selection (DEG identification)

## Inputs

- Normalized scRNA-seq expression matrix (rows = genes, columns = cells; TPM or CPM depth normalization acceptable)
- Cell-level ground-truth annotations (malignant/non-malignant) for performance evaluation
- Five cancer scRNA-seq training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) bundled with PreCanCell

## Outputs

- Per-cell classification table with columns Sample, freq_cancer, freq_non_cancer, pred_label (cancer / non_cancer)
- Per-dataset metrics: accuracy, sensitivity, specificity, balanced accuracy, and AUROC (e.g. assembled into test_set_metrics.csv)

## How to apply

First preprocess the normalized expression matrix (TPM/CPM acceptable) with PreCanCell_data(), which intersects genes with the 259-gene DEG feature panel and min-max scales expression values to [0,1]. Then run PreCanCell_classifier() on the preprocessed matrix: for each of the five training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC), classify every cell as malignant or non-malignant by k-NN with k=5 (via the R fastknn package), then assign the final label by majority vote of the five k-NN results, recording freq_cancer and freq_non_cancer per cell. Ground-truth evaluation uses accuracy, sensitivity, specificity, balanced accuracy (mean of sensitivity and specificity), and AUROC; in the paper, PreCanCell exceeded 0.8 in all five metrics across 19 tested datasets, with accuracy above 0.90 in seven of 11 independent test sets and balanced accuracy above 0.80 in all 11.

## Related tools

- **PreCanCell** (R package providing PreCanCell_data() for DEG-panel selection and [0,1] scaling and PreCanCell_classifier() for the five-way k-NN ensemble with majority voting) — https://github.com/WangX-Lab/PreCanCell
- **fastknn** (R package providing the k-NN classifier (k=5) used for each of the five training-set classifications) — https://github.com/davpinto/fastknn
- **R** (Runtime environment for the PreCanCell and fastknn packages)

## Examples

```
testdata <- PreCanCell_data(input); results <- PreCanCell_classifier(testdata, cores = 2)
```

## Evaluation signals

- Output table contains per-cell columns Sample, freq_cancer, freq_non_cancer and pred_label, with freq values in [0,1] and pred_label equal to 'cancer' when freq_cancer > freq_non_cancer
- Against published cell annotations, accuracy exceeds 0.90 in at least 7 of 11 independent test sets and balanced accuracy exceeds 0.80 in all 11
- AUROC exceeds 0.80 in at least 8 test sets, sensitivity exceeds 0.80 in at least 9, and specificity exceeds 0.80 in all 11
- All five metrics (accuracy, sensitivity, specificity, balanced accuracy, AUROC) remain above 0.8 across tested datasets, matching the paper's reported performance
- Preprocessed matrix retains only the 259 DEG panel genes with expression values min-max scaled to [0,1]

## Limitations

- Training-set predictive performance is overestimated because the predicted sets were involved in feature selection (DEG identification)
- The algorithm only annotates cancer vs non-cancer cells and is not designed for identifying subpopulations of non-cancer cells
- No version number is stated for the PreCanCell R package, complicating reproducibility
- No accession IDs or repository links are provided for several named train/test datasets (P1_BC, T5_BC, P4_SyS, P5_SyS, P7_GBM, P8_PDAC, P10_GBM)

## Evidence

- [intro] With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5): "With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5)"
- [methods] running `PreCanCell_classifier()` function to predict malignant and non-malignant cells: "running `PreCanCell_classifier()` function to predict malignant and non-malignant cells"
- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [abstract] In all these datasets, PreCanCell achieved above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC: "In all these datasets, PreCanCell achieved above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC"
- [readme] The function `PreCanCell_data()` is used to data preprocessing. Its input should be normalized expression matrix with rownames being genes and colnames being cells.: "The function `PreCanCell_data()` is used to data preprocessing. Its input should be normalized expression matrix with rownames being genes and colnames being cells."
