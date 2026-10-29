---
name: single-cell-hidden-factor-detection
description: Use when scRNA-seq count data appears homogeneous under known covariates
  (e.g., a single cell type within patients) but unexplained heterogeneity — such
  as doublets, cell-cycle stage, or outlier cells — is suspected and PCA/tSNE fails
  to separate it.
when_to_use_negative:
- When all sources of unwanted variation are already known and fully captured by the
  design matrix covariates (patient identity, geometric library size) — plain regression
  or SVA suffices
- When the goal is cell-type assignment across a mixed population where PCA or ZINB-WaVe
  may suffice; IA-SVA is designed for hidden factors correlated with known covariates
  within seemingly homogeneous cells
- When expecting many unknown factors without a stringent SI cut-off strategy — the
  article warns the SI is not an empirical p-value and users must select a stringent
  cut-off in that setting
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: iasva
  role: 'Core R/Bioconductor package implementing iteratively adjusted SVA: regress
    residuals, SVD for PC1, permutation-based significance testing, and iterative
    addition of significant SVs as known factors'
  repo: https://github.com/UcarLab/iasva
- name: IRLBA
  role: Augmented implicitly restarted Lanczos bidiagonalization algorithm used to
    compute the leading singular vectors (PC1) efficiently during IA-SVA's SVD step
- name: fast_iasva
  role: Fast IA-SVA mode that selects important SVs based on the proportion of variation
    explained instead of full permutation testing
  repo: https://github.com/UcarLab/iasva
- name: Seurat
  role: Provides Spectral tSNE visualization used to confirm that IA-SVA-derived factors
    (and associated marker genes) separate hidden cell subpopulations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:07f13e86cbc0a6d4c0821977c312c102c58ab7adf8f4745a9a4cb3c6d2258f48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/iasva
  tools:
  - IRLBA
  - Seurat
  - fast_iasva
  - iasva
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-hidden-factor-detection

## Summary

Detects hidden sources of variation in single-cell RNA-seq data that are correlated with known covariates, using Iteratively Adjusted Surrogate Variable Analysis (IA-SVA). Unlike SVA/USVA, IA-SVA iteratively identifies genes associated with each hidden factor and uses a permutation-based significance index to decide how many surrogate variables to retain.

## When to use

Use when scRNA-seq count data appears homogeneous under known covariates (e.g., a single cell type within patients) but unexplained heterogeneity — such as doublets, cell-cycle stage, or outlier cells — is suspected and PCA/tSNE fails to separate it.

## When NOT to use

- When all sources of unwanted variation are already known and fully captured by the design matrix covariates (patient identity, geometric library size) — plain regression or SVA suffices
- When the goal is cell-type assignment across a mixed population where PCA or ZINB-WaVe may suffice; IA-SVA is designed for hidden factors correlated with known covariates within seemingly homogeneous cells
- When expecting many unknown factors without a stringent SI cut-off strategy — the article warns the SI is not an empirical p-value and users must select a stringent cut-off in that setting

## Inputs

- Raw scRNA-seq read count matrix (genes x cells), e.g., iasva_counts_test.Rds (101 healthy alpha cells from Lawlor et al., Fluidigm C1)
- Sample annotation table with covariates such as patient_id and library size, e.g., iasva_anns_test.Rds
- Design matrix of known factors built via model.matrix(~patient_id + geo_lib_size) with the intercept column dropped

## Outputs

- Surrogate variable (SV) scores per cell, e.g., SV1 values for all 101 cells written to iasva_sv1_results.csv
- Permutation-based significance index (SI) for each detected SV
- Genes significantly associated with each SV (adjusted p < 0.05, R^2 > 0.3) usable for marker analysis and visualization

## How to apply

