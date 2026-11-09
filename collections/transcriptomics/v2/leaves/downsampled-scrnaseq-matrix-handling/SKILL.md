---
name: downsampled-scrnaseq-matrix-handling
description: Use when you need to compare zero-preserving imputation methods on scRNA-seq
  count matrices by quantifying whether imputation improves cell-type separability
  via out-of-bag classification error.
when_to_use_negative:
- Datasets exceeding 100,000 cells where SAVER and scImpute become extremely slow
  — use scalable methods instead
- Tasks where the goal is to preserve true biological zeros at zero (MAGIC and DCA
  output a matrix in which every gene is expressed in every cell)
- Already-normalized or already-imputed matrices — this skill expects raw downsampled
  count matrices plus labels
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: ALRA
  role: Primary imputation method; normalize_data for library/log normalization, choose_k
    for rank selection, alra for adaptively thresholded low-rank completion with quantile.prob
    = 0.001
  repo: https://github.com/KlugerLab/ALRA
- name: MAGIC
  role: Baseline imputer; negative values set to zero before comparison
- name: SAVER
  role: Baseline imputer run with do.fast = TRUE on the same downsampled matrices
- name: DCA
  role: Baseline deep-count autoencoder imputer on the same matrices
- name: scImpute
  role: Baseline imputer (run on a subset where needed for tractability); error increased
    to 28%
- name: ranger
  role: Random-forest classifier trained with default settings to compute out-of-bag
    classification error on raw vs imputed matrices
- name: Seurat
  role: Single-cell toolkit context; ALRA is integrated as RunALRA() in Seurat v3.0
  repo: https://github.com/satijalab/seurat
- name: rpca-mkl / fastRPCA
  role: Optional MKL-accelerated randomized SVD backend enabling dramatic speedups
    via ALRA's use.mkl flag
  repo: https://github.com/KlugerLab/rpca-mkl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:63d69220bf92dd93c6dea4a7e7088aa579df30ffc8d62eb8c22d645d5c79ec6d
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
  - rpca-mkl / fastRPCA
  - scImpute
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# downsampled-scrnaseq-matrix-handling

## Summary

Protocol for loading and benchmarking imputation methods (ALRA, MAGIC, SAVER, DCA, scImpute) on downsampled scRNA-seq expression matrices (Zeisel, Baron, Chen, La Manno) using random-forest out-of-bag classification error as the evaluation metric.

## When to use

Use when you need to compare zero-preserving imputation methods on scRNA-seq count matrices by quantifying whether imputation improves cell-type separability via out-of-bag classification error.

## When NOT to use

- Datasets exceeding 100,000 cells where SAVER and scImpute become extremely slow — use scalable methods instead
- Tasks where the goal is to preserve true biological zeros at zero (MAGIC and DCA output a matrix in which every gene is expressed in every cell)
- Already-normalized or already-imputed matrices — this skill expects raw downsampled count matrices plus labels

## Inputs

- Downsampled scRNA-seq expression matrices (Zeisel 1799 cells, Baron 1076 cells, Chen 7712 cells, La Manno 947 cells) distributed with Huang et al. 2018 (SAVER)
- Cell-identity class labels from the original authors for each dataset

## Outputs

- Normalized and imputed expression matrices for ALRA, MAGIC, SAVER, DCA, and scImpute per dataset
- Per-dataset, per-method out-of-bag random-forest classification error values
- Comparison table oob_error_table.csv (average raw error 16.4% vs ALRA 8%, MAGIC 10.3%, SAVER 9.8%, DCA 9.4%, scImpute 28%)

## How to apply

Load the four downsampled expression matrices distributed with Huang et al. 2018 (SAVER) — Zeisel (1799 cells), Baron (1076 cells), Chen (7712 cells), La Manno (947 cells) — together with the original cell-identity labels. Normalize each matrix by library normalization to 10,000 counts and log-transformation (ALRA's normalize_data), with cells as rows and genes as columns as required by alra(). Impute with ALRA letting choose_k automatically select the rank (expected k = 12, 21, 15, 16 for Baron, Chen, La Manno, Zeisel respectively) using alra with quantile.prob = 0.001, and run baselines on the same matrices: MAGIC (with negative values set to zero), SAVER (do.fast = TRUE), DCA, and scImpute (on a subset where needed for tractability). Train a ranger random forest with default settings to predict cell class from expression values on each raw and imputed matrix, record out-of-bag error per dataset/method, average across the four datasets, and write oob_error_table.csv.

## Related tools

- **ALRA** (Primary imputation method; normalize_data for library/log normalization, choose_k for rank selection, alra for adaptively thresholded low-rank completion with quantile.prob = 0.001) — https://github.com/KlugerLab/ALRA
- **MAGIC** (Baseline imputer; negative values set to zero before comparison)
- **SAVER** (Baseline imputer run with do.fast = TRUE on the same downsampled matrices)
- **DCA** (Baseline deep-count autoencoder imputer on the same matrices)
- **scImpute** (Baseline imputer (run on a subset where needed for tractability); error increased to 28%)
- **ranger** (Random-forest classifier trained with default settings to compute out-of-bag classification error on raw vs imputed matrices)
- **Seurat** (Single-cell toolkit context; ALRA is integrated as RunALRA() in Seurat v3.0) — https://github.com/satijalab/seurat
- **rpca-mkl / fastRPCA** (Optional MKL-accelerated randomized SVD backend enabling dramatic speedups via ALRA's use.mkl flag) — https://github.com/KlugerLab/rpca-mkl

## Examples

```
library(ALRA); A_norm <- normalize_data(b_nk_example); k_choice <- choose_k(A_norm); A_norm_completed <- alra(A_norm, k=k_choice$k, quantile.prob=0.001)[[3]]
```

## Evaluation signals

- Out-of-bag error on raw matrices should be near 16.4% on average across the four datasets, and ALRA-imputed near 8%
- choose_k should return k ≈ 12, 21, 15, 16 for Baron, Chen, La Manno, and Zeisel respectively
- ALRA output should remain non-negative after thresholding and preserve true biological zeros, unlike MAGIC/DCA which impute every gene as expressed in every cell
- scImpute as a negative control should increase average OOB error to ~28%, confirming the benchmark can detect degradation
- oob_error_table.csv exists with one error value per dataset × method combination

## Limitations

- MAGIC and DCA treat all zeros as missing data and output a matrix in which every gene is expressed in every cell, so they cannot preserve biological zeros
- SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells, restricting scImpute to subsets here
- ALRA is supported/tested on OS X, Linux, and Windows; rpca-mkl pre-compiled MKL libraries are Linux/OS X only and its Windows install requires compiling from source with Intel MKL
- Results reflect downsampled datasets; the fraction of zeros can exceed 90% in full droplet-based datasets

## Evidence

- [intro] ALRA imputation improved cell-type separation: "Some methods such as DCA and MAGIC treat all zeros as “missing data” and output a matrix in which every gene is expressed in every cell"
- [intro] scalability limitation: "SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells"
- [readme] ALRA usage and normalization: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] normalization step: "We use library and log normalization, but other approaches may also work well."
- [readme] rank selection rationale: "The k is chosen based on the spacings between the singular values, as it can be quite hard to identify the ``beginning of noise'' from just looking at the spectrum itself."
- [readme] MKL acceleration option: "The functions now have a flag `use.mkl` for users who have installed [rpca-mkl](https://github.com/KlugerLab/rpca-mkl), which allows for dramatic speedups over the default rpca-based version."
