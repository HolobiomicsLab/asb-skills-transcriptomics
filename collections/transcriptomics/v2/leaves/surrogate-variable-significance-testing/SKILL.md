---
name: surrogate-variable-significance-testing
description: Use when scRNA-seq read-count data appear homogeneous after known covariate
  adjustment but residual heterogeneity is suspected, and you need to test significance
  of hidden factors and estimate how many SVs to keep.
when_to_use_negative:
- PC1 already captures only library-size variation (r ≈ 0.99 with geometric library
  size) and should be discarded rather than treated as a hidden factor
- The input is already normalized/embedded (e.g., a tSNE or PCA embedding); IA-SVA
  operates on the residualized read-count matrix
- Compromised/outlier cells detected by IA-SVA should be excluded before downstream
  analyses — do not model them as a biological condition
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: iasva
  role: R/Bioconductor package implementing IA-SVA with iterative hidden-factor detection
    and permutation-based significance testing
  repo: https://github.com/UcarLab/iasva
- name: fast_iasva
  role: Fast IA-SVA function that selects important SVs based on the proportion of
    variation explained
  repo: https://github.com/UcarLab/iasva
- name: IRLBA
  role: Augmented implicitly restarted Lanczos bidiagonalization algorithm used for
    SVD on the residuals to obtain PC1
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:b236a96f38dcaa82a92cda7fd5d6ea5a4639f2d097310b50485d7e1f61cb7ec3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/iasva
  tools:
  - IRLBA
  - fast_iasva
  - iasva
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# surrogate-variable-significance-testing

## Summary

Apply IA-SVA (Iteratively Adjusted SVA) to scRNA-seq count matrices to iteratively detect hidden sources of variation (e.g., doublets, cell-cycle stage, rare cell states) that are correlated with known covariates, and use its permutation-based significance index (SI) to decide how many surrogate variables to retain.

## When to use

Use when scRNA-seq read-count data appear homogeneous after known covariate adjustment but residual heterogeneity is suspected, and you need to test significance of hidden factors and estimate how many SVs to keep.

## When NOT to use

- PC1 already captures only library-size variation (r ≈ 0.99 with geometric library size) and should be discarded rather than treated as a hidden factor
- The input is already normalized/embedded (e.g., a tSNE or PCA embedding); IA-SVA operates on the residualized read-count matrix
- Compromised/outlier cells detected by IA-SVA should be excluded before downstream analyses — do not model them as a biological condition

## Inputs

- Gene-by-cell raw read count matrix (e.g., iasva_counts_test.Rds, 101 alpha cells from Lawlor et al. islet data, wrapped in a SummarizedExperiment)
- Sample annotation table (e.g., iasva_anns_test.Rds) with known covariates such as patient identity
- Geometric library size per cell computed as colSums(log(counts + 1))

## Outputs

- Surrogate variable values per cell (e.g., SV1 scores for all 101 cells) written to a table such as iasva_sv1_results.csv
- Significance index (SI) per SV from the permutation test, used to determine the number of hidden factors
- Genes significantly associated with each SV (multiple testing adjusted p-value < 0.05, R-squared > 0.3) for downstream analyses

## How to apply

Regress the gene-by-cell count matrix on all known factors (e.g., model.matrix(~patient_id + geo_lib_size) with the intercept dropped) to obtain residuals, then run SVD on the residuals (via IRLBA) and take PC1 as the candidate hidden factor. Test its significance with the permutation-based quasi-inferential procedure (test statistic T_obs = λ1^2 / Σ λk^2, α = 0.05, 50 permutations) to obtain a significance index; if significant, add the estimated factor to the known covariates and iterate to uncover further SVs (num.sv = 5 is a typical run). In the Lawlor alpha-cell example, IA-SVA on 101 healthy alpha cells (iasva_counts_test.Rds) separated six outlier cells via SV1 > -0.1 that PCA, USVA, and tSNE failed to detect clearly. Optionally use the fast_iasva variant, which selects important SVs based on the proportion of variation explained, for faster runs.

## Related tools

- **iasva** (R/Bioconductor package implementing IA-SVA with iterative hidden-factor detection and permutation-based significance testing) — https://github.com/UcarLab/iasva
- **fast_iasva** (Fast IA-SVA function that selects important SVs based on the proportion of variation explained) — https://github.com/UcarLab/iasva
- **IRLBA** (Augmented implicitly restarted Lanczos bidiagonalization algorithm used for SVD on the residuals to obtain PC1)

## Examples

```
sv <- iasva(sumExp, patient_design[,-1], num.sv = 5, permute = TRUE); sv1 <- sv$sv$SV1; outliers <- which(sv1 > -0.1)
```

## Evaluation signals

- The SV1 result table contains exactly one SV1 value per cell (101 rows for the alpha-cell test set) plus a significance index
- Thresholding SV1 at -0.1 reproduces the reported 6 outlier cells separated from the remaining 95 alpha cells
- Geometric library size (colSums(log(counts+1))) correlates highly with PC1 (r ≈ 0.99) of log-transformed counts, confirming the covariate construction
- SI values below the α = 0.05 cutoff (50 permutations) lead to stopping rather than retaining additional SVs
- SV1 splits the alpha cells into two clear groups in contrast to PCA, USVA, and tSNE, which failed to clearly detect them

## Limitations

- The significance index (SI) is not an empirical p-value and caveats remain in determining the number of hidden factors
- Users should select a stringent SI cut-off when many unknown factors are expected, but no specific cut-off threshold is provided
- USVA and SSVA give biased estimates under correlated factors due to orthogonality assumptions; the comparison baseline behavior differs
- Empirical Type I error rates remain nonzero (0.04 for IA-SVA at α = 0.05 with 50 permutations)

## Evidence

- [readme] Iteratively Adjusted Surrogate Variable Analysis: "IA-SVA provides a flexible methodology to i) identify a hidden factor for unwanted heterogeneity while adjusting for all known factors"
- [methods] significance testing procedure: [evidence span withheld — anchor status: not_found]
- [results] permutation-based SI: "applied IA-SVA, USVA and SSVA (α = 0.05, 50 permutations) on them"
- [results] covariate construction: [evidence span withheld — anchor status: not_found]
- [discussion] SI caveat: "We recommend users of IA-SVA to select a stringent cut-off for SI values especially if they expect to detect many unknown factors from their data"
