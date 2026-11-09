---
name: zero-preserving-low-rank-matrix-imputation
description: Use when a scRNA-seq normalized expression matrix shows extreme sparsity/dropout
  (zeros can exceed 90% with droplet techniques) and you need marker-gene recovery
  that preserves biological zeros at exactly zero.
when_to_use_negative:
- When all zeros should be treated as missing data — methods like DCA or MAGIC fill
  every entry, whereas ALRA deliberately keeps true biological zeros at zero
- When the input is not a normalized gene-by-cell scRNA-seq matrix (ALRA assumes non-negativity,
  low-rank structure, and many biological zeros)
- When you need only one of the alternate imputers (SAVER, scImpute) — note the article
  reports those can be extremely slow on datasets over 100,000 cells
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ALRA
  role: 'Core R package implementing adaptively thresholded low-rank approximation:
    normalize_data, choose_k, and alra functions for zero-preserving imputation'
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD package (only prerequisite of ALRA) used to compute the rank-k
    approximation and the top 100 singular values in choose_k
  repo: https://github.com/KlugerLab/ALRA
- name: rpca-mkl (fastRPCA)
  role: Optional MKL-accelerated randomized SVD backend enabling dramatic speedups
    via the use.mkl flag; not on CRAN and not required
  repo: https://github.com/KlugerLab/rpca-mkl
- name: Seurat
  role: Single-cell toolkit exposing ALRA as RunALRA() in Seurat v3.0+; alraSeurat2.R
    provides a wrapper for Seurat v2 objects
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:5a17ac0c9236d6eb907a40d72797556e3bc0774017382264514362f4bf99d1c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - Seurat
  - rpca-mkl (fastRPCA)
  - rsvd
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# zero-preserving-low-rank-matrix-imputation

## Summary

ALRA (Adaptively thresholded Low-Rank Approximation) imputes technical (false) zeros in scRNA-seq count matrices via randomized-SVD low-rank approximation followed by per-gene adaptive thresholding, restoring true biological zeros at exactly zero. It recovers marker-gene expression (e.g., NCAM1, CR2) that dropout hides in raw data while remaining fast on very large datasets.

## When to use

Use when a scRNA-seq normalized expression matrix shows extreme sparsity/dropout (zeros can exceed 90% with droplet techniques) and you need marker-gene recovery that preserves biological zeros at exactly zero.

## When NOT to use

- When all zeros should be treated as missing data — methods like DCA or MAGIC fill every entry, whereas ALRA deliberately keeps true biological zeros at zero
- When the input is not a normalized gene-by-cell scRNA-seq matrix (ALRA assumes non-negativity, low-rank structure, and many biological zeros)
- When you need only one of the alternate imputers (SAVER, scImpute) — note the article reports those can be extremely slow on datasets over 100,000 cells

## Inputs

- scRNA-seq count matrix (e.g., the b_nk_example dataset in the ALRA package), cells as rows and genes as columns
- Cell-type label vector (e.g., labels_example) for per-population marker verification
- Marker gene names to track across imputation (e.g., NCAM1, CR2)

## Outputs

- Completed, zero-preserved normalized expression matrix A_norm_completed (alra result.completed[[3]])
- Chosen rank k from choose_k (k_choice$k) with the top-100 singular values
- Per cell-type x gene percentage-nonzero table before vs. after imputation (e.g., marker_expression_fractions.csv)

## How to apply

Pass ALRA a matrix with cells as rows and genes as columns, after library and log normalization (normalize_data). Compute a rank-k approximation with randomized SVD, choosing k automatically via choose_k, which computes the top 100 singular values with rsvd (q = 2 power iterations) and selects the largest k whose singular-value spacing exceeds mu + 6*sigma of the noise spacings. Run alra(A_norm, k=k_choice$k); ALRA then thresholds each gene at the magnitude of its most negative value — justified because entries corresponding to true biological zeros are symmetrically distributed around zero — and rescales the matrix. Extract the completed matrix as result.completed[[3]] (A_norm_completed) and compare per-cell-type percent-nonzero (100*sum(x>0)/length(x)) before and after imputation for marker genes. In the bundled b_nk_example, NCAM1 in NK cells rises from ~4% and CR2 in B cells from ~1% after completion, so post-imputation percentages in that high range signal success.

## Related tools

- **ALRA** (Core R package implementing adaptively thresholded low-rank approximation: normalize_data, choose_k, and alra functions for zero-preserving imputation) — https://github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD package (only prerequisite of ALRA) used to compute the rank-k approximation and the top 100 singular values in choose_k) — https://github.com/KlugerLab/ALRA
- **rpca-mkl (fastRPCA)** (Optional MKL-accelerated randomized SVD backend enabling dramatic speedups via the use.mkl flag; not on CRAN and not required) — https://github.com/KlugerLab/rpca-mkl
- **Seurat** (Single-cell toolkit exposing ALRA as RunALRA() in Seurat v3.0+; alraSeurat2.R provides a wrapper for Seurat v2 objects) — https://github.com/satijalab/seurat

## Examples

```
library(ALRA); data(b_nk_example); data(labels_example); A_norm <- normalize_data(b_nk_example); k_choice <- choose_k(A_norm); A_norm_completed <- alra(A_norm, k=k_choice$k)[[3]]
```

## Evaluation signals

- Marker recovery: percent of cells with nonzero NCAM1 in NK cells increases substantially from ~4% after imputation; CR2 in B cells from ~1%
- choose_k diagnostics: the singular-value spacing plot should place the vertical cutoff at k_choice$k, where spacings fall below mu + 6*sigma of the noise spacings
- Biological-zero preservation: genes that were truly non-expressed remain at exactly zero in A_norm_completed after thresholding
- Output schema check: the aggregate table contains four cell-type x gene percentage groups with the required fields, and the CSV (marker_expression_fractions.csv) exists
- Result extraction correctness: the completed matrix is taken as result.completed[[3]]

## Limitations

- Results improve when using the entire dataset rather than the bundled subset (see the ALRA-paper repo for those codes)
- The automatic k choice usually works well but in some cases you might want to take a closer look at the singular-value spectrum and spacings
- MKL acceleration via rpca-mkl is not on CRAN, is still under development, and its pre-compiled libraries are only included for Linux and OS X (not Windows)
- Only the randomized-SVD package RSVD is required; other normalization schemes beyond library/log normalization 'may also work well' but are not guaranteed

## Evidence

- [abstract] Low-rank approximation preserves biological zeros: "We present a method based on low-rank matrix approximation which imputes these values while preserving biologically non-expressed genes (true biological zeros) at zero expression levels."
- [intro] Thresholding rationale via symmetry around zero: "the entries of this matrix that correspond to the true biological zeros are symmetrically distributed around zero"
- [intro] Two-step ALRA procedure: "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
- [readme] README usage: cells as rows, completed matrix is element 3: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] README example: expected marker recovery percentages: "NCAM1 should be expressed in all NK cells, but is only expressed in 4% of of NK cells in the original data."
