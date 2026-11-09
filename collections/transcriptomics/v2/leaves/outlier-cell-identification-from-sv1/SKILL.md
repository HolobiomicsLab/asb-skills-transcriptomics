---
name: outlier-cell-identification-from-sv1
description: Use when a scRNA-seq cell population appears homogeneous but you suspect
  hidden heterogeneity (e.g., compromised/doublet cells), and known covariates (patient
  identity, geometric library size) fail to explain the residual variation.
when_to_use_negative:
- The goal is de novo cell-type clustering of a mixed population — IA-SVA here is
  for hidden factors within one ostensibly homogeneous cell type, not cell-type discovery
- A known factor (e.g., patient identity or library size) already fully explains the
  variation; PC1 highly correlated with geometric library size should be discarded
  rather than interpreted as biology
- The SV being examined is not significant by the permutation-based significance index;
  the paper cautions SI is not an empirical p-value and a stringent SI cut-off is
  recommended when many unknown factors are expected
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: iasva
  role: 'Core algorithm: iteratively regresses out known factors (patient_id, geo_lib_size,
    prior SVs), runs SVD on residuals, and tests each hidden factor via permutation
    to produce SV1 and significance index'
  repo: https://github.com/UcarLab/iasva
- name: IRLBA
  role: Augmented implicitly restarted Lanczos bidiagonalization algorithm used for
    the efficient partial SVD of residual matrices inside IA-SVA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:6bbc728675c942b61ee1551e6b4753eb957ab6fd4b887e1c0edb0af8b6a9edc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/iasva
  tools:
  - IRLBA
  - iasva
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# outlier-cell-identification-from-SV1

## Summary

Identifies hidden outlier cells within a seemingly homogeneous single-cell population by thresholding the first surrogate variable (SV1) inferred with IA-SVA, as demonstrated on 101 human alpha cells from the Lawlor islet dataset where 6 outlier cells were separated from 95 normal cells.

## When to use

Use when a scRNA-seq cell population appears homogeneous but you suspect hidden heterogeneity (e.g., compromised/doublet cells), and known covariates (patient identity, geometric library size) fail to explain the residual variation.

## When NOT to use

- The goal is de novo cell-type clustering of a mixed population — IA-SVA here is for hidden factors within one ostensibly homogeneous cell type, not cell-type discovery
- A known factor (e.g., patient identity or library size) already fully explains the variation; PC1 highly correlated with geometric library size should be discarded rather than interpreted as biology
- The SV being examined is not significant by the permutation-based significance index; the paper cautions SI is not an empirical p-value and a stringent SI cut-off is recommended when many unknown factors are expected

## Inputs

- Raw read count matrix (Rds, e.g. iasva_counts_test.Rds) for a single cell type (n = 101 alpha cells)
- Cell-level sample annotations (Rds, e.g. iasva_anns_test.Rds) including patient_id
- Design covariates: patient identity and geometric library size

## Outputs

- Per-cell SV1 values (one row per cell, 101 rows) written to iasva_sv1_results.csv
- Significance index for each inferred SV
- Binary outlier/normal cell grouping (SV1 > -0.1 → 6 outliers vs 95 remaining cells)

## How to apply

Load the alpha cell read count matrix (iasva_counts_test.Rds, 101 cells) and annotations (iasva_anns_test.Rds) from the iasva package. Compute the geometric library size as colSums(log(counts + 1)) and confirm its high correlation (r = 0.99 reported for PC1) with library size. Build the design matrix with model.matrix(~patient_id + geo_lib_size), drop the intercept column, wrap counts in a SummarizedExperiment, and run iasva() with num.sv = 5 using the permutation-based procedure (α = 0.05, 50 permutations; test statistic T_obs = λ1^2 / Σ λk^2) to obtain the significance index. Extract per-cell SV1 values and split cells at SV1 > -0.1 into an outlier group; the paper reproduces 6 outlier cells versus 95 remaining alpha cells, a separation that PCA, USVA, and tSNE failed to clearly detect. Outlier (compromised) cells should then be excluded from downstream analyses.

## Related tools

- **iasva** (Core algorithm: iteratively regresses out known factors (patient_id, geo_lib_size, prior SVs), runs SVD on residuals, and tests each hidden factor via permutation to produce SV1 and significance index) — https://github.com/UcarLab/iasva
- **IRLBA** (Augmented implicitly restarted Lanczos bidiagonalization algorithm used for the efficient partial SVD of residual matrices inside IA-SVA)

## Examples

```
iasva_out <- iasva(se, mod = model.matrix(~patient_id + geo_lib_size)[, -1], num.sv = 5, permute = TRUE)
```

## Evaluation signals

- iasva_sv1_results.csv contains exactly 101 rows (one SV1 value per cell) plus a significance index column
- Thresholding SV1 at -0.1 yields exactly 6 outlier cells and 95 remaining alpha cells, matching the reported separation
- Geometric library size colSums(log(counts + 1)) shows high correlation (r ≈ 0.99) with PC1 of log-transformed counts, confirming covariate choice
- The top two significant SVs separate alpha cells into two groups, whereas PCA, USVA, and tSNE fail to clearly detect the same outliers
- Permutation settings match the paper: α = 0.05 with 50 permutations

## Limitations

- The significance index is not an empirical p-value; despite its caveats it is still useful for determining the number of hidden factors
- Users must select a stringent SI cut-off when many unknown factors are expected, but no specific cut-off threshold is provided
- Detected outliers may reflect compromised cells (possible cell contamination) and should be excluded from downstream analyses rather than treated as new biology
- No software versions are stated for comparison tools (Seurat, ZINB-WaVe, Cell View, tSNE), complicating exact reproduction

## Evidence

- [discussion] IA-SVA detects heterogeneity stemming from compromised alpha cells (which may be explained by cell contamination), which should be excluded from the downstream analyses: "IA-SVA detects heterogeneity stemming from compromised alpha cells (which may be explained by cell contamination), which should be excluded from the downstream analyses"
- [methods] we use read counts of alpha cells from healthy (non-diabetic) subjects (n = 101): "we use read counts of alpha cells from healthy (non-diabetic) subjects (n = 101)"
- [results] For each method (except for PCA), patient identity and geometric library size were used as covariates: [evidence span withheld — anchor status: not_found]
- [discussion] its permutation-based quasi-inferential procedure to determine the number of hidden factors similar to the permutation-based parallel analysis: "its permutation-based quasi-inferential procedure to determine the number of hidden factors similar to the permutation-based parallel analysis"
- [readme] To install IA-SVA package, start R and enter the following commands: devtools::install_github("UcarLab/iasva"): "To install IA-SVA package, start R and enter the following commands:

      library(devtools)
      devtools::install_github("UcarLab/iasva")"
