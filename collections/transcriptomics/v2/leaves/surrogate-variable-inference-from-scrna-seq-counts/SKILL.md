---
name: surrogate-variable-inference-from-scrna-seq-counts
description: Use when scRNA-seq read-count matrices are suspected to contain hidden
  factors (e.g., cell-cycle stage, doublets, patient-specific effects) correlated
  with known covariates, and a surrogate variable plus associated gene set is needed
  while adjusting for patient_id and library size.
when_to_use_negative:
- Input is already normalized/transformed or a reduced feature table — IA-SVA is defined
  on raw read counts adjusted for known factors
- All sources of variation are already known and modeled in the design matrix, so
  no hidden factor remains to be uncovered
- Goal is discarding variation rather than estimating a hidden factor and its associated
  gene set (e.g., simply regressing out geometric library size, which can reach r
  = 0.99 with PCA PC1)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: iasva
  role: Reference implementation of the IA-SVA algorithm used to obtain the package's
    SV1 estimates for comparison against the standalone reimplementation
  repo: https://github.com/UcarLab/iasva
- name: IRLBA
  role: Augmented implicitly restarted Lanczos bidiagonalization algorithm used to
    compute truncated SVD/PC1 on residual and weighted matrices
  repo: https://bwlewis.github.io/irlba/
- name: SummarizedExperiment
  role: Container class for the count matrix and annotations passed to iasva()
- name: fast_iasva
  role: Fast IA-SVA variant that selects important SVs based on the proportion of
    variation explained, used in runtime benchmarking
  repo: https://github.com/UcarLab/iasva
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:88ec1f449adb6b6a45eab9c5c42e7aab43a7db6a5562cdcec8f4a78e050737d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/iasva
  tools:
  - IRLBA
  - SummarizedExperiment
  - fast_iasva
  - iasva
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# surrogate-variable-inference-from-scrna-seq-counts

## Summary

Reconstructs the six-step IA-SVA procedure (regress out known factors, SVD to PC1, permutation-based significance test, gene-wise R-squared weighting, second SVD) as a standalone function that computes a surrogate variable for a hidden factor from single-cell read counts, validated against the iasva Bioconductor/GitHub package.

## When to use

Use when scRNA-seq read-count matrices are suspected to contain hidden factors (e.g., cell-cycle stage, doublets, patient-specific effects) correlated with known covariates, and a surrogate variable plus associated gene set is needed while adjusting for patient_id and library size.

## When NOT to use

- Input is already normalized/transformed or a reduced feature table — IA-SVA is defined on raw read counts adjusted for known factors
- All sources of variation are already known and modeled in the design matrix, so no hidden factor remains to be uncovered
- Goal is discarding variation rather than estimating a hidden factor and its associated gene set (e.g., simply regressing out geometric library size, which can reach r = 0.99 with PCA PC1)

## Inputs

- Read-count matrix from iasva_counts_test.Rds (alpha cells, n = 101, loaded via system.file())
- Sample annotations from iasva_anns_test.Rds containing patient_id (known factor)
- SummarizedExperiment object (summ_exp) wrapping counts and annotations
- Known-factors design matrix mod from model.matrix(~patient_id + geo_lib_size)

## Outputs

- SV1 surrogate-variable estimates (vector over cells) from the reimplemented function
- Per-gene R-squared values associating genes with the hidden factor
- sv1_comparison.csv containing both SV1 estimate sets and their Pearson correlation

## How to apply

