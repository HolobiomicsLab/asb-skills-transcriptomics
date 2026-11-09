---
name: matrix-normalization-for-imputation
description: Use when a raw scRNA-seq count matrix (e.g. Zeisel, Baron, Chen, La Manno
  downsampled matrices) is about to be imputed with ALRA or benchmarked against MAGIC/SAVER/DCA/scImpute
  and must first be library-normalized and log-transformed.
when_to_use_negative:
- Input is already library-normalized and log-transformed (e.g. a Seurat-normalized
  data slot) — re-normalizing would double-scale the data
- The downstream method requires raw integer counts (e.g. DCA, which models count
  distributions) rather than log-normalized values
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ALRA
  role: Provides normalize_data (library + log normalization), choose_k (rank selection),
    and alra (thresholded low-rank completion) applied to the normalized matrix
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Prerequisite package supplying the randomized SVD used by ALRA's low-rank
    approximation
- name: rpca-mkl
  role: Optional MKL-accelerated randomized SVD backend enabled via ALRA's use.mkl
    flag for dramatic speedups
  repo: https://github.com/KlugerLab/rpca-mkl
- name: Seurat
  role: Integrates the normalized-imputation workflow via RunALRA() (or alraSeurat2.R
    for Seurat v2 objects)
  repo: https://github.com/satijalab/seurat
- name: ranger
  role: Random-forest classifier trained on raw vs imputed matrices to compute out-of-bag
    error for benchmarking
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:9818efbed659d81011d286ad1220d59f79273442969312228722061938284bd2
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
  - rsvd
  - scImpute
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# matrix-normalization-for-imputation

## Summary

Library-size normalization to 10,000 counts followed by log-transformation (ALRA's normalize_data) is the required preprocessing that converts a raw scRNA-seq count matrix into the normalized matrix (cells as rows, genes as columns) that ALRA-style low-rank imputation expects.

## When to use

Use when a raw scRNA-seq count matrix (e.g. Zeisel, Baron, Chen, La Manno downsampled matrices) is about to be imputed with ALRA or benchmarked against MAGIC/SAVER/DCA/scImpute and must first be library-normalized and log-transformed.

## When NOT to use

- Input is already library-normalized and log-transformed (e.g. a Seurat-normalized data slot) — re-normalizing would double-scale the data
- The downstream method requires raw integer counts (e.g. DCA, which models count distributions) rather than log-normalized values

## Inputs

- Raw scRNA-seq count/expression matrix (cells x genes) from downsampled datasets such as Zeisel (1799 cells), Baron (1076 cells), Chen (7712 cells), or La Manno (947 cells)
- Optional cell-identity labels used downstream for classification

## Outputs

- Log-normalized expression matrix (A_norm) with 10,000-count library scaling, suitable as direct input to alra()/choose_k()
- ALRA-completed matrix (A_norm_completed) preserving biological zeros at zero

## How to apply

Orient the count matrix so cells are rows and genes are columns, as ALRA's README explicitly requires. Apply library normalization to 10,000 counts per cell and log-transform using ALRA's normalize_data, since ALRA assumes the true expression matrix is non-negative, low-rank, and contains many biological zeros. After normalization, let choose_k automatically select the SVD rank (expected k = 12, 21, 15, 16 for Baron, Chen, La Manno, Zeisel respectively) before calling alra with quantile.prob = 0.001 to impute technical zeros while thresholding to restore biological zeros. This same normalized matrix serves as the raw baseline input when benchmarking imputers (MAGIC, SAVER, DCA, scImpute) by ranger random-forest out-of-bag classification error.

## Related tools

- **ALRA** (Provides normalize_data (library + log normalization), choose_k (rank selection), and alra (thresholded low-rank completion) applied to the normalized matrix) — https://github.com/KlugerLab/ALRA
- **rsvd** (Prerequisite package supplying the randomized SVD used by ALRA's low-rank approximation)
- **rpca-mkl** (Optional MKL-accelerated randomized SVD backend enabled via ALRA's use.mkl flag for dramatic speedups) — https://github.com/KlugerLab/rpca-mkl
- **Seurat** (Integrates the normalized-imputation workflow via RunALRA() (or alraSeurat2.R for Seurat v2 objects)) — https://github.com/satijalab/seurat
- **ranger** (Random-forest classifier trained on raw vs imputed matrices to compute out-of-bag error for benchmarking)

## Examples

```
library(ALRA); A_norm <- normalize_data(A_raw); k_choice <- choose_k(A_norm); A_norm_completed <- alra(A_norm, k = k_choice$k, quantile.prob = 0.001)[[3]]
```

## Evaluation signals

- Matrix orientation check: cells are rows and genes are columns before calling alra, per the README requirement
- Normalized values are log-transformed with column sums corresponding to 10,000-count library scaling and are non-negative
- choose_k returns expected ranks (k = 12 Baron, 21 Chen, 15 La Manno, 16 Zeisel) on the downsampled datasets
- Completed matrix preserves zeros for known biological non-expressers while NCAM1 becomes expressed in all NK cells and CR2 in all B cells in the b_nk_example
- Downstream sanity signal: ranger out-of-bag error drops from 16.4% (raw) toward 8% after ALRA imputation, vs 28% for scImpute

## Limitations

- Automatically chosen k via singular-value spacings 'worked quite well' in the paper but may need manual inspection per dataset
- Completion results improve when using the entire dataset rather than subsets, as noted for the b_nk_example
- ALRA's quantile thresholding assumes entries corresponding to true biological zeros are symmetrically distributed around zero, which can fail if the low-rank/non-negativity assumption does not hold

## Evidence

- [readme] Cells-as-rows orientation requirement: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] Library and log normalization usage: "# Let A_norm be a normalized expression matrix where cells are rows and genes are columns.
# We use library and log normalization, but other approaches may also work well."
- [intro] Preservation of biological zeros as design goal: "it should accurately impute the data, in particular preserving at zero true biological zeros while completing the technical ones"
- [intro] Thresholding restores biological zeros after SVD: "the last step of ALRA is to restore the biological zeros in the matrix by thresholding its entries"