Load a raw read-count matrix (e.g., iasva_counts_test.Rds with 101 alpha cells) and sample annotations, then compute the geometric library size as colSums(log(counts + 1)) and confirm its high correlation with PC1 of the log-transformed counts. Build a design matrix with model.matrix(~patient_id + geo_lib_size), drop the intercept, wrap the counts in a SummarizedExperiment, and run iasva() with num.sv = 5 so the algorithm iteratively regresses out known factors, performs SVD on the residuals to obtain PC1, and applies the permutation-based quasi-inferential procedure (T_obs = λ1^2 / Σ λk^2, α = 0.05, 50 permutations) to assign each SV a significance index. Threshold the first SV to isolate hidden subpopulations — e.g., cells with SV1 > -0.1 in the Lawlor alpha-cell data flag 6 outlier cells that PCA, USVA, and tSNE did not clearly separate. Genes significantly associated with each SV (adjusted p-value < 0.05, R-squared > 0.3 via find_markers) can be carried into downstream visualization (e.g., tSNE via Seurat) to confirm the clustering improvement.

## Related tools

- **iasva** (Core R/Bioconductor package implementing iteratively adjusted SVA: regress residuals, SVD for PC1, permutation-based significance testing, and iterative addition of significant SVs as known factors) — https://github.com/UcarLab/iasva
- **IRLBA** (Augmented implicitly restarted Lanczos bidiagonalization algorithm used to compute the leading singular vectors (PC1) efficiently during IA-SVA's SVD step)
- **fast_iasva** (Fast IA-SVA mode that selects important SVs based on the proportion of variation explained instead of full permutation testing) — https://github.com/UcarLab/iasva
- **Seurat** (Provides Spectral tSNE visualization used to confirm that IA-SVA-derived factors (and associated marker genes) separate hidden cell subpopulations)

## Examples

```
svd.res <- iasva(counts_se, design_mat[, -1], num.sv = 5, permute = TRUE); outlier_cells <- colnames(counts_se)[svd.res$sv[, 1] > -0.1]
```

## Evaluation signals

- Thresholding SV1 at -0.1 on the Lawlor alpha-cell data reproduces exactly 6 outlier cells separated from the remaining 95 alpha cells
- The results table contains one SV1 value per cell (101 rows) plus a significance index for the SV
- The geometric library size (colSums(log(counts + 1))) shows high correlation (r ~ 0.99) with PC1 of log-transformed counts, confirming it was properly accounted for
- Empirical Type I error on simulated data should be ~0.04 (vs 0.09 for USVA and SSVA) at α = 0.05 with 50 permutations
- Marker genes associated with SV1 recapitulate known biology (e.g., glucagon for alpha cells, cell-cycle genes via SCRAN predictions), and downstream tSNE clusters improve after IA-SVA feature selection

## Limitations

- The significance index (SI) is not an empirical p-value, and caveats remain in determining the number of hidden factors
- Users must select a stringent SI cut-off, especially when many unknown factors are expected, but no specific cut-off threshold is provided in the article
- Compromised alpha cells (possible cell contamination) detected by IA-SVA should be excluded from downstream analyses rather than interpreted as novel biology
- No software versions are stated for companion visualization tools (Seurat, ZINB-WaVe, Cell View, tSNE) used alongside IA-SVA

## Evidence

- [readme] Iteratively Adjusted Surrogate Variable Analysis (IA-SVA) is a statistical framework to uncover hidden sources of variation even when these sources are correlated with the biological variable of interest.: "Iteratively Adjusted Surrogate Variable Analysis (IA-SVA) is a statistical framework to uncover hidden sources of variation even when these sources are correlated"
- [methods] Step 1 regress residuals: [evidence span withheld — anchor status: not_found]
- [methods] Step 2 SVD on residuals: "Conduct SVD on the obtained residuals ( ′ × Y m n ) and obtain the first PC (PC1)"
- [results] geometric library size as covariate: [evidence span withheld — anchor status: not_found]
- [methods] alpha cell dataset definition: "we use read counts of alpha cells from healthy (non-diabetic) subjects (n = 101)"
- [discussion] compromised cells caveat: "IA-SVA detects heterogeneity stemming from compromised alpha cells (which may be explained by cell contamination), which should be excluded from the downstream analyses"
