---
name: svd-based-principal-component-extraction
description: Use when analyzing a scRNA-seq count matrix (e.g., Lawlor alpha cells,
  101 cells) and you need the singular vectors/PCs — e.
when_to_use_negative:
- When the leading PC is already explained by a known covariate (e.g., PC1 correlating
  r = 0.99 with geometric library size in the alpha-cell data) — it should be discarded
  and used as a known factor instead of being treated as a hidden factor
- When the goal is detecting hidden factors correlated with known factors, use full
  IA-SVA rather than a single plain SVD step
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
tools:
- name: IRLBA
  role: Truncated SVD engine (augmented implicitly restarted Lanczos bidiagonalization)
    used to compute PC1 of log-transformed, column-centered counts and of IA-SVA residuals
    efficiently
  repo: https://bwlewis.github.io/irlba/
- name: iasva
  role: Package whose six-step procedure embeds two SVD steps — SVD on known-factor-adjusted
    residuals to get PC1 and a second SVD on the R-squared-weighted matrix to obtain
    the SV
  repo: https://github.com/UcarLab/iasva
- name: SummarizedExperiment
  role: Container for the alpha-cell read counts and sample annotations passed to
    the iasva() call
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:ff6835baff64e7d5e80f99831c4ad79d1763ce70d5fe3c1c0a1211bb1332c605
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IRLBA
  - SummarizedExperiment
  - iasva
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# svd-based-principal-component-extraction

## Summary

Extracts leading principal components from single-cell count matrices via singular value decomposition (using the IRLBA implicitly restarted Lanczos bidiagonalization algorithm), both on residuals of known-factor-adjusted counts (step 2 of IA-SVA) and on the R-squared-weighted matrix to obtain the surrogate variable.

## When to use

Use when analyzing a scRNA-seq count matrix (e.g., Lawlor alpha cells, 101 cells) and you need the leading singular vectors/PCs — e.g., to identify the dominant axis of variation, test whether PC1 tracks a known covariate like geometric library size, or supply PC1 to a hidden-factor procedure such as IA-SVA.

## When NOT to use

- When the leading PC is already explained by a known covariate (e.g., PC1 correlating r = 0.99 with geometric library size in the alpha-cell data) — it should be discarded and used as a known factor instead of being treated as a hidden factor
- When the goal is detecting hidden factors correlated with known factors, use full IA-SVA rather than a single plain SVD step

## Inputs

- scRNA-seq gene-by-cell read count matrix (e.g., iasva_counts_test.Rds, 101 Lawlor alpha cells)
- Residual matrix Y' of counts adjusted for known factors (patient_id, geometric library size, prior SVs)
- R-squared-weighted matrix Y'' = Y * W from IA-SVA gene identification

## Outputs

- PC1 (first right singular vector) of the input matrix or residual matrix
- Singular values lambda_k used for the significance statistic T = lambda1^2 / sum_k(lambda_k^2)
- Surrogate variable (SV) estimate obtained from the second SVD on the weighted matrix

## How to apply

Log-transform the read counts (log(counts+1)), column-center the matrix (subtract row means), and run truncated SVD with the IRLBA package to take only the first right singular vector, e.g. irlba(lcounts - rowMeans(lcounts), 1)$v[, 1]. Within the IA-SVA procedure, first regress the m×n log-transformed count matrix Y on all known factors (X, including SVs from previous iterations) to obtain residuals Y', then conduct SVD on Y' and take PC1, whose significance is scored by the permutation statistic T = lambda1^2 / sum_k(lambda_k^2). If significant, PC1 is used to identify associated genes (per-gene R-squared from regressing Y' on PC1), and a second SVD on the weighted matrix Y'' = Y * W (W diagonal of R-squared values) yields the surrogate variable. Sanity-check PC1 against known covariates: in the Lawlor alpha-cell data PC1 was discarded because it correlates at r = 0.99 with the geometric library size (colSums of log(counts+1)), so such a correlation justifies treating library size as a known factor rather than a hidden one.

## Related tools

- **IRLBA** (Truncated SVD engine (augmented implicitly restarted Lanczos bidiagonalization) used to compute PC1 of log-transformed, column-centered counts and of IA-SVA residuals efficiently) — https://bwlewis.github.io/irlba/
- **iasva** (Package whose six-step procedure embeds two SVD steps — SVD on known-factor-adjusted residuals to get PC1 and a second SVD on the R-squared-weighted matrix to obtain the SV) — https://github.com/UcarLab/iasva
- **SummarizedExperiment** (Container for the alpha-cell read counts and sample annotations passed to the iasva() call)

## Examples

```
lcounts <- log(counts + 1); pc1 <- irlba(lcounts - rowMeans(lcounts), 1)$v[, 1]; cor(geo_lib_size, pc1)
```

## Evaluation signals

- Pearson correlation between the geometric library size (colSums of log(counts+1)) and PC1 should be very high (~r = 0.99 on the 101 Lawlor alpha cells)
- Reimplemented SV1 from the residual-SVD/weighted-SVD procedure should correlate near 1 with the reference iasva() package SV1 output (paper reports average correlation 0.926 over 100 runs for the brain cell-type SV)
- PC1 significance statistic T = lambda1^2 / sum_k(lambda_k^2) computed against permuted columns of Y should flag the component as significant before proceeding to gene identification
- The truncated SVD must request only k = 1 component via irlba, matching the single leading singular vector, not a full decomposition

## Limitations

- PC1 of raw log-counts may simply reflect library size rather than biology, as in the alpha-cell data (r = 0.99 with geometric library size), requiring its exclusion
- A high R-squared R^2 > 0.3 and adjusted p-value < 0.05 are needed for genes fed into the second SVD; too lenient a cutoff dilutes the surrogate variable
- Significance index (SI) caveats remain for determining the number of hidden factors; a stringent SI cut-off is recommended but no specific threshold is provided

## Evidence

- [methods] Step 2: conduct SVD on residuals and obtain PC1: "Conduct SVD on the obtained residuals ( ′ × Y m n ) and obtain the first PC (PC1)"
- [results] IRLBA algorithm used for truncated SVD: [evidence span withheld — anchor status: not_found]
- [results] PC1 discarded due to library-size correlation: "PC1 was discarded since it is highly correlated (r = 0.99) with the geometric library size"
- [readme] IA-SVA purpose statement: "uncover hidden sources of variation even when these sources are correlated with the biological variable of interest"
