---
name: random-forest-cell-type-classification
description: Use when comparing imputation or normalization methods on labeled single-cell
  RNA-seq expression matrices and you need a classifier-based metric (out-of-bag error)
  of cell-type separation before vs after imputation.
when_to_use_negative:
- Cell-identity labels are unavailable or unreliable, since the classifier requires
  supervised labels to produce OOB error
- The matrix is not in cells-as-rows, genes-as-columns orientation (ALRA/normalize_data
  expects this layout, and mixing it up silently corrupts the classification)
- The goal is clustering of unlabeled cells rather than quantifying class separability
  with a supervised metric
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ranger
  role: Random forest implementation (default settings) used to predict cell class
    and report out-of-bag classification error
- name: ALRA
  role: Normalization (normalize_data) and zero-preserving imputation of the expression
    matrices prior to classification
  repo: github.com/KlugerLab/ALRA
- name: MAGIC
  role: Baseline imputer run on the same matrices (with negative values set to zero)
    for OOB-error comparison
- name: SAVER
  role: Baseline imputer (do.fast = TRUE) for OOB-error comparison
- name: DCA
  role: Baseline imputer for OOB-error comparison
- name: scImpute
  role: Baseline imputer for OOB-error comparison; found to increase OOB error
- name: Seurat
  role: Single-cell toolkit integrating ALRA (RunALRA) for preparing Seurat objects
    for classification
  repo: satijalab/seurat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:b3a664df383e4be33552d86af88b37837f3f5485c2eec73332f23811b119738d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALRA
  - DCA
  - MAGIC
  - SAVER
  - Seurat
  - ranger
  - scImpute
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# random-forest-cell-type-classification

## Summary

Trains a ranger random forest with default settings to predict cell class from normalized (raw or imputed) scRNA-seq expression matrices and uses the out-of-bag classification error as a quantitative benchmark for whether imputation (e.g. ALRA vs MAGIC, SAVER, DCA, scImpute) improves cell-type separability.

## When to use

Use when comparing imputation or normalization methods on labeled single-cell RNA-seq expression matrices and you need a classifier-based metric (out-of-bag error) of cell-type separation before vs after imputation.

## When NOT to use

- Cell-identity labels are unavailable or unreliable, since the classifier requires supervised labels to produce OOB error
- The matrix is not in cells-as-rows, genes-as-columns orientation (ALRA/normalize_data expects this layout, and mixing it up silently corrupts the classification)
- The goal is clustering of unlabeled cells rather than quantifying class separability with a supervised metric

## Inputs

- Normalized scRNA-seq expression matrix (cells as rows, genes as columns), raw or imputed (e.g. ALRA, MAGIC, SAVER, DCA, scImpute output)
- Cell-identity labels provided by the original dataset authors (Zeisel, Baron, Chen, La Manno)

## Outputs

- Per-dataset, per-method out-of-bag classification error values
- Comparison table (oob_error_table.csv) of average OOB error across datasets for each imputation method

## How to apply

Load the labeled expression matrices (e.g. downsampled Zeisel 1799 cells, Baron 1076 cells, Chen 7712 cells, La Manno 947 cells from Huang et al. 2018) and normalize each by library normalization to 10,000 counts followed by log-transformation (ALRA's normalize_data). For each method condition (raw plus ALRA with choose_k-selected rank and quantile.prob = 0.001, MAGIC with negatives set to zero, SAVER do.fast = TRUE, DCA, and scImpute), train a ranger random forest with default settings using cell-identity labels as the response and gene expression values as predictors. Record the out-of-bag error per dataset and method, average across the four datasets, and write the comparison table (e.g. oob_error_table.csv). OOB error is used instead of a held-out test split because ranger estimates it internally from the trees in which each cell was not used, giving an unbiased error estimate without extra computation.

## Related tools

- **ranger** (Random forest implementation (default settings) used to predict cell class and report out-of-bag classification error)
- **ALRA** (Normalization (normalize_data) and zero-preserving imputation of the expression matrices prior to classification) — github.com/KlugerLab/ALRA
- **MAGIC** (Baseline imputer run on the same matrices (with negative values set to zero) for OOB-error comparison)
- **SAVER** (Baseline imputer (do.fast = TRUE) for OOB-error comparison)
- **DCA** (Baseline imputer for OOB-error comparison)
- **scImpute** (Baseline imputer for OOB-error comparison; found to increase OOB error)
- **Seurat** (Single-cell toolkit integrating ALRA (RunALRA) for preparing Seurat objects for classification) — satijalab/seurat

## Examples

```
A_norm <- normalize_data(b_nk_example); k_choice <- choose_k(A_norm); A_completed <- alra(A_norm, k = k_choice$k, quantile.prob = 0.001)[[3]]; rf <- ranger(label ~ ., data = data.frame(A_completed, label = labels))
```

## Evaluation signals

- ALRA-imputed matrices should yield substantially lower OOB error than raw (16.4% → ~8% average across the four downsampled datasets), outperforming MAGIC (10.3%), SAVER (9.8%), and DCA (9.4%)
- scImpute-treated matrices should show increased OOB error (~28%), a known negative-control direction
- The predictor matrix passed to the classifier must have cells as rows and genes as columns, matching the normalization/imputation input requirement
- Expected auto-selected ALRA ranks: k = 12 (Baron), 21 (Chen), 15 (La Manno), 16 (Zeisel)
- Output must contain one OOB-error value per dataset per method and an averaged comparison table (oob_error_table.csv)

## Limitations

- scImpute can increase classification error substantially (to 28% in this benchmark), so imputation is not uniformly beneficial
- Some methods (MAGIC, DCA) treat all zeros as missing data and output a matrix in which every gene is expressed in every cell, which changes the zeros semantics the classifier sees
- SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells; scImpute may need to be run on a subset for tractability
- Cell-type separation as measured by OOB error is an indirect measure of imputation quality, not a direct check that biological zeros were preserved

## Evidence

- [intro] dropout makes cell types less separable; classifiers quantify this: "a phenomenon called “dropout” in which an expressed transcript is not detected and hence assigned a zero expression value"
- [intro] zeros dominate the matrix being classified: "the fraction of zeros in a typical scRNA-seq expression matrix is much higher, even exceeding 90%"
- [readme] ALRA input orientation for matrices fed downstream to classification: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] normalization step preceding imputation and classification: "# We use library and log normalization, but other approaches may also work well.
result.completed <- alra(A_norm)"
- [readme] rank selection used before producing the imputed matrix for the classifier: "A_norm_completed <- alra(A_norm,k=k_choice$k)[[3]]"
