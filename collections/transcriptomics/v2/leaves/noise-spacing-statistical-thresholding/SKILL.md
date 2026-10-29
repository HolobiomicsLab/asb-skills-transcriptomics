---
name: noise-spacing-statistical-thresholding
description: Use when you must choose the rank k of a randomized-SVD low-rank approximation
  of a normalized scRNA-seq matrix and cannot identify the 'beginning of noise' by
  eye from the spectrum; 50–300 chars ok.
when_to_use_negative:
- You already have a biologically justified fixed rank for the SVD (e.g. replicating
  a paper's reported k) and do not need automatic selection
- You have fewer than 100 singular values or want k near or above 100 — the criterion
  is defined only on the top-100 spectrum with the upper bound k < 100
- You are thresholding the completed matrix entries themselves (ALRA's per-gene magnitude
  thresholding step), not selecting the SVD rank
edam_operation: http://edamontology.org/operation_0306
tools:
- name: ALRA choose_k
  role: Computes the top-100 singular values (rsvd, q = 2) and applies the mu + 6*sigma
    spacing criterion to return k_choice$k
  repo: https://github.com/KlugerLab/ALRA
- name: ALRA normalize_data
  role: Produces the library/log-normalized input matrix (alpha = 10,000 transcripts,
    pseudocount 1, log-transform) on which choose_k is run
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD backend used by choose_k to compute the top-100 singular values
    with q = 2 additional power iterations
  repo: https://github.com/KlugerLab/ALRA
- name: rpca-mkl (fastRPCA)
  role: Optional MKL-accelerated randomized SVD backend; enable via ALRA's use.mkl
    flag for speedups over the default rpca-based version
  repo: KlugerLab/rpca-mkl
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:35947eb2995e811e365d11a8097aa4c7ce55061366f2701f5cd43cc675379e41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA choose_k
  - ALRA normalize_data
  - rpca-mkl (fastRPCA)
  - rsvd
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# noise-spacing-statistical-thresholding

## Summary

Automatically selects the rank k for ALRA's low-rank approximation by testing the spacings between the top 100 singular values against a statistical threshold: the largest spacing exceeding mu + 6*sigma, where mu and sigma are estimated from the tail (s80–s100) noise spacings. This replaces visual inspection of the singular-value spectrum with a data-driven cutoff.

## When to use

Use when you must choose the rank k of a randomized-SVD low-rank approximation of a normalized scRNA-seq matrix and cannot identify the 'beginning of noise' by eye from the spectrum; 50–300 chars ok.

## When NOT to use

- You already have a biologically justified fixed rank for the SVD (e.g. replicating a paper's reported k) and do not need automatic selection
- You have fewer than 100 singular values or want k near or above 100 — the criterion is defined only on the top-100 spectrum with the upper bound k < 100
- You are thresholding the completed matrix entries themselves (ALRA's per-gene magnitude thresholding step), not selecting the SVD rank

## Inputs

- Normalized scRNA-seq expression matrix A_norm (library and log normalized; cells as rows, genes as columns), e.g. from normalize_data(b_nk_example)
- Top-100 singular values of A_norm computed with rsvd using q = 2 power iterations (choose_k returns k_choice$k and k_choice$d)

## Outputs

- Integer rank k (1 <= k < 100) for the ALRA rank-k approximation
- Vector of the top-100 singular values and their consecutive spacing vector s_i = sigma_i - sigma_{i-1} (i = 2..100)
- Noise statistics mu and sigma estimated from spacings s80..s100; e.g. written to rank_selection.csv with per-dataset k (k = 26 purified PBMCs, 48 Hrvatin, 53 Tasic, 14 Drop-seq melanoma)

## How to apply

Run ALRA's choose_k on the library/log-normalized matrix (cells as rows, genes as columns; normalize each cell to alpha = 10,000 transcripts, add pseudocount 1, log-transform) to obtain the top-100 singular values via rsvd with q = 2 additional power iterations. Compute consecutive spacings s_i = sigma_i - sigma_{i-1} for i = 2..100, estimate the noise level as the mean mu and standard deviation sigma of the tail spacings s80..s100, then set k to the largest index with s_k > mu + 6*sigma, honoring the upper bound k < 100. The rationale, per the ALRA README, is that it is hard to identify the 'beginning of noise' from the spectrum itself, but spacing statistics make the noise regime detectable. Use the chosen k when calling alra(A_norm, k=k_choice$k); the paper's automatically chosen k performed well, but inspect the spectrum/spacings plot in ambiguous cases.

## Related tools

- **ALRA choose_k** (Computes the top-100 singular values (rsvd, q = 2) and applies the mu + 6*sigma spacing criterion to return k_choice$k) — https://github.com/KlugerLab/ALRA
- **ALRA normalize_data** (Produces the library/log-normalized input matrix (alpha = 10,000 transcripts, pseudocount 1, log-transform) on which choose_k is run) — https://github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD backend used by choose_k to compute the top-100 singular values with q = 2 additional power iterations) — https://github.com/KlugerLab/ALRA
- **rpca-mkl (fastRPCA)** (Optional MKL-accelerated randomized SVD backend; enable via ALRA's use.mkl flag for speedups over the default rpca-based version) — KlugerLab/rpca-mkl

## Examples

```
library(ALRA); A_norm <- normalize_data(b_nk_example); k_choice <- choose_k(A_norm); k <- k_choice$k
```

## Evaluation signals

- Returned k is a positive integer with k < 100 (the upper bound imposed by the tail-noise window)
- The spacing criterion is exercised: the selected index satisfies s_k > mu + 6*sigma where mu, sigma come from spacings s80..s100
- Recompute spacings s_i = sigma_i - sigma_{i-1} from k_choice$d and confirm the argmax above threshold matches k_choice$k
- Per-dataset k values match paper-reported ranks: k = 26 (purified PBMCs), k = 48 (Hrvatin), k = 53 (Tasic), k = 14 (Drop-seq melanoma)
- Inspection of the singular-value spectrum and spacing plots places the vertical cutoff line (k_choice$k) at the 'beginning of noise'

## Limitations

- The criterion is capped at the top-100 singular values, so it cannot detect ranks above 100
- The README cautions that the automatic k 'worked quite well' in the paper but in some cases you might want to take a closer look — manual inspection of the spectrum/spacings is advised in ambiguous cases
- It is designed for library/log-normalized scRNA-seq data (cells as rows); other normalizations 'may also work well' but are not the validated setting

## Evidence

- [readme] spacing criterion from tail noise: "The k is chosen based on the spacings between the singular values, as it can be quite hard to identify the ``beginning of noise'' from just looking at the spectrum itself."
- [readme] manual inspection caveat: "For the results in the paper, automatically chosen k worked quite well, but in some cases you might want to take a closer look, as we do here."
- [abstract] low-rank approximation context: "We present a method based on low-rank matrix approximation which imputes these values while preserving biologically non-expressed genes (true biological zeros) at zero expression levels."
