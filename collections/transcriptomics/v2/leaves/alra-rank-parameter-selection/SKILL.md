---
name: alra-rank-parameter-selection
description: Use when running ALRA (or a similar low-rank scRNA-seq imputation) on
  a library/log-normalized expression matrix and an automatic, data-driven rank k
  must be chosen instead of hand-tuning.
when_to_use_negative:
- You already have a biologically justified k for the dataset and want to supply it
  explicitly to alra(A_norm, k=...) rather than adaptively estimate it
- 'The input is raw counts not yet library/log-normalized (normalize_data: 10,000
  transcripts per cell, pseudocount 1, log-transform) — choose_k assumes the normalized
  matrix'
- The dataset's signal spectrum extends beyond the top-100 singular values, since
  the criterion only examines spacings among the top 100
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: ALRA
  role: R package supplying normalize_data for library/log normalization and choose_k
    for automatic rank selection; installed via devtools::install_github("KlugerLab/ALRA")
  repo: github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD package (the only prerequisite for ALRA) used by choose_k to
    compute the top-100 singular values with q = 2 power iterations
  repo: https://cran.r-project.org/package=rsvd
- name: rpca-mkl
  role: Optional MKL-based fast randomized SVD backend; enabled via the use.mkl flag
    for dramatic speedups over the default rsvd-based version
  repo: github.com/KlugerLab/rpca-mkl
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:fcd21ffe7ea78d5accd49de185a766fb6876097526edfb168ea66c48c0150e75
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALRA
  - ALRA choose_k
  - ALRA normalize_data
  - rpca-mkl
  - rsvd
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# alra-rank-parameter-selection

## Summary

ALRA's automatic selection of its only free parameter, the rank k of the low-rank approximation, via the spacings between the top-100 singular values rather than the spectrum itself.

## When to use

Use when running ALRA (or a similar low-rank scRNA-seq imputation) on a library/log-normalized expression matrix and an automatic, data-driven rank k must be chosen instead of hand-tuning.

## When NOT to use

- You already have a biologically justified k for the dataset and want to supply it explicitly to alra(A_norm, k=...) rather than adaptively estimate it
- The input is raw counts not yet library/log-normalized (normalize_data: 10,000 transcripts per cell, pseudocount 1, log-transform) — choose_k assumes the normalized matrix
- The dataset's signal spectrum extends beyond the top-100 singular values, since the criterion only examines spacings among the top 100

## Inputs

- Library/log-normalized scRNA-seq expression matrix (cells as rows, genes as columns), e.g. normalize_data(b_nk_example)
- Top-100 singular value vector from randomized SVD (rsvd, q = 2)

## Outputs

- Selected rank k (positive integer <= 100), e.g. k_choice$k
- Singular value vector k_choice$d (top 100)
- Per-dataset ranks comparable to paper-reported values: k = 26 purified PBMCs, k = 48 Hrvatin, k = 53 Tasic, k = 14 Drop-seq melanoma

## How to apply

First normalize the expression matrix with ALRA's normalize_data (scale each cell to alpha = 10,000 transcripts, add pseudocount 1, log-transform), since choose_k operates on the normalized matrix with cells as rows and genes as columns. Compute the top-100 singular values using the rsvd randomized SVD (q = 2 additional power iterations) via choose_k(A_norm), which returns k_choice$k and k_choice$d. Form the consecutive spacings s_i = sigma_i - sigma_{i-1} for i = 2..100, estimate the noise level as the mean mu and standard deviation sigma of the tail noise spacings s80..s100, and pick k as the largest index whose spacing exceeds mu + 6*sigma (bounded by k < 100). The rationale, per the README, is that k is 'chosen based on the spacings between the singular values, as it can be quite hard to identify the beginning of noise from just looking at the spectrum itself.'

## Related tools

- **ALRA** (R package supplying normalize_data for library/log normalization and choose_k for automatic rank selection; installed via devtools::install_github("KlugerLab/ALRA")) — github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD package (the only prerequisite for ALRA) used by choose_k to compute the top-100 singular values with q = 2 power iterations) — https://cran.r-project.org/package=rsvd
- **rpca-mkl** (Optional MKL-based fast randomized SVD backend; enabled via the use.mkl flag for dramatic speedups over the default rsvd-based version) — github.com/KlugerLab/rpca-mkl

## Examples

```
library(ALRA); A_norm <- normalize_data(b_nk_example); k_choice <- choose_k(A_norm); A_norm_completed <- alra(A_norm, k=k_choice$k)[[3]]
```

## Evaluation signals

- Returned k is a positive integer <= 100 (upper bound k < 100 is honored)
- Spacing criterion s_k > mu + 6*sigma is actually exercised, with mu and sigma computed from tail noise spacings s80..s100
- Selected k reproduces paper-reported ranks for the corresponding datasets: k = 26 purified PBMCs, k = 48 Hrvatin, k = 53 Tasic, k = 14 Drop-seq melanoma
- Spacings are computed as consecutive differences s_i = sigma_i - sigma_{i-1} for i = 2..100 of the top-100 rsvd singular values
- Downstream alra(A_norm, k=k_choice$k) runs without error on the normalized matrix

## Limitations

- The criterion only scans the top 100 singular values, so k is bounded by 100; very high-rank datasets cannot exceed this ceiling
- Automatic k 'worked quite well' for the paper's results, but the README advises taking a closer look with diagnostic plots (spectrum and spacing plots with ggplot2) in some cases
- Default rsvd-based computation can be slow; the faster MKL path (use.mkl) requires rpca-mkl, which is under development and not on CRAN, and its pre-compiled MKL libraries exist only for Linux and OS X (not Windows)

## Evidence

- [readme] Choose k based on singular-value spacings: "The k is chosen based on the spacings between the singular values, as it can be quite hard to identify the ``beginning of noise'' from just looking at the spectrum itself."
- [intro] Randomized SVD first step of ALRA: "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
- [intro] Non-negativity and low-rank structure assumption: "ALRA takes advantage of the non-negativity and low-rank structure of an expression matrix"
- [readme] Automatic k worked well for paper results: "For the results in the paper, automatically chosen k worked quite well, but in some cases you might want to take a closer look, as we do here."
- [readme] rsvd is the only prerequisite: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
