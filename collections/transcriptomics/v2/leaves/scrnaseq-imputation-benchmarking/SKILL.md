---
name: scrnaseq-imputation-benchmarking
description: Use when comparing imputation methods on scRNA-seq count matrices to
  test whether imputation improves cell-type separation or preserves biological zeros,
  e.g. before-vs-after classification error on downsampled Zeisel/Baron/Chen/La Manno
  data.
when_to_use_negative:
- Input is not raw scRNA-seq count data needing imputation, e.g. already-normalized
  bulk RNA-seq or a feature/summary table
- The dataset exceeds ~100,000 cells and runtime matters — the article notes 'SAVER
  and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells'
- The goal is differential-expression analysis or trajectory inference on already-complete
  data rather than imputation benchmarking
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ALRA
  role: 'Primary zero-preserving imputation method: normalize_data, choose_k, and
    alra with quantile.prob = 0.001'
  repo: https://github.com/KlugerLab/ALRA
- name: MAGIC
  role: Baseline imputer whose dense output has negative values set to zero before
    comparison
- name: SAVER
  role: Baseline zero-preserving imputer run with do.fast = TRUE on the same matrices
- name: DCA
  role: Baseline deep-count autoencoder imputer run on the same matrices
- name: scImpute
  role: Baseline imputer, run on a subset where needed for tractability
- name: ranger
  role: Random forest used to compute out-of-bag classification error of cell class
    from raw vs imputed expression values
- name: Seurat
  role: Single-cell toolkit hosting RunALRA() integration for applying ALRA within
    Seurat workflows
  repo: https://github.com/satijalab/seurat
- name: rpca-mkl
  role: Optional MKL-based randomized SVD backend enabling speedups of ALRA via the
    use.mkl flag
  repo: https://github.com/KlugerLab/rpca-mkl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:d76c81bce8b7f58c95d8722a819c6831858cc0cf8e32a31ee54628261c46df37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - DCA
  - MAGIC
  - SAVER
  - Seurat
  - ranger
  - rpca-mkl
  - scImpute
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# scrnaseq-imputation-benchmarking

## Summary

Benchmarks zero-preserving imputation of scRNA-seq expression matrices — chiefly ALRA's low-rank approximation with adaptive thresholding — against MAGIC, SAVER, DCA, and scImpute using downsampled datasets and downstream readouts such as ranger random-forest out-of-bag classification error.

## When to use

Use when comparing imputation methods on scRNA-seq count matrices to test whether imputation improves cell-type separation or preserves biological zeros, e.g. before-vs-after classification error on downsampled Zeisel/Baron/Chen/La Manno data.

## When NOT to use

- Input is not raw scRNA-seq count data needing imputation, e.g. already-normalized bulk RNA-seq or a feature/summary table
- The dataset exceeds ~100,000 cells and runtime matters — the article notes 'SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells'
- The goal is differential-expression analysis or trajectory inference on already-complete data rather than imputation benchmarking

## Inputs

- Downsampled scRNA-seq expression matrices (Zeisel 1799 cells, Baron 1076 cells, Chen 7712 cells, La Manno 947 cells) as distributed with Huang et al. 2018 (SAVER)
- Cell-identity class labels provided by the original dataset authors

## Outputs

- Imputed expression matrices from ALRA, MAGIC, SAVER, DCA, and scImpute
- Out-of-bag classification error per dataset and method, averaged across the four datasets, written to oob_error_table.csv

## How to apply

Load the downsampled expression matrices (Zeisel 1799 cells, Baron 1076 cells, Chen 7712 cells, La Manno 947 cells) with original cell-identity labels, then library-normalize to 10,000 counts and log-transform with ALRA's normalize_data. Impute with ALRA, letting choose_k automatically select the rank (expected k = 12, 21, 15, 16 for Baron, Chen, La Manno, Zeisel respectively) and using alra with quantile.prob = 0.001; ALRA computes a rank-k randomized-SVD approximation, thresholds each gene by the magnitude of its most negative value to restore biological zeros, and rescales. Run baseline imputers on the same matrices: MAGIC (negative values set to zero), SAVER (do.fast = TRUE), DCA, and scImpute (on a subset where needed for tractability). Train a ranger random forest with default settings to predict cell class from each raw and imputed matrix, record out-of-bag error per dataset and method, average across the four datasets, and write oob_error_table.csv. Correct application is judged by ALRA's zero-preservation property and by lower average OOB error than baselines.

## Related tools

- **ALRA** (Primary zero-preserving imputation method: normalize_data, choose_k, and alra with quantile.prob = 0.001) — https://github.com/KlugerLab/ALRA
- **MAGIC** (Baseline imputer whose dense output has negative values set to zero before comparison)
- **SAVER** (Baseline zero-preserving imputer run with do.fast = TRUE on the same matrices)
- **DCA** (Baseline deep-count autoencoder imputer run on the same matrices)
- **scImpute** (Baseline imputer, run on a subset where needed for tractability)
- **ranger** (Random forest used to compute out-of-bag classification error of cell class from raw vs imputed expression values)
- **Seurat** (Single-cell toolkit hosting RunALRA() integration for applying ALRA within Seurat workflows) — https://github.com/satijalab/seurat
- **rpca-mkl** (Optional MKL-based randomized SVD backend enabling speedups of ALRA via the use.mkl flag) — https://github.com/KlugerLab/rpca-mkl

## Examples

```
library(ALRA); A_norm <- normalize_data(A); k_choice <- choose_k(A_norm); A_norm_completed <- alra(A_norm, k = k_choice$k, quantile.prob = 0.001)[[3]]
```

## Evaluation signals

- ALRA-imputed matrices retain true biological zeros at zero expression while completing technical zeros (zero-preserving property)
- choose_k automatically selects rank k ≈ 12, 21, 15, 16 for Baron, Chen, La Manno, Zeisel respectively
- Average OOB error drops substantially with ALRA (16.4% raw → 8%), outperforming MAGIC (10.3%), SAVER (9.8%), and DCA (9.4%)
- scImpute degrades cell-type separation, raising average OOB error to 28% — a signal of an over-smoothing imputer
- oob_error_table.csv contains one OOB error per dataset × method combination covering all four datasets and five imputers plus raw

## Limitations

- MAGIC and DCA treat all zeros as missing data and output a matrix in which every gene is expressed in every cell, so they do not preserve biological zeros
- SAVER and scImpute may be extremely slow on datasets with over 100,000 cells, motivating subsetting
- ALRA requires a normalized matrix with cells as rows and genes as columns; passing the wrong orientation or unnormalized data is unsupported
- rpca-mkl is under development, not on CRAN, and its pre-compiled MKL libraries are only available for Linux and OS X

## Evidence

- [readme] ALRA is a method for imputation of missing values in single cell RNA-sequencing data: "ALRA is a method for imputation of missing values in single cell RNA-sequencing data"
- [readme] ALRA first computes its rank-k approximation using randomized SVD and thresholds each gene: "ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene."
- [intro] Article intro: methods such as DCA and MAGIC do not preserve biological zeros: "Some methods such as DCA and MAGIC treat all zeros as “missing data” and output a matrix in which every gene is expressed in every cell"
- [intro] Article intro: scalability caveat for SAVER and scImpute: "SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells"
- [intro] Article intro: ALRA's low-rank, non-negativity assumption: "we assume that the true expression matrix is non-negative, low-rank, and contains many zeros (biological zeros)"
