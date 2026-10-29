---
name: randomized-svd-computation
description: Use when a large normalized single-cell expression matrix (cells as rows,
  genes as columns) needs a rank-k SVD, or when the low-rank parameter k must be selected
  automatically from the top-100 singular-value spectrum.
when_to_use_negative:
- The input matrix is not normalized (raw counts) — library/log normalization should
  precede the SVD so the spectrum reflects the modeled expression scale
- The goal is a full-rank or exact SVD of a small matrix; randomized SVD is designed
  for fast rank-k approximation of large matrices
- Imputation methods that do not rely on low-rank structure (e.g. DCA, MAGIC, SAVER,
  scImpute) are required instead
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: RSVD (rsvd R package)
  role: Computes the randomized SVD used by ALRA for the rank-k approximation and
    for choose_k's top-100 singular values (q = 2 power iterations)
- name: ALRA
  role: Hosts normalize_data, choose_k, and alra, which orchestrate normalization,
    randomized-SVD-based rank selection, and adaptive thresholding
  repo: github.com/KlugerLab/ALRA
- name: rpca-mkl (fastRPCA)
  role: Optional Intel MKL-based, out-of-core randomized SVD backend enabling dramatic
    speedups when ALRA's use.mkl flag is set
  repo: KlugerLab/rpca-mkl
- name: Seurat
  role: Provides RunALRA() so the randomized-SVD-based imputation can be invoked on
    Seurat objects
  repo: satijalab/seurat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:e342d95895d139bdf727bc9ace1bbb04921c143a7a30e93a19600bed4c0744ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALRA
  - RSVD (rsvd R package)
  - Seurat
  - rpca-mkl (fastRPCA)
  - rsvd
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# randomized-svd-computation

## Summary

Computes the rank-k randomized singular value decomposition of a large scRNA-seq expression matrix, as the computational core of ALRA's low-rank approximation and its automatic rank selection (choose_k) based on singular-value spacings.

## When to use

Use when a large normalized single-cell expression matrix (cells as rows, genes as columns) needs a rank-k SVD, or when the low-rank parameter k must be selected automatically from the top-100 singular-value spectrum.

## When NOT to use

- The input matrix is not normalized (raw counts) — library/log normalization should precede the SVD so the spectrum reflects the modeled expression scale
- The goal is a full-rank or exact SVD of a small matrix; randomized SVD is designed for fast rank-k approximation of large matrices
- Imputation methods that do not rely on low-rank structure (e.g. DCA, MAGIC, SAVER, scImpute) are required instead

## Inputs

- Normalized scRNA-seq expression matrix (cells as rows, genes as columns), e.g. A_norm from normalize_data(b_nk_example)
- Optional parameters: number of top singular values (default 100), power iterations q = 2, target rank k

## Outputs

- Rank-k low-rank approximation (left/right singular vectors and singular values) consumed by alra()
- k_choice object containing the selected rank k and the singular-value vector d (top 100)
- Optionally, singular-value spacing vector s_i = sigma_i - sigma_{i-1} used for the mu + 6*sigma criterion

## How to apply

Normalize the expression matrix first (ALRA's normalize_data: scale each cell to alpha = 10,000 transcripts, add a pseudocount of 1, log-transform), then compute the top-100 singular values with the RSVD package using q = 2 additional power iterations for accuracy. To choose the rank, compute consecutive spacings s_i = sigma_i - sigma_{i-1} for i = 2..100, estimate the noise spacings' mean mu and standard deviation sigma from s80..s100, and take k as the largest index with s_k > mu + 6*sigma (k < 100). Rationale: the spectrum alone makes the 'beginning of noise' hard to identify, whereas a large spacing separates signal singular values from the noise tail. The resulting rank-k approximation feeds ALRA's thresholding step that restores biological zeros; for speed, rpca-mkl can be used via ALRA's use.mkl flag.

## Related tools

- **RSVD (rsvd R package)** (Computes the randomized SVD used by ALRA for the rank-k approximation and for choose_k's top-100 singular values (q = 2 power iterations))
- **ALRA** (Hosts normalize_data, choose_k, and alra, which orchestrate normalization, randomized-SVD-based rank selection, and adaptive thresholding) — github.com/KlugerLab/ALRA
- **rpca-mkl (fastRPCA)** (Optional Intel MKL-based, out-of-core randomized SVD backend enabling dramatic speedups when ALRA's use.mkl flag is set) — KlugerLab/rpca-mkl
- **Seurat** (Provides RunALRA() so the randomized-SVD-based imputation can be invoked on Seurat objects) — satijalab/seurat

## Examples

```
library(ALRA); A_norm <- normalize_data(b_nk_example); k_choice <- choose_k(A_norm); A_norm_completed <- alra(A_norm, k=k_choice$k)[[3]]
```

## Evaluation signals

- k_choice$k is a positive integer <= 100 and k_choice$d contains exactly 100 singular values
- The chosen k satisfies s_k > mu + 6*sigma with mu, sigma estimated from noise spacings s80..s100
- Reported ranks are consistent with paper-reported values for corresponding datasets (k = 26 purified PBMCs, k = 48 Hrvatin, k = 53 Tasic, k = 14 Drop-seq melanoma)
- Re-running with rpca-mkl (use.mkl = TRUE) yields the same rank selection and approximation up to randomized-SVD tolerance
- The low-rank approximation, after ALRA thresholding, preserves biological zeros at zero while completing technical zeros

## Limitations

- Automatic k works well for the paper's datasets but users may want to inspect the spectrum, as noted in the ALRA README example
- Rank selection is bounded at k < 100, so datasets with higher intrinsic rank fall back to the spacing heuristic within the top 100
- rpca-mkl is under development, not on CRAN, and its pre-compiled MKL libraries are only for Linux and OS X (not Windows)
- Randomized SVD gives an approximate rank-k factorization, so results can vary slightly across runs without a fixed seed

## Evidence

- [readme] ALRA first computes its rank-k approximation using randomized SVD.: "Given a scRNA-seq expression matrix, ALRA first computes its rank-k approximation using randomized SVD."
- [readme] The only prerequisite for ALRA is the randomized SVD package RSVD.: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
- [readme] The k is chosen based on the spacings between the singular values.: "The k is
chosen based on the spacings between the singular values, as it can be quite
hard to identify the ``beginning of noise'' from just looking at the spectrum
itself."
- [intro] The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular value decomposition (SVD).: [evidence span withheld — anchor status: not_found]
