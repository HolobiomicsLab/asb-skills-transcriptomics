---
name: scrna-seq-expression-matrix-preprocessing
description: Use when you have a normalized (TPM/CPM or Seurat NormalizeData()) scRNA-seq
  matrix — genes as rows, cells as columns — that must be matched to the PreCanCell
  DEG panel and scaled before malignant/non-malignant cell classification.
when_to_use_negative:
- The input is raw unnormalized counts — normalize first (e.g. Seurat NormalizeData())
  since PreCanCell_data() expects a library-depth-normalized matrix
- The goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial)
  — PreCanCell is limited to cancer vs non-cancer labels only
- You need to retrain on new annotated data — use Custom_preprocessing()/Custom_classifier()
  instead of the fixed 259-gene DEG pipeline
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: PreCanCell
  role: Provides PreCanCell_data() which selects matched DEG-panel genes and min-max
    scales expression values to [0,1]
  repo: https://github.com/WangX-Lab/PreCanCell
- name: Seurat
  role: NormalizeData() produces the normalized expression matrix required as input;
    Seurat version >= 4.3.0 is used for data preprocessing
- name: fastknn
  role: k-NN (k=5) implementation underlying the downstream PreCanCell_classifier()
    step that consumes the preprocessed matrix
  repo: https://github.com/davpinto/fastknn
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:d4629a09a30359c9a3c26797cee6ee0a98e026b8d1ae24b44cfc5d58a383d5a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - ConsensusClusterPlus
  - PreCanCell
  - R
  - RCC
  - Seurat
  - fastknn
  - mclust
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# scRNA-seq-expression-matrix-preprocessing

## Summary

Preprocess a normalized single-cell RNA-seq expression matrix with PreCanCell_data() so it matches the 259-gene DEG feature panel (73 TMGs + 186 NMGs) and is min-max scaled to [0,1], making it a valid input for the PreCanCell k-NN ensemble classifier.

## When to use

Use when you have a normalized (TPM/CPM or Seurat NormalizeData()) scRNA-seq matrix — genes as rows, cells as columns — that must be matched to the PreCanCell DEG panel and scaled before malignant/non-malignant cell classification.

## When NOT to use

- The input is raw unnormalized counts — normalize first (e.g. Seurat NormalizeData()) since PreCanCell_data() expects a library-depth-normalized matrix
- The goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial) — PreCanCell is limited to cancer vs non-cancer labels only
- You need to retrain on new annotated data — use Custom_preprocessing()/Custom_classifier() instead of the fixed 259-gene DEG pipeline

## Inputs

- Normalized scRNA-seq expression matrix (e.g. TPM or CPM) with rownames = genes and colnames = cells
- Seurat-normalized expression matrix (NormalizeData() output) if starting from raw counts

## Outputs

- Preprocessed matrix restricted to the 259-gene DEG panel (73 TMGs + 186 NMGs) with gene expression min-max scaled to [0,1], suitable as 'testdata' for PreCanCell_classifier()

## How to apply

Start from a normalized expression matrix whose rownames are genes and colnames are cells; any library-depth normalization such as TPM or CPM is acceptable, and Seurat's NormalizeData() can be used if counts are raw. Run PreCanCell_data() from the PreCanCell R package, which intersects the input genes with the 259-gene DEG feature panel (73 TMGs and 186 NMGs identified across the five training datasets T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) and min-max scales each gene's expression values to [0,1]. The rationale is that the downstream PreCanCell_classifier() ensemble was trained on exactly these features at this scale, so unmatched genes or unscaled values break the k-NN (k=5) majority-vote predictions. Validate the output by checking that the returned matrix contains only DEG-panel genes, all values lie in [0,1], and cells remain as columns; only then pass it to PreCanCell_classifier(testdata, cores) for classification.

## Related tools

- **PreCanCell** (Provides PreCanCell_data() which selects matched DEG-panel genes and min-max scales expression values to [0,1]) — https://github.com/WangX-Lab/PreCanCell
- **Seurat** (NormalizeData() produces the normalized expression matrix required as input; Seurat version >= 4.3.0 is used for data preprocessing)
- **fastknn** (k-NN (k=5) implementation underlying the downstream PreCanCell_classifier() step that consumes the preprocessed matrix) — https://github.com/davpinto/fastknn

## Examples

```
library(PreCanCell); testdata <- PreCanCell_data(input)  # input: normalized matrix, genes as rows, cells as columns
```

## Evaluation signals

- Output matrix rownames are a subset of the 259-gene DEG panel (73 TMGs + 186 NMGs); no non-panel genes remain
- All expression values in the preprocessed matrix lie within [0,1] after min-max scaling
- Cell (column) count is unchanged from input; no cells dropped during gene intersection
- Downstream PreCanCell_classifier() returns per-cell freq_cancer, freq_non_cancer, and pred_label (cancer/non_cancer) without errors
- On the 11 independent test sets (P1_BC ... P11_Melanoma), the full preprocess+classify pipeline yields accuracy > 0.80 and balanced accuracy > 0.80 per the paper's reported performance

## Limitations

- PreCanCell_data() requires an already-normalized matrix (TPM, CPM, etc.); feeding raw counts is invalid
- Genes not in the fixed 259-gene DEG panel are discarded, so the preprocessing is not usable for custom feature sets — use Custom_preprocessing() for newly trained models
- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection
- No version number is stated for the PreCanCell R package

## Evidence

- [methods] PreCanCell_data() selects matched genes and scales gene expression: "We then using the `PreCanCell_data()` function for data preprocessing that will select matched genes and scale gene expression values."
- [readme] Input must be normalized matrix with genes as rows: "The function `PreCanCell_data()` is used to data preprocessing. Its input should be normalized expression matrix with rownames being genes and colnames being cells."
- [readme] Accepted normalizations and scaling target: "The input data can be any library-depth normalization (e.g. TPM, CPM)."
- [results] DEG panel composition of 73 TMGs and 186 NMGs: "The DEGs included 73 TMGs and 186 NMGs."
- [intro] Output feeds the classifier on DEG features: "With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5)"
