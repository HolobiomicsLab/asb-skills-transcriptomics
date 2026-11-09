---
name: linear-covariate-adjustment-scrnaseq
description: Use when a scRNA-seq count matrix has known unwanted covariates (e.g.,
  patient/batch ID, library size) that must be adjusted for before inferring hidden
  factors or surrogate variables.
when_to_use_negative:
- When the only goal is visualization and no known covariates need adjustment — plain
  PCA may suffice, though note PCA's PC1 was discarded since it is highly correlated
  (r = 0.99) with geometric library size
- When hidden factors are assumed orthogonal to all known factors — plain SVA (SSVA)
  or USVA may be used, though they deliver biased estimates under correlation
- When input is already normalized/adjusted data with covariates regressed out — re-adjusting
  would remove genuine signal
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: iasva
  role: R/Bioconductor package that iteratively adjusts for known covariates and detects
    significant hidden SVs via permutation-based testing
  repo: https://github.com/UcarLab/iasva
- name: IRLBA
  role: Augmented implicitly restarted Lanczos bidiagonalization algorithm used for
    the SVD step on the residuals within IA-SVA
- name: fast_iasva
  role: Fast IA-SVA variant that selects important SVs based on the proportion of
    variation explained
  repo: https://github.com/UcarLab/iasva
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:c57a605e247e568af43de9bb27bc9680555e59ed02d57eda55aece05b0768dcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/iasva
  tools:
  - IRLBA
  - fast_iasva
  - iasva
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# linear-covariate-adjustment-scRNAseq

## Summary

Build a design matrix of known covariates (patient identity and geometric library size) and regress them out before hidden-factor detection with IA-SVA, so that residual SVD captures only unmodeled heterogeneity in scRNA-seq count matrices.

## When to use

Use when a scRNA-seq count matrix has known unwanted covariates (e.g., patient/batch ID, library size) that must be adjusted for before inferring hidden factors or surrogate variables.

## When NOT to use

- When the only goal is visualization and no known covariates need adjustment — plain PCA may suffice, though note PCA's PC1 was discarded since it is highly correlated (r = 0.99) with geometric library size
- When hidden factors are assumed orthogonal to all known factors — plain SVA (SSVA) or USVA may be used, though they deliver biased estimates under correlation
- When input is already normalized/adjusted data with covariates regressed out — re-adjusting would remove genuine signal

## Inputs

- Read count matrix for a single cell type (e.g., iasva_counts_test.Rds, 101 healthy human alpha cells)
- Sample/cell annotations with patient/batch identity (e.g., iasva_anns_test.Rds)
- Design matrix of known covariates (patient_id, geometric library size), intercept dropped

## Outputs

- Surrogate variable (SV) values per cell (e.g., SV1 scores for 101 alpha cells)
- Permutation-based significance index (SI) for each SV
- Outlier grouping from thresholding SVs (e.g., SV1 > -0.1 → 6 outlier cells vs 95 cells)

## How to apply

Compute the geometric library size per cell as geo_lib_size <- colSums(log(counts + 1)) and confirm its high correlation with PC1 of the log-transformed counts (r = 0.99 in the Lawlor islet data). Build the design matrix with model.matrix(~patient_id + geo_lib_size) and drop the intercept column before passing it to iasva(); for every method except PCA, patient identity and geometric library size were used as covariates. Wrap the read counts in a SummarizedExperiment and run iasva() with num.sv = 5; the algorithm iteratively regresses Y on all known factors (X) — including SVs from previous iterations — to obtain residuals, then runs SVD on the residuals to obtain PC1, whose significance is judged by the permutation-based statistic T_obs = λ1^2 / Σ λk^2. Iterative adjustment is what lets IA-SVA estimate hidden factors even when they are correlated with the known covariates.

## Related tools

- **iasva** (R/Bioconductor package that iteratively adjusts for known covariates and detects significant hidden SVs via permutation-based testing) — https://github.com/UcarLab/iasva
- **IRLBA** (Augmented implicitly restarted Lanczos bidiagonalization algorithm used for the SVD step on the residuals within IA-SVA)
- **fast_iasva** (Fast IA-SVA variant that selects important SVs based on the proportion of variation explained) — https://github.com/UcarLab/iasva

## Examples

```
iasva.res <- iasva(SummarizedExperiment(counts), model.matrix(~patient_id + geo_lib_size)[,-1], num.sv = 5, permuteit = TRUE)
```

## Evaluation signals

- Geometric library size must show high correlation with PC1 of log-transformed counts (r = 0.99 in the alpha-cell data), justifying its inclusion as a covariate
- Output table must contain one SV1 value per cell (101 rows) plus a significance index
- Thresholding SV1 at -0.1 must reproduce the reported separation of 6 outlier cells from the remaining 95 alpha cells
- IA-SVA's empirical Type I error rate should be controlled (0.04 at α = 0.05, 50 permutations in simulations)
- Detected SVs should separate cells that PCA, USVA, and tSNE failed to clearly distinguish

## Limitations

- The significance index (SI) is not an empirical p-value; caveats remain in determining the number of hidden factors, and a stringent SI cut-off is recommended when many unknown factors are expected (no specific threshold provided)
- Software versions for companion tools (Seurat, ZINB-WaVe, Cell View, tSNE) are not stated
- Benchmark datasets for systematic comparison of methods are not yet established (pending Human Cell Atlas consortium effort)

## Evidence

- [results] patient identity and geometric library size as covariates: [evidence span withheld — anchor status: not_found]
- [results] PC1 correlated with library size: "PC1 was discarded since it is highly correlated (r = 0.99) with the geometric library size"
- [methods] iterative residual regression and SVD: [evidence span withheld — anchor status: not_found]
- [abstract] IA-SVA correlated hidden factors: "We developed iteratively adjusted surrogate variable analysis (IA-SVA) that can estimate hidden factors even when they are correlated with other sources of variation by identifying a set of genes"
- [readme] package installation: [evidence span withheld — anchor status: external_doc]