Load the alpha-cell read-count matrix (iasva_counts_test.Rds, n=101 cells) and annotations (iasva_anns_test.Rds) via system.file(); compute the geometric library size as colSums of log(counts+1) and build the design matrix model.matrix(~patient_id + geo_lib_size). Then implement the six iterative steps in one R function: regress log-transformed counts Y on all known factors including prior SVs to obtain residuals Y', run SVD (irlba) on Y' to extract PC1, test PC1 significance with the permutation statistic T = lambda1^2 / sum_k(lambda_k^2) using permuted columns of Y, regress Y' on PC1 for per-gene R-squared values, form Y'' = Y * W with W a diagonal matrix of R-squared values, and run a second SVD on Y'' to obtain the SV. As a reference, run iasva(summ_exp, mod[,-1], permute=FALSE, num.sv=5) on the same SummarizedExperiment input and compute the Pearson correlation between the reimplemented and package SV1 vectors, writing both plus the correlation to sv1_comparison.csv. Marker-gene selection uses adjusted p-value < 0.05 and R-squared > 0.3; alpha = 0.05 with 50 permutations is used for the significance index.

## Related tools

- **iasva** (Reference implementation of the IA-SVA algorithm used to obtain the package's SV1 estimates for comparison against the standalone reimplementation) — https://github.com/UcarLab/iasva
- **IRLBA** (Augmented implicitly restarted Lanczos bidiagonalization algorithm used to compute truncated SVD/PC1 on residual and weighted matrices) — https://bwlewis.github.io/irlba/
- **SummarizedExperiment** (Container class for the count matrix and annotations passed to iasva())
- **fast_iasva** (Fast IA-SVA variant that selects important SVs based on the proportion of variation explained, used in runtime benchmarking) — https://github.com/UcarLab/iasva

## Examples

```
iasva(summ_exp, mod[,-1], permute=FALSE, num.sv=5)
```

## Evaluation signals

- Pearson correlation between the reimplemented SV1 and the iasva-package SV1 must be close to 1 on the alpha-cell data
- Correlation should approach the paper's reported level for IA-SVA hidden-factor estimates (average 0.926 over 100 runs on brain cell-type SV; SVA/PCA only ~0.85, ZINB-WaVe 0.81)
- Permutation significance index (T = lambda1^2 / sum_k(lambda_k^2)) must flag the hidden factor as significant at alpha = 0.05 with 50 permutations before an SV is accepted
- Selected marker genes must pass adjusted p-value < 0.05 and R-squared > 0.3 cutoffs, and should recapitulate known islet marker genes (e.g., alpha-cell markers)
- sv1_comparison.csv must contain both SV1 vectors and a valid scalar correlation

## Limitations

- The significance index (SI) is not an empirical p-value and has caveats for determining the number of hidden factors; a stringent SI cut-off is recommended when many unknown factors are expected, but no specific threshold is provided
- Heterogeneity explained by compromised alpha cells (possible cell contamination) is detected but those cells should be excluded from downstream analyses
- No software versions are stated for companion tools, and the discussion notes no code-availability statement for IA-SVA there (repos are given in methods)

## Evidence

- [methods] Regress Y m×n on all known factors (X m×p) , including SVs obtained from previous iterations, to obtain residuals: [evidence span withheld — anchor status: not_found]
- [methods] Conduct SVD on the obtained residuals and obtain PC1: "Conduct SVD on the obtained residuals ( ′ × Y m n ) and obtain the first PC (PC1)"
- [methods] marker genes that are significantly associated with SV1 (multiple testing adjusted p-value < 0.05, default significance cutoff, and R-squared value > 0.3: "marker genes that are significantly associated with SV1 (multiple testing adjusted p-value < 0.05, default significance cutoff, and R-squared value > 0.3"
- [results] augmented implicitly restarted Lanczos bidiagonalization algorithm (IRLBA): [evidence span withheld — anchor status: not_found]
- [methods] freely available from https://github.com/Ucar Lab/iasva: [evidence span withheld — anchor status: not_found]
- [readme] IA-SVA provides a flexible methodology to i) identify a hidden factor for unwanted heterogeneity while adjusting for all known factors: "IA-SVA provides a flexible methodology to i) identify a hidden factor for unwanted heterogeneity while adjusting for all known factors"
- [results] average correlation is 0.926 for 100 runs: "average correlation is 0.926 for 100 runs). SVA and PCA were the second-best performers for this task (average correlations around 0.85)."
