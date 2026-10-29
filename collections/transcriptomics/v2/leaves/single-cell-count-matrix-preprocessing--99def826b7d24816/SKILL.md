---
name: single-cell-count-matrix-preprocessing
description: Use when a raw scRNAseq count matrix must be pre-processed into standardized
  residuals prior to SVD/CA — e.g., to address overdispersion of counts instead of
  applying log(x+1) followed by PCA.
when_to_use_negative:
- Input is already log-transformed 'logcounts' destined for PCA — this skill instead
  replaces that log(x+1) + PCA route with a residual-based CA pre-processing stage
- The goal is multi-table batch integration rather than single-table pre-processing
  — use corralm instead
- Input data are microarray or bulk RNA-seq rather than single-cell count data
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: corral
  role: Reference implementation whose corral_preproc function (with rtype/vst_mth
    settings) provides the ground-truth residual matrices each re-implemented transformation
    is verified against
  repo: github.com/laurenhsu1/corral
- name: R
  role: Implementation language for re-implementing the residual and variance-stabilizing
    transformations on SingleCellExperiment objects
- name: Bioconductor
  role: Ecosystem providing the SingleCellExperiment class used as the common test
    input format
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:ff52a1d67d434c32902550f8b716c19dcbad3f41776bb9713f7a23c0c4665785
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-count-matrix-preprocessing

## Summary

Transforms a scRNAseq count matrix into a chi-squared residual matrix (Pearson or Freeman–Tukey, optionally with square-root/Anscombe/Freeman–Tukey variance-stabilizing transforms and sign-preserving power deflation α) as the pre-processing stage before CA-based dimension reduction, replacing the distortive log(x+1) transform.

## When to use

Use when a raw scRNAseq count matrix must be pre-processed into standardized residuals prior to SVD/CA — e.g., to address overdispersion of counts instead of applying log(x+1) followed by PCA.

## When NOT to use

- Input is already log-transformed 'logcounts' destined for PCA — this skill instead replaces that log(x+1) + PCA route with a residual-based CA pre-processing stage
- The goal is multi-table batch integration rather than single-table pre-processing — use corralm instead
- Input data are microarray or bulk RNA-seq rather than single-cell count data

## Inputs

- scRNAseq count matrix loaded as a SingleCellExperiment in R/Bioconductor
- Choice of residual type (rtype: Pearson or Freeman–Tukey) and vst_mth setting (sqrt, Anscombe, Freeman–Tukey) and power deflation exponent α

## Outputs

- Chi-squared residual matrix (Pearson or Freeman–Tukey) ready for SVD-based CA
- preproc_verification_results.csv with element-wise maximum absolute differences and pass/fail status versus corral::corral_preproc output

## How to apply

Load the scRNAseq count matrix into R/Bioconductor as a SingleCellExperiment, then compute expected values from the row and column marginals and form the standardized Pearson residuals (observed minus expected over sqrt of expected); compute the Freeman–Tukey residual matrix as the alternative chi-squared statistic. Optionally apply the square-root, Anscombe, or Freeman–Tukey variance-stabilizing transforms, and address overdispersion with power deflation: raise each transformed residual to a power α while preserving the sign of each value, smoothing the residual matrix before SVD. Validate each re-implementation by running corral::corral_preproc with the matching rtype/vst_mth setting on the identical count matrix and comparing element-wise, recording maximum absolute differences and pass/fail status (e.g., in preproc_verification_results.csv); accept only within numerical tolerance.

## Related tools

- **corral** (Reference implementation whose corral_preproc function (with rtype/vst_mth settings) provides the ground-truth residual matrices each re-implemented transformation is verified against) — github.com/laurenhsu1/corral
- **R** (Implementation language for re-implementing the residual and variance-stabilizing transformations on SingleCellExperiment objects)
- **Bioconductor** (Ecosystem providing the SingleCellExperiment class used as the common test input format)

## Examples

```
devtools::install_github('laurenhsu1/corral'); R <- corral::corral_preproc(sce, rtype = 'FT')
```

## Evaluation signals

- Each re-implemented transformation reproduces the corresponding corral::corral_preproc (rtype/vst_mth) output on the identical count matrix within element-wise numerical tolerance
- preproc_verification_results.csv records maximum absolute differences and pass/fail status for every transformation (Pearson, Freeman–Tukey, and each VST with power deflation)
- Sign preservation: power deflation raises residual values to power α without flipping signs (negative residuals stay negative)
- Expected values derive from the row and column marginals of the count matrix; Pearson residual = (observed − expected)/sqrt(expected)

## Limitations

- No tool or package versions are reported for corral or its dependencies, so reproducibility of the pre-processing comparison may depend on the installed corral version
- CA is based on decomposition of the chi-squared residual matrix and avoids log-transformation; applying this pre-processing assumes the count matrix (not normalized logcounts) is the appropriate input
- The benchmarking datasets used to motivate the adaptations (SCMixology, Zhengmix) comprise well-defined cell clusters and are by design simpler than data from complex tissues

## Evidence

- [abstract] CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation: "CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation"
- [intro] most single cell workflows begin with a log(x + 1) transformation of the counts matrix, and then use PCA to decompose the resulting “logcounts” data: "most single cell workflows begin with a log(x + 1) transformation of the counts matrix, and then use PCA to decompose the resulting “logcounts” data"
- [intro] CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [intro] We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment): "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment)"
- [readme] To install the dev version: `devtools::install_github('laurenhsu1/corral')`: "To install the dev version: `devtools::install_github('laurenhsu1/corral')`"
