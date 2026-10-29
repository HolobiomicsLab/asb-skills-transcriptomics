---
name: signed-power-deflation-transform
description: Use when scRNAseq count data are overdispersed relative to Poisson expectations
  and you need to deflate a CA chi-squared residual matrix (Pearson or Freeman-Tukey)
  before SVD in a corral preprocessing stage.
when_to_use_negative:
- When the goal is batch integration of multiple tables rather than single-table preprocessing
  — corralm, not the corral preprocessing stage, is the appropriate path
- When the pipeline already uses log(x + 1) transformation followed by PCA — this
  skill specifically replaces that with CA on a chi-squared residual matrix, so mixing
  them is inappropriate
- When counts are not overdispersed and standard CA on plain Pearson residuals suffices
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: corral
  role: R/Bioconductor package whose corral_preproc function (with rtype/vst_mth settings)
    is the reference implementation the re-implemented signed power-deflation residual
    matrix must reproduce; applies CA to count data
  repo: github.com/laurenhsu1/corral
- name: R
  role: implementation language for the residual-transformation pre-processing stage
    (Pearson, Freeman-Tukey, VST, power deflation)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:b85f434426cbd5890ca3a72d0828071363eb06f246a1a8d762fdf62834989f2d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - corral
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# signed-power-deflation-transform

## Summary

A power-deflation adaptation of correspondence analysis for scRNAseq counts: the chi-squared residual matrix (Pearson or Freeman-Tukey, or a variance-stabilizing transform) is raised element-wise to a power α with signs preserved, smoothing the residual matrix prior to SVD to address overdispersion.

## When to use

Use when scRNAseq count data are overdispersed relative to Poisson expectations and you need to deflate a CA chi-squared residual matrix (Pearson or Freeman-Tukey) before SVD in a corral preprocessing stage.

## When NOT to use

- When the goal is batch integration of multiple tables rather than single-table preprocessing — corralm, not the corral preprocessing stage, is the appropriate path
- When the pipeline already uses log(x + 1) transformation followed by PCA — this skill specifically replaces that with CA on a chi-squared residual matrix, so mixing them is inappropriate
- When counts are not overdispersed and standard CA on plain Pearson residuals suffices

## Inputs

- scRNAseq count matrix loaded in R/Bioconductor as a SingleCellExperiment
- row and column marginals of the contingency table for computing expected values

## Outputs

- signed power-deflated chi-squared residual matrix (element-wise α power with sign preserved) ready for SVD
- preproc_verification_results.csv with element-wise maximum absolute differences and pass/fail status versus corral::corral_preproc

## How to apply

Load the scRNAseq count matrix into R/Bioconductor as a SingleCellExperiment, then compute the standardized Pearson residuals (observed minus expected over sqrt of expected, with expectations from row and column marginals) or the Freeman-Tukey residuals as the alternative chi-squared statistic. Compute the square-root, Anscombe, or Freeman-Tukey variance-stabilizing transform, applying power deflation by raising each transformed residual to a power α while preserving the sign of each value. Run corral::corral_preproc with the corresponding rtype/vst_mth setting on the identical count matrix as the reference implementation. Verify element-wise that each re-implemented residual matrix reproduces the corral_preproc output within numerical tolerance (maximum absolute differences recorded in preproc_verification_results.csv); accept only if element-wise agreement holds. The rationale is that signed power deflation smooths the residual matrix before SVD, which the article identifies as one of five CA adaptations addressing overdispersion in scRNAseq counts.

## Related tools

- **corral** (R/Bioconductor package whose corral_preproc function (with rtype/vst_mth settings) is the reference implementation the re-implemented signed power-deflation residual matrix must reproduce; applies CA to count data) — github.com/laurenhsu1/corral
- **R** (implementation language for the residual-transformation pre-processing stage (Pearson, Freeman-Tukey, VST, power deflation))

## Examples

```
resid_deflated <- sign(R) * abs(R)^alpha; ref <- corral::corral_preproc(sce, rtype = "FT")
```

## Evaluation signals

- Re-implemented Pearson residual matrix equals corral_preproc counterpart element-wise within numerical tolerance (pass/fail recorded in preproc_verification_results.csv)
- Freeman-Tukey residual re-implementation agrees with corral_preproc on the same count matrix within tolerance
- Square-root, Anscombe, and Freeman-Tukey VST variants with power deflation each match their corresponding corral_preproc rtype/vst_mth output element-wise
- Sign of every residual value is preserved after raising to power α (e.g., negative residuals remain negative)

## Limitations

- No tool or package versions for corral or its dependencies are reported, so numerical tolerance comparisons may vary across versions
- The corral README provides no parameter documentation in the excerpt; details on rtype/vst_mth settings and α choice must come from package vignettes, not the article text
- Power deflation smooths the residual matrix before SVD, but the article's performance claims are tied to ARI on benchmark datasets whose cell-type labels were not independently validated

## Evidence

- [other] Power deflation operates on the chi-squared residual matrix by raising the transformed residual values to a power α prior to performing SVD, while preserving the sign of each value: [evidence span withheld — anchor status: not_found]
- [intro] We propose and evaluate five adaptations of CA to address overdispersion in scRNAseq counts.: "We propose and evaluate five adaptations of CA to address overdispersion in scRNAseq counts."
- [intro] CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [intro] We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment): "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment)"
- [readme] corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq).: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
