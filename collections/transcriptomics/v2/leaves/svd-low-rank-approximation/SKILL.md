---
name: svd-low-rank-approximation
description: Use when a normalized scRNA-seq count matrix (cells as rows, genes as
  columns) contains a large fraction of zeros (e.g. >90% with droplet-based data)
  and you need a low-rank approximation to complete technical zeros before adaptive
  thresholding.
when_to_use_negative:
- The input is not an scRNA-seq expression matrix or the cells/genes orientation is
  transposed — ALRA explicitly requires cells as rows and genes as columns
- The data is not assumed to be non-negative and low-rank (e.g. already fully denoised
  or a different modality where low-rank structure does not hold)
- All zeros must be treated as missing data and every gene imputed as expressed (e.g.
  when using a MAGIC/DCA-style approach is desired instead of zero-preserving imputation)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: rsvd
  role: Randomized SVD package used to compute the rank-k approximation and the top-100
    singular values for rank selection
  repo: https://github.com/KlugerLab/ALRA
- name: ALRA
  role: 'R package wrapping the full pipeline: normalize_data, choose_k, rank-k randomized-SVD
    approximation, and per-gene adaptive thresholding'
  repo: https://github.com/KlugerLab/ALRA
- name: fastRPCA (rpca-mkl)
  role: Optional Intel MKL-based, out-of-core randomized-SVD implementation used via
    ALRA's use.mkl flag for dramatic speedups on large matrices
  repo: https://github.com/KlugerLab/rpca-mkl
- name: Seurat
  role: Provides RunALRA() to apply the ALRA low-rank approximation and thresholding
    on a Seurat v3 object
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:84c6ee4ef1c4c7afa47c819ddb5d73798959bc853001a82aa43fa2ecf978c413
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - Seurat
  - fastRPCA (rpca-mkl)
  - rsvd
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# svd-low-rank-approximation

## Summary

Compute a rank-k approximation of a scRNA-seq expression matrix via (randomized) SVD as the core denoising step of ALRA, exploiting the non-negativity and low-rank structure of expression data so that subsequent thresholding can impute technical (dropout) zeros while preserving biological zeros.

## When to use

Use when a normalized scRNA-seq count matrix (cells as rows, genes as columns) contains a large fraction of zeros (e.g. >90% with droplet-based data) and you need a low-rank approximation to complete technical zeros before adaptive thresholding.

## When NOT to use

- The input is not an scRNA-seq expression matrix or the cells/genes orientation is transposed — ALRA explicitly requires cells as rows and genes as columns
- The data is not assumed to be non-negative and low-rank (e.g. already fully denoised or a different modality where low-rank structure does not hold)
- All zeros must be treated as missing data and every gene imputed as expressed (e.g. when using a MAGIC/DCA-style approach is desired instead of zero-preserving imputation)

## Inputs

- Raw scRNA-seq count matrix (e.g. the b_nk_example dataset), cells as rows and genes as columns
- Library- and log-normalized expression matrix A_norm produced by normalize_data

## Outputs

- Rank-k SVD approximation of the normalized expression matrix (input to adaptive thresholding)
- Chosen rank k with top-100 singular values from choose_k (k_choice$d, k_choice$k)
- Completed matrix A_norm_completed after thresholding and rescaling (alra_thresholded_matrix.csv)

## How to apply

First library- and log-normalize the raw count matrix with normalize_data so cells are rows and genes are columns (this orientation is required by ALRA). Estimate the rank k with choose_k, which computes the top 100 singular values via randomized SVD (rsvd::rsvd) and selects the largest k whose singular-value spacing exceeds a noise cutoff derived from spacings s80..s100 (k < 100). Compute the rank-k SVD approximation of the normalized matrix using randomized SVD — optionally with fastRPCA (KlugerLab/rpca-mkl) via ALRA's use.mkl flag for MKL-accelerated, out-of-core performance on large matrices. The rationale is that ALRA assumes the true expression matrix is non-negative, low-rank, and contains many biological zeros, so the low-rank approximation places entries at biological-zero positions approximately symmetrically around zero, which is the property the downstream per-gene thresholding (zeroing entries smaller in magnitude than each gene row's most negative value) relies on. For the paper's benchmarks, the automatically chosen k performed well, but inspect the singular-value and spacing plots when in doubt.

## Related tools

- **rsvd** (Randomized SVD package used to compute the rank-k approximation and the top-100 singular values for rank selection) — https://github.com/KlugerLab/ALRA
- **ALRA** (R package wrapping the full pipeline: normalize_data, choose_k, rank-k randomized-SVD approximation, and per-gene adaptive thresholding) — https://github.com/KlugerLab/ALRA
- **fastRPCA (rpca-mkl)** (Optional Intel MKL-based, out-of-core randomized-SVD implementation used via ALRA's use.mkl flag for dramatic speedups on large matrices) — https://github.com/KlugerLab/rpca-mkl
- **Seurat** (Provides RunALRA() to apply the ALRA low-rank approximation and thresholding on a Seurat v3 object) — https://github.com/satijalab/seurat

## Examples

```
library(ALRA); A_norm <- normalize_data(b_nk_example); k_choice <- choose_k(A_norm); A_norm_completed <- alra(A_norm, k=k_choice$k)[[3]]
```

## Evaluation signals

- Entries of the rank-k approximation at positions that are zero in the observed normalized matrix are approximately symmetrically distributed around zero for each gene
- The completed matrix retains zeros at biological-zero positions while imputing technical zeros — about 97% biological-zero preservation reported on the simulated benchmark
- choose_k's selected k is bounded by k < 100 and corresponds to the largest singular-value spacing exceeding the noise cutoff derived from s80..s100
- Matrix orientation check: aggregation over cell-type labels shows previously sparse markers (e.g. NCAM1 in NK cells, CR2 in B cells) gain nonzero values after completion
- Output matrix is rescaled and non-negative after the final rescaling step of alra()

## Limitations

- The pre-compiled MKL libraries in rpca-mkl are only included for Linux and OS X, so the fastRPCA accelerated path does not install on Windows without compiling from source
- The automatically chosen k from choose_k worked well for the paper's results but may need manual inspection in some cases
- rpca-mkl is still under development, is not on CRAN, and is not a required package; ALRA defaults to the rpda-based version if use.mkl is not enabled
- The out-of-core ('blocked') functionality of rpca-mkl has not been appropriately tested

## Evidence

- [intro] SVD as first ALRA step: "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
- [intro] biological zeros symmetric around zero: "the entries of this matrix that correspond to the true biological zeros are symmetrically distributed around zero"
- [readme] README pipeline description: "ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene."
- [readme] input orientation requirement: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] rsvd prerequisite: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
