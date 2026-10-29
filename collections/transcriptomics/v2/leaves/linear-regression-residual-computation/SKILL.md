---
name: linear-regression-residual-computation
description: Use when an IA-SVA (or SVA-style) analysis must remove known-factor variation
  from single-cell log-transformed counts before extracting PCs, i.e., residualization
  of Y on the design matrix built via model.matrix(~patient_id + geo_lib_size).
when_to_use_negative:
- Input is already a residual/adjusted matrix or a normalized feature table — re-regressing
  would double-adjust known covariates
- No known covariates exist and the goal is a plain PCA on raw counts rather than
  IA-SVA hidden-factor detection
- The analysis goal is differential expression rather than surrogate-variable construction
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: IRLBA
  role: Run truncated SVD (implicitly restarted Lanczos bidiagonalization) on the
    residual matrix Y' to extract PC1
  repo: https://bwlewis.github.io/irlba/
- name: iasva
  role: Reference implementation whose SV outputs the residual-based PC1/SV reconstruction
    is compared against
  repo: github.com/UcarLab/iasva
- name: SummarizedExperiment
  role: Container format for the read-count matrix and sample annotations passed to
    iasva()
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:a232aeef90c150ae70577b6b1b4a598e00f08618a59a266d7e2a76554805b16e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IRLBA
  - SummarizedExperiment
  - iasva
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# linear-regression-residual-computation

## Summary

Regresses a gene-by-cell read-count matrix Y on all known covariates (patient_id, geometric library size, and surrogate variables from prior IA-SVA iterations) and uses the residuals Y' as the input to downstream SVD for hidden-factor detection.

## When to use

Use when an IA-SVA (or SVA-style) analysis must remove known-factor variation from single-cell log-transformed counts before extracting PCs, i.e., residualization of Y on the design matrix built via model.matrix(~patient_id + geo_lib_size).

## When NOT to use

- Input is already a residual/adjusted matrix or a normalized feature table — re-regressing would double-adjust known covariates
- No known covariates exist and the goal is a plain PCA on raw counts rather than IA-SVA hidden-factor detection
- The analysis goal is differential expression rather than surrogate-variable construction

## Inputs

- gene-by-cell read-count matrix Y (e.g., iasva_counts_test.Rds, alpha cells n=101)
- sample-annotation table (e.g., iasva_anns_test.Rds) with patient_id
- geometric library size vector (colSums of log(counts+1))
- design matrix X of known covariates including prior-iteration SVs

## Outputs

- residual matrix Y' (genes m x cells n) adjusted for known factors
- PC1 of Y' obtained by SVD, used as the putative hidden-factor estimate

## How to apply

Build the known-factors design matrix from the sample annotations (e.g., model.matrix(~patient_id + geo_lib_size), where geo_lib_size is the column-sums of log(counts+1), i.e., the geometric library size). Regress the log-transformed count matrix Y (genes m x cells n) on all known factors X, including SVs obtained from previous IA-SVA iterations, to obtain the residual matrix Y'. Feed Y' into a truncated SVD (IRLBA) to extract PC1, which the IA-SVA procedure then tests for significance via the permutation statistic T = lambda1^2 / sum_k(lambda_k^2). The residualization step is what allows IA-SVA to uncover hidden factors even when they are correlated with known covariates.

## Related tools

- **IRLBA** (Run truncated SVD (implicitly restarted Lanczos bidiagonalization) on the residual matrix Y' to extract PC1) — https://bwlewis.github.io/irlba/
- **iasva** (Reference implementation whose SV outputs the residual-based PC1/SV reconstruction is compared against) — github.com/UcarLab/iasva
- **SummarizedExperiment** (Container format for the read-count matrix and sample annotations passed to iasva())

## Examples

```
mod <- model.matrix(~patient_id + geo_lib_size, anns); sv <- iasva(summ_exp, mod[,-1], permute=FALSE, num.sv=5)
```

## Evaluation signals

- SV1 from the reimplementation correlates with the package iasva() SV1 with Pearson correlation near 1 (paper benchmark: average correlation 0.926 for the brain cell-type SV over 100 runs)
- Residuals Y' show no remaining association with patient_id or geo_lib_size covariates
- PC1 significance test statistic T = lambda1^2 / sum_k(lambda_k^2) computed against permuted columns of Y is significant before proceeding to gene identification
- Comparison table sv1_comparison.csv contains both SV1 estimate sets plus the correlation value

## Limitations

- Users must select a stringent SI cut-off, especially when many unknown factors are expected, but no specific cut-off threshold is provided
- SI is not an empirical p-value and caveats remain in determining the number of hidden factors
- IA-SVA-detected heterogeneity from compromised alpha cells (possible cell contamination) should be excluded from downstream analyses

## Evidence

- [methods] Step 1: regress Y on all known factors including SVs from previous iterations to obtain residuals: [evidence span withheld — anchor status: not_found]
- [methods] Step 2: SVD on residuals to obtain PC1: "Conduct SVD on the obtained residuals ( ′ × Y m n ) and obtain the first PC (PC1)"
- [results] Covariates used for regression in the analyses: [evidence span withheld — anchor status: not_found]
- [readme] IA-SVA framework description from repository README: "IA-SVA is a statistical framework to uncover hidden sources of variation even when these sources are correlated with the biological variable of interest"
