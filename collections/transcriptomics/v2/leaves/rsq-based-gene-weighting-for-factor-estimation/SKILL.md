---
name: rsq-based-gene-weighting-for-factor-estimation
description: Use when reconstructing or adapting IA-SVA to estimate a surrogate variable
  for a hidden factor (e.g., patient or cell-type heterogeneity) from single-cell
  read counts after adjusting for known covariates such as patient_id and geo_lib_size.
when_to_use_negative:
- Input is already normalized/transformed expression or a feature table rather than
  raw read counts, since the procedure regresses log-transformed read counts on the
  design matrix
- The goal is orthogonal factor decomposition only (e.g., plain PCA/SVA); IA-SVA specifically
  targets hidden factors that may be correlated with known covariates
- No permutation-based significance test is desired or feasible, since the R-squared
  weighting step is gated on PC1 significance testing with permuted columns
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: iasva
  role: Reference implementation providing iasva() for comparison and study_R2() for
    tuning R-squared thresholds on marker-gene selection
  repo: github.com/UcarLab/iasva
- name: IRLBA
  role: Augmented implicitly restarted Lanczos bidiagonalization algorithm used to
    run the SVD steps that extract PC1 and the surrogate variable
- name: iasvaExamples
  role: R data package with example datasets and vignettes (e.g., alpha-cell hidden-heterogeneity
    tutorial) grounding the workflow
  repo: github.com/dleelab/iasvaExamples
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:83ad41cba53349ea6973f9ff00b66c94a6d9481d1aa0965559848affd40c73f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IRLBA
  - SummarizedExperiment
  - iasva
  - iasvaExamples
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# rsq-based-gene-weighting-for-factor-estimation

## Summary

A core mechanism of IA-SVA in which per-gene R-squared values from regressing residualized read counts on PC1 are used to build a weighted matrix Y'' = Y * W, whose second SVD yields the surrogate variable for a hidden factor.

## When to use

Use when reconstructing or adapting IA-SVA to estimate a surrogate variable for a hidden factor (e.g., patient or cell-type heterogeneity) from single-cell read counts after adjusting for known covariates such as patient_id and geo_lib_size.

## When NOT to use

- Input is already normalized/transformed expression or a feature table rather than raw read counts, since the procedure regresses log-transformed read counts on the design matrix
- The goal is orthogonal factor decomposition only (e.g., plain PCA/SVA); IA-SVA specifically targets hidden factors that may be correlated with known covariates
- No permutation-based significance test is desired or feasible, since the R-squared weighting step is gated on PC1 significance testing with permuted columns

## Inputs

- Raw single-cell read count matrix Y (genes x cells), e.g., iasva_counts_test.Rds (alpha cells, n=101)
- Sample annotation table with known covariates (patient_id, geometric library size = colSums of log(counts+1))
- Known-factors design matrix from model.matrix(~patient_id + geo_lib_size)
- Prior surrogate variables from previous IA-SVA iterations, if any

## Outputs

- Per-gene R-squared values from regressing residuals Y' on PC1
- Weighted matrix Y'' = Y * W (W diagonal of R-squared values)
- Surrogate variable (SV) estimate from the second SVD of Y''
- Set of genes significantly associated with the hidden factor (adj p < 0.05, R-squared > 0.3)

## How to apply

Regress the log-transformed gene-by-cell count matrix Y on all known factors (e.g., model.matrix(~patient_id + geo_lib_size)) including SVs from prior iterations to obtain residuals Y', then run SVD (via IRLBA) on Y' to extract PC1 and test its significance with the permutation statistic T = lambda1^2 / sum_k(lambda_k^2). Regress Y' on PC1 to compute per-gene R-squared values, form the diagonal-weighted matrix Y'' = Y * W where W carries those R-squared values, and run a second SVD on Y'' to obtain the SV. Marker-gene selection typically uses adjusted p-value < 0.05 and R-squared > 0.3, and the study_R2() function lets users visualize how different R-squared thresholds affect the number of genes and variance explained. Validate by comparing the reimplemented SV1 to the reference iasva() output via Pearson correlation (the paper reports ~0.926 average correlation for a hidden brain cell-type SV over 100 runs).

## Related tools

- **iasva** (Reference implementation providing iasva() for comparison and study_R2() for tuning R-squared thresholds on marker-gene selection) — github.com/UcarLab/iasva
- **IRLBA** (Augmented implicitly restarted Lanczos bidiagonalization algorithm used to run the SVD steps that extract PC1 and the surrogate variable)
- **iasvaExamples** (R data package with example datasets and vignettes (e.g., alpha-cell hidden-heterogeneity tutorial) grounding the workflow) — github.com/dleelab/iasvaExamples

## Examples

```
iasva(summ_exp, mod[,-1], permute=FALSE, num.sv=5)
```

## Evaluation signals

- Reimplemented SV1 correlates with reference iasva(summ_exp, mod[,-1], permute=FALSE, num.sv=5) SV1 at Pearson r close to 1 on the alpha-cell data
- Estimated SV reaches the level the paper reports for hidden-factor recovery (average correlation 0.926 for the brain cell-type SV over 100 runs)
- Selected marker genes satisfy adj p-value < 0.05 and R-squared > 0.3
- Second SVD is performed on Y'' = Y * W (R-squared-weighted counts), not on the raw residual matrix Y'
- PC1 significance test statistic T = lambda1^2 / sum_k(lambda_k^2) computed against permuted columns of Y matches a sensible empirical null

## Limitations

- The significance index (SI) is not a true empirical p-value and has caveats; users must choose a stringent SI cut-off when many unknown factors are expected, but no specific threshold is provided
- No software versions are stated for the companion tools, complicating exact reproduction
- R-squared threshold choice affects both the number of selected genes and variance explained and must be tuned (e.g., via study_R2())

## Evidence

- [methods] Step 1: regress Y on all known factors including SVs from previous iterations to obtain residuals: [evidence span withheld — anchor status: not_found]
- [methods] Step 2: SVD on residuals yields PC1 feeding the R-squared weighting: "Conduct SVD on the obtained residuals ( ′ × Y m n ) and obtain the first PC (PC1)"
- [methods] Default marker-gene R-squared and significance cutoffs: "marker genes that are significantly associated with SV1 (multiple testing adjusted p-value < 0.05, default significance cutoff, and R-squared value > 0.3"
- [methods] study_R2 for threshold tuning: "With the study_R2() function, users can visualize how different R-squared thresholds influence both factors"
- [readme] IA-SVA purpose framing from the package README: "identify a hidden factor for unwanted heterogeneity while adjusting for all known factors; ii) test the significance of the putative hidden factor"
