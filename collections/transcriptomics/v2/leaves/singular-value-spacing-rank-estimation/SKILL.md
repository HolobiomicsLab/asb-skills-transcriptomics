---
name: singular-value-spacing-rank-estimation
description: Use when a rank-k SVD approximation of a library/log-normalized scRNA-seq
  expression matrix is needed and k must be chosen automatically from the data rather
  than fixed by hand.
when_to_use_negative:
- The matrix is raw counts that have not been library/log normalized — choose_k expects
  a normalized matrix such as the output of normalize_data
- The user already has a domain-driven k and only wants the completed matrix — alra(A_norm,
  k=...) can be called directly without the spacing criterion
- The data are not a low-rank non-negative expression matrix (ALRA's core assumption),
  e.g. already-denoised or feature-selected tables where singular-value tail structure
  is not meaningful
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: ALRA
  role: Provides the choose_k and normalize_data functions implementing the singular-value-spacing
    rank selection; install via devtools::install_github("KlugerLab/ALRA")
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD package used by choose_k to compute the top 100 singular values
    with q = 2 power iterations; the only prerequisite of ALRA (install.packages('rsvd'))
- name: rpca-mkl
  role: Optional MKL-accelerated randomized SVD backend usable via ALRA's use.mkl
    flag for dramatic speedups; not required
  repo: https://github.com/KlugerLab/rpca-mkl
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:da26374b494c86c9024095668d504df7a1dfdc3c94afb63b244ddb93efcb4a9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - rpca-mkl
  - rsvd
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# singular-value-spacing-rank-estimation

## Summary

Automatically selects the rank k for ALRA's low-rank approximation of a normalized scRNA-seq matrix by finding the largest singular-value spacing that exceeds mu + 6*sigma, where mu and sigma are estimated from the noise spacings s80..s100 of the top 100 singular values computed with randomized SVD (rsvd, q = 2).

## When to use

Use when a rank-k SVD approximation of a library/log-normalized scRNA-seq expression matrix is needed and k must be chosen automatically from the data rather than fixed by hand.

## When NOT to use

- The matrix is raw counts that have not been library/log normalized — choose_k expects a normalized matrix such as the output of normalize_data
- The user already has a domain-driven k and only wants the completed matrix — alra(A_norm, k=...) can be called directly without the spacing criterion
- The data are not a low-rank non-negative expression matrix (ALRA's core assumption), e.g. already-denoised or feature-selected tables where singular-value tail structure is not meaningful

## Inputs

- Normalized scRNA-seq expression matrix (cells as rows, genes as columns), e.g. A_norm from normalize_data applied to a count matrix like the ALRA b_nk_example dataset

## Outputs

- Selected rank k (positive integer <= 100), e.g. k_choice$k
- Vector of the top 100 singular values, e.g. k_choice$d
- Optional: singular-value and spacing vectors written to a table (e.g. rank_selection.csv) for comparison against paper-reported ranks (k = 26 purified PBMCs, k = 48 Hrvatin, k = 53 Tasic, k = 14 Drop-seq melanoma)

## How to apply

First normalize the count matrix (cells as rows, genes as columns) with ALRA's normalize_data, which scales each cell to alpha = 10,000 transcripts, adds a pseudocount of 1, and log-transforms. Then run ALRA's choose_k, which computes the top 100 singular values via rsvd with q = 2 additional power iterations. Compute consecutive spacings s_i = sigma_i - sigma_{i-1} for i = 2..100, estimate the noise level as the mean mu and standard deviation sigma of the tail spacings s80..s100, and select k as the largest index whose spacing satisfies s_k > mu + 6*sigma, honoring the upper bound k < 100. The rationale is that identifying the 'beginning of noise' from the spectrum itself is hard, but noise-level spacings are statistically homogeneous, so a spacing far above the tail-noise mean plus six standard deviations marks the signal/noise boundary.

## Related tools

- **ALRA** (Provides the choose_k and normalize_data functions implementing the singular-value-spacing rank selection; install via devtools::install_github("KlugerLab/ALRA")) — https://github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD package used by choose_k to compute the top 100 singular values with q = 2 power iterations; the only prerequisite of ALRA (install.packages('rsvd')))
- **rpca-mkl** (Optional MKL-accelerated randomized SVD backend usable via ALRA's use.mkl flag for dramatic speedups; not required) — https://github.com/KlugerLab/rpca-mkl

## Examples

```
library(ALRA); A_norm <- normalize_data(b_nk_example); k_choice <- choose_k(A_norm); k_choice$k
```

## Evaluation signals

- Returned k is a positive integer <= 100 (choose_k's upper bound)
- The spacing criterion s_k > mu + 6*sigma over noise spacings s80..s100 is actually exercised in the computation
- On the b_nk_example subset, the chosen k and its vertical line appear at the signal/noise boundary in the singular-value and spacing plots (geom_vline at k_choice$k and k_choice$k+1)
- Selected k is consistent with paper-reported ranks for corresponding datasets (k = 26 purified PBMCs, k = 48 Hrvatin, k = 53 Tasic, k = 14 Drop-seq melanoma)

## Limitations

- k is capped at 100 because choose_k computes only the top 100 singular values; datasets whose true rank approaches or exceeds 100 cannot be resolved by this criterion
- The README notes the automatically chosen k worked well for the paper's results but 'in some cases you might want to take a closer look', i.e. the criterion may need manual inspection per dataset
- Results improve when using the entire dataset rather than a subset like b_nk_example

## Evidence

- [readme] spacing-based k selection rationale: "The k is chosen based on the spacings between the singular values, as it can be quite hard to identify the ``beginning of noise'' from just looking at the spectrum itself."
- [readme] rsvd prerequisite: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
- [intro] SVD first step of ALRA: "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
- [methods] noise-spacing criterion from source card: [evidence span withheld — anchor status: not_found]
