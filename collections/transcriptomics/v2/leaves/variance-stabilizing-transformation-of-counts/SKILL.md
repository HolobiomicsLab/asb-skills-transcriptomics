---
name: variance-stabilizing-transformation-of-counts
description: Use when an scRNAseq count matrix is overdispersed relative to the multinomial/Poisson
  assumption underlying standard CA and you need a variance-stabilized residual matrix
  (or power-deflated residuals) prior to SVD-based dimension reduction.
when_to_use_negative:
- Data are not raw counts (e.g., already log-transformed 'logcounts' or normalized
  expression), since CA's residual computation expects count marginals
- The goal is integrative multi-table batch alignment across batches/samples, where
  corralm (not single-table CA pre-processing) is the appropriate entry point
- Only one transformation output is needed without verification — still cross-check
  against corral_preproc, since re-implementations are accepted only within element-wise
  numerical tolerance
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: corral
  role: R/Bioconductor package providing corral_preproc with rtype/vst_mth settings
    that implement standard CA, Freeman-Tukey residuals, VST variants, and power deflation
    on count data
  repo: https://github.com/laurenhsu1/corral
- name: Bioconductor
  role: Framework whose SingleCellExperiment class is the expected input container
    for the count matrix
- name: R
  role: Language in which the residual transforms are re-implemented and compared
    to the corral reference
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:7e31775283639928977282c005884ccd5a3ff76220680e9c6e9a57ff5dc84ed6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/laurenhsu1/corral
  tools:
  - Bioconductor
  - R
  - corral
  - glmPCA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# variance-stabilizing-transformation-of-counts

## Summary

Pre-processing skill from the corral correspondence analysis (CA) framework: transform scRNAseq count matrices via variance-stabilizing transformations (square-root, Anscombe, Freeman-Tukey) or power deflation before SVD, to address overdispersion that distorts standard CA's Pearson-residual decomposition.

## When to use

Use when an scRNAseq count matrix is overdispersed relative to the multinomial/Poisson assumption underlying standard CA and you need a variance-stabilized residual matrix (or power-deflated residuals) prior to SVD-based dimension reduction.

## When NOT to use

- Data are not raw counts (e.g., already log-transformed 'logcounts' or normalized expression), since CA's residual computation expects count marginals
- The goal is integrative multi-table batch alignment across batches/samples, where corralm (not single-table CA pre-processing) is the appropriate entry point
- Only one transformation output is needed without verification — still cross-check against corral_preproc, since re-implementations are accepted only within element-wise numerical tolerance

## Inputs

- scRNAseq count matrix (genes x cells) loaded as a SingleCellExperiment object in R/Bioconductor
- Choice of transformation: rtype/vst_mth setting for square-root, Anscombe, Freeman-Tukey VST, Freeman-Tukey residuals, or power deflation with alpha (e.g., 0.9)

## Outputs

- Transformed chi-squared residual matrix (sign-preserving, optionally power-deflated) ready for SVD in CA
- Element-wise verification results comparing re-implemented transforms to corral_preproc output (max absolute differences, pass/fail status)

## How to apply

Load the scRNAseq counts as a SingleCellExperiment object in R/Bioconductor and compute the standardized Pearson residual matrix (observed minus expected over sqrt of expected, with expectations from row and column marginals) or the alternative Freeman-Tukey chi-squared residual. Apply one of the variance-stabilizing transforms — square-root, Anscombe, or Freeman-Tukey — or apply power deflation by raising each transformed residual to a power alpha (e.g., alpha = 0.9) while preserving the sign of each value, smoothing the residual matrix before decomposition. Run corral::corral_preproc with the matching rtype/vst_mth setting on the identical count matrix to obtain the reference implementation, then verify each re-implemented transformation element-wise against it, recording maximum absolute differences and pass/fail status (e.g., in preproc_verification_results.csv). The rationale is that these adaptations address overdispersion in scRNAseq counts, and CA-FT and the other adaptations outperform standard CA and glmPCA in 8 of 9 benchmark datasets.

## Related tools

- **corral** (R/Bioconductor package providing corral_preproc with rtype/vst_mth settings that implement standard CA, Freeman-Tukey residuals, VST variants, and power deflation on count data) — https://github.com/laurenhsu1/corral
- **Bioconductor** (Framework whose SingleCellExperiment class is the expected input container for the count matrix)
- **R** (Language in which the residual transforms are re-implemented and compared to the corral reference)

## Examples

```
library(corral); res <- corral_preproc(sce, rtype = "FT", vst_mth = "sqrt", deflate = TRUE, alpha = 0.9)
```

## Evaluation signals

- Re-implemented transformation reproduces the corral_preproc output for the same rtype/vst_mth within element-wise numerical tolerance (max absolute difference ~ 0)
- Pearson residuals satisfy (observed - expected) / sqrt(expected) with expectations from row and column marginals
- Power deflation preserves the sign of every residual value while raising magnitudes to power alpha
- Downstream CA embeddings from CA-FT / VST variants yield higher clustering ARI than standard CA, as observed in the 9-dataset benchmark
- Verification table (preproc_verification_results.csv) records pass status for each transformation

## Limitations

- No tool or package versions reported for corral or its dependencies, so exact numerical behavior may vary across installations
- CA avoids log-transformation distortions but the optimal adaptation (FT residuals vs. VST vs. power deflation) cannot yet be selected from data characteristics due to lack of systematic benchmarking frameworks
- ARI-based evaluation may penalize methods that better distinguish sub-populations or rare cell types relative to reference labels

## Evidence

- [abstract] CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation: "CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation"
- [abstract] Overdispersion adaptations outperform standard CA and glmPCA in 8 of 9 datasets: "we propose five adaptations of CA, which are fast, scalable, and outperform standard CA and glmPCA, to compute cell embeddings with more performant or comparable clustering accuracy in 8 out of 9"
- [intro] CA computed by SVD of the Pearson residuals: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [intro] Implemented standard and adapted CA in corral interfacing with Bioconductor classes: "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes"
- [readme] corral install instructions from README: "To install the dev version: `devtools::install_github('laurenhsu1/corral')`"
