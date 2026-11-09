---
name: synthetic-scrnaseq-dataset-merging
description: 'Use when evaluating cancer-cell prediction on data lacking mixed ground
  truth: you have one all-malignant and one all-normal scRNA-seq matrix and need a
  labeled mixed benchmark to compute FPR, FNR, accuracy and AUROC.'
when_to_use_negative:
- The task is to identify subpopulations of non-cancer cells (immune, stromal, epithelial)
  — PreCanCell is limited to annotating cancer vs non-cancer cells only
- The input is a mixed malignant/non-malignant dataset that already has ground-truth
  labels for both classes — evaluate directly instead of constructing a synthetic
  merge
- The input is raw unnormalized counts with no library-depth normalization applied
  — PreCanCell_data() expects a normalized expression matrix (TPM, CPM, etc.)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PreCanCell
  role: R package providing PreCanCell_data() (DEG-feature selection and [0,1] min-max
    scaling) and PreCanCell_classifier() (ensemble k-NN k=5 with majority vote over
    five training datasets)
  repo: https://github.com/WangX-Lab/PreCanCell
- name: Seurat
  role: Normalization of scRNA-seq expression values via NormalizeData() before merging
    and preprocessing (version >= 4.3.0)
- name: fastknn
  role: Dependency providing the k-NN implementation underlying PreCanCell_classifier()
  repo: https://github.com/davpinto/fastknn
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:d0ccdfc6eda8b3e42fcc7314164fb0097ccc593cba0c58bfe0424ad7ab1d9a6b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# Synthetic scRNA-seq dataset merging

## Summary

Constructs a synthetic cancer/non-cancer benchmark by artificially merging a solely malignant scRNA-seq dataset (GSE157220 pan-cancer cell lines) with a solely normal one (GSE126030 healthy-tissue immune cells) into a single expression matrix, enabling quantitative evaluation of PreCanCell's malignant vs non-malignant classification.

## When to use

Use when evaluating cancer-cell prediction on data lacking mixed ground truth: you have one all-malignant and one all-normal scRNA-seq matrix and need a labeled mixed benchmark to compute FPR, FNR, accuracy and AUROC.

## When NOT to use

- The task is to identify subpopulations of non-cancer cells (immune, stromal, epithelial) — PreCanCell is limited to annotating cancer vs non-cancer cells only
- The input is a mixed malignant/non-malignant dataset that already has ground-truth labels for both classes — evaluate directly instead of constructing a synthetic merge
- The input is raw unnormalized counts with no library-depth normalization applied — PreCanCell_data() expects a normalized expression matrix (TPM, CPM, etc.)

## Inputs

- GSE157220 normalized pan-cancer cell-line scRNA-seq expression matrix (solely malignant labels)
- GSE126030 normalized healthy-tissue immune-cell scRNA-seq expression matrix (solely non-malignant labels)
- PreCanCell DEG feature list (73 TMGs and 186 NMGs) shipped with the PreCanCell R package
- Number of cores for parallel k-NN execution

## Outputs

- Merged synthetic expression matrix with ground-truth cancer/non-cancer labels per cell
- Per-cell predictions with freq_cancer, freq_non_cancer and pred_label (cancer/non_cancer)
- Evaluation metrics: FPR, FNR, accuracy, sensitivity, specificity, balanced accuracy, AUROC (written to evaluation_metrics.csv)

## How to apply

Download GSE157220 (pan-cancer cell-line scRNA-seq) and GSE126030 (immune cells across healthy human tissues) from GEO, applying the quality control described in their original publications and normalizing (e.g., Seurat NormalizeData; any library-depth normalization like TPM or CPM is accepted by PreCanCell). Merge the malignant and non-malignant matrices into one expression matrix with rownames as genes and colnames as cells. Preprocess with PreCanCell_data(), which selects the matched DEG features (73 TMGs and 186 NMGs) and min-max scales expression values to [0,1]. Run PreCanCell_classifier(), which classifies every cell by k-NN (k=5) against each of the five training datasets and assigns the final label by majority vote of the five results. Compute FPR on GSE126030, FNR on GSE157220, and accuracy, sensitivity, specificity, balanced accuracy and AUROC on the merged synthetic dataset against the ground-truth annotations; the published reference values are FPR 0.0089, FNR 0.0041, and 0.995/0.998/0.993/0.995/0.995 respectively.

## Related tools

- **PreCanCell** (R package providing PreCanCell_data() (DEG-feature selection and [0,1] min-max scaling) and PreCanCell_classifier() (ensemble k-NN k=5 with majority vote over five training datasets)) — https://github.com/WangX-Lab/PreCanCell
- **Seurat** (Normalization of scRNA-seq expression values via NormalizeData() before merging and preprocessing (version >= 4.3.0))
- **fastknn** (Dependency providing the k-NN implementation underlying PreCanCell_classifier()) — https://github.com/davpinto/fastknn

## Examples

```
testdata <- PreCanCell_data(merged_matrix); results <- PreCanCell_classifier(testdata, cores = 2)
```

## Evaluation signals

- FPR on GSE126030 should match the reported 0.0089 (fraction of non-malignant cells mislabeled cancer)
- FNR on GSE157220 should match the reported 0.0041 (fraction of malignant cells mislabeled non-cancer)
- Synthetic merged-dataset accuracy, sensitivity, specificity, balanced accuracy and AUROC should reproduce ~0.995, 0.998, 0.993, 0.995, 0.995
- PreCanCell_data() output must contain only the matched DEG features and values scaled to [0,1]
- All metrics should be reproducibly written to evaluation_metrics.csv

## Limitations

- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection
- The algorithm is limited to annotating cancer vs non-cancer cells and does not identify non-cancer cell subpopulations
- No version number is stated for the PreCanCell R package, complicating exact reproduction

## Evidence

- [other] Task card: dataset construction: [evidence span withheld — anchor status: not_found]
- [methods] Dataset provenance (cancer): "we obtained a scRNA-seq dataset for pan-cancer cell lines (GSE157220)"
- [methods] Dataset provenance (normal): "a scRNA-Seq dataset for immune cells across health human tissues (GSE126030)"
- [abstract] Classifier mechanics: "a single cell is classified as malignant or non-malignant by k-NN (k = 5)"
- [readme] README: preprocessing requirements: "The input data can be any library-depth normalization (e.g. TPM, CPM)."
- [readme] README: classifier input: ""testdata" is a output matrix of the function `PreCanCell_data()`."
- [methods] Feature selection scaling: "We then using the `PreCanCell_data()` function for data preprocessing that will select matched genes and scale gene expression values."
