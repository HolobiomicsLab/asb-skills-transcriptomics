---
name: pearson-residual-matrix-computation
description: Use when you have a raw scRNAseq count matrix and need the chi-squared
  residual matrix for CA dimension reduction, or when verifying a re-implementation
  of the corral_preproc count-to-residual pre-processing stage element-wise.
when_to_use_negative:
- The downstream analysis already requires a variance-stabilized or power-deflated
  residual variant (e.g. Freeman–Tukey residuals, square-root/Anscombe VST, or signed
  power deflation) rather than the plain standardized Pearson residuals.
- The goal is batch/multi-table integration across samples — corralm, not the single-table
  residual pre-processing, is the appropriate entry point.
- The pipeline expects the conventional log(x + 1) 'logcounts' transformation followed
  by PCA instead of CA on chi-squared residuals.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: corral
  role: Reference R/Bioconductor package whose corral_preproc count-to-residual pre-processing
    (rtype setting) provides the ground-truth Pearson residual matrix for element-wise
    verification
  repo: https://github.com/laurenhsu1/corral
- name: R
  role: Implementation language for re-computing the standardized Pearson residuals
    from row and column marginals
- name: Bioconductor
  role: Provides SingleCellExperiment as the common test-input container for the scRNAseq
    count matrix
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:3a46f318481ee946637b4c0f3612d9a1d1ba61f37c40cc011959ab2c1c7b8191
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/laurenhsu1/corral
  tools:
  - Bioconductor
  - R
  - corral
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pearson-residual-matrix-computation

## Summary

Computes the standardized Pearson residual matrix (observed minus expected over sqrt of expected, with expectations from row and column marginals) from an scRNAseq count matrix as the chi-squared residual stage that precedes SVD in correspondence analysis (CA), replacing the log(x+1) + PCA transformation used in typical single-cell workflows.

## When to use

Use when you have a raw scRNAseq count matrix and need the chi-squared residual matrix for CA dimension reduction, or when verifying a re-implementation of the corral_preproc count-to-residual pre-processing stage element-wise.

## When NOT to use

- The downstream analysis already requires a variance-stabilized or power-deflated residual variant (e.g. Freeman–Tukey residuals, square-root/Anscombe VST, or signed power deflation) rather than the plain standardized Pearson residuals.
- The goal is batch/multi-table integration across samples — corralm, not the single-table residual pre-processing, is the appropriate entry point.
- The pipeline expects the conventional log(x + 1) 'logcounts' transformation followed by PCA instead of CA on chi-squared residuals.

## Inputs

- scRNAseq count matrix loaded in R/Bioconductor as a SingleCellExperiment (or comparable count table)

## Outputs

- Standardized Pearson chi-squared residual matrix (same dimensions as the input count matrix, entries = (observed − expected)/sqrt(expected))
- preproc_verification_results.csv with maximum absolute element-wise differences and pass/fail status per transformation (verification mode)

## How to apply

Load the count matrix into R/Bioconductor as a SingleCellExperiment. Compute expected values from the row and column marginals, then form each entry of the residual matrix as (observed − expected) / sqrt(expected), the standardized Pearson residual. CA is then computed by SVD of this Pearson residual matrix, avoiding the distortive log-transformation of typical workflows. When this skill is used for verification, run corral::corral_preproc with rtype='Pearson' (or the corresponding rtype/vst_mth setting) on the identical count matrix, compare each re-implemented matrix element-wise, record maximum absolute differences and pass/fail in preproc_verification_results.csv; accept only if the outputs agree within numerical tolerance.

## Related tools

- **corral** (Reference R/Bioconductor package whose corral_preproc count-to-residual pre-processing (rtype setting) provides the ground-truth Pearson residual matrix for element-wise verification) — https://github.com/laurenhsu1/corral
- **R** (Implementation language for re-computing the standardized Pearson residuals from row and column marginals)
- **Bioconductor** (Provides SingleCellExperiment as the common test-input container for the scRNAseq count matrix)

## Examples

```
resid <- corral::corral_preproc(sce, rtype = 'Pearson')
```

## Evaluation signals

- Element-wise agreement of the re-implemented Pearson residual matrix with corral::corral_preproc output (rtype='Pearson') on the identical count matrix within numerical tolerance
- Maximum absolute difference between the re-implementation and corral_preproc counterpart recorded in preproc_verification_results.csv passes the tolerance check
- Each residual entry equals (observed − expected)/sqrt(expected) with expectations derived from row and column marginals of the same count matrix
- The residual matrix is a dense numeric matrix with the same dimensions as the input count matrix (no log-transform applied)
- SVD of the residual matrix yields CA embeddings whose row–column associations deviate from independence expectation

## Limitations

- Standard CA (Pearson residuals) does not address overdispersion in scRNAseq counts; the article's five adaptations (including power deflation and Freeman–Tukey residuals) were proposed precisely to address overdispersion and CA-FT was most performant overall in the cluster recovery task.
- No tool or package versions are reported for corral or its dependencies, so numerical tolerance comparisons may be affected by version drift.
- The residual matrix is unbounded relative to logcounts, so downstream clustering/ARI performance depends on dataset characteristics (e.g. simple pseudo-cell mixtures vs complex tissues).

## Evidence

- [abstract] CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation: "CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation"
- [intro] CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [full_text] Workflow: compute standardized Pearson residual matrix from marginals, then compare against corral_preproc: [evidence span withheld — anchor status: not_found]
- [full_text] Run corral::corral_preproc with each corresponding rtype/vst_mth setting on the identical count matrix and compare element-wise within numerical tolerance: [evidence span withheld — anchor status: not_found]
- [readme] corral interfaces directly with single cell classes in Bioconductor: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
