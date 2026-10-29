---
name: sex-covariate-batch-adjusted-design-matrix-construction
description: Use when running multi-sample pseudotime differential testing (e.g.,
  lamian_test_h5 with testvar=2) on a multi-sample scRNA-seq atlas where a biological
  covariate such as sex must be tested while batch/technical variables are adjusted
  for.
when_to_use_negative:
- When each condition has only one sample per batch level, batch indicator columns
  are not estimable jointly with the sex covariate
- When the goal is topology testing or trajectory inference rather than covariate-adjusted
  expression/abundance testing along pseudotime
- When samples have fewer than 1000 cells or batches lack both male and female samples,
  as these are filtered by the paper's criteria
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0203
tools:
- name: Lamian
  role: Provides saveh5(), lamian_test_h5, and cellPropTest, which consume the intercept+sex+batch
    design matrix to test sex (testvar=2) while adjusting for batch columns
  repo: https://github.com/Winnie09/Lamian
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:5bbee66776f96e2096c6a82468e3c0019b77ed2e7b2f975d545f5e386feb94fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sex-covariate-batch-adjusted-design-matrix-construction

## Summary

Constructs a sample-level design matrix whose first column is the intercept, second column is sex, and remaining columns are batch indicators, so that Lamian can test sex-associated XDE/XCD changes along NAM-PC2 pseudotime in the TB atlas (337,191 memory T cells, 184 donors, GSE158769) while adjusting for batch effects.

## When to use

Use when running multi-sample pseudotime differential testing (e.g., lamian_test_h5 with testvar=2) on a multi-sample scRNA-seq atlas where a biological covariate such as sex must be tested while batch/technical variables are adjusted for.

## When NOT to use

- When each condition has only one sample per batch level, batch indicator columns are not estimable jointly with the sex covariate
- When the goal is topology testing or trajectory inference rather than covariate-adjusted expression/abundance testing along pseudotime
- When samples have fewer than 1000 cells or batches lack both male and female samples, as these are filtered by the paper's criteria

## Inputs

- cell-to-sample annotation for GSE158769 memory T cells (184 samples)
- sample-level sex labels (male/female)
- sample-level batch identifiers (TB batches)
- NAM-PC2 pseudotime values per cell
- expression matrix GSE158769_exprs_raw.tsv.gz (optionally converted to hdf5 via Lamian's saveh5())

## Outputs

- sample-level design matrix: column 1 = intercept, column 2 = sex, remaining columns = batch indicators
- per-gene XDE statistics including fdr.overall from lamian_test_h5 with testvar=2
- XCD result (pval.overall, z.overall) from cellPropTest with testvar=2

## How to apply

Assemble a sample-level annotation table for the GSE158769 TBRU dataset and build the design matrix with the intercept in column 1, sex in column 2 (the testvar), and one binary indicator column per retained batch. Before building it, apply the paper's sample/batch filters: retain only batches with at least one male and one female sample, and retain samples with at least 1000 cells whose genes have expression values > 0.1 in at least 1% of cells — this guarantees every batch column is estimable jointly with sex. Pass this design to lamian_test_h5 (test.type='variable', test.method='permutation', permuiter=100) so sex is tested while all batch columns are conditioned out, and to cellPropTest (test.type='Variable', testvar=2) for the XCD test. The rationale is that including batch indicators in the Lamian regression yields more reproducible XDE gene calls across data splits (Fig. 6h) at atlas scale.

## Related tools

- **Lamian** (Provides saveh5(), lamian_test_h5, and cellPropTest, which consume the intercept+sex+batch design matrix to test sex (testvar=2) while adjusting for batch columns) — https://github.com/Winnie09/Lamian

## Examples

```
design <- data.frame(intercept=1, sex=sampleInfo$sex, batch=sampleInfo$batch); res <- lamian_test_h5(exprs.h5, cellanno, pseudotime, design=design, test.type='variable', testvar=2, test.method='permutation')
```

## Evaluation signals

- Design matrix dimensionality equals number of retained samples (184 pre-filtering) with one binary column per retained batch plus intercept and sex
- Every batch column contains at least one male and at least one female sample after filtering
- XCD pval.overall reaches the reported significance of p = 8.9e-6 for sex
- Number of sex-associated XDE genes (fdr.overall < 0.05) approximates the reported 1120 genes across 14 clusters
- Batch-corrected XDE calls show higher cross-split reproducibility than uncorrected calls across the 12 data splits (consistent with Fig. 6h)

## Limitations

- Lamian accounts for cross-sample variability but a systematic comparison of harmonization methods is beyond the scope of the study
- Pseudotime/pca inputs to Lamian modules must come from supported tools (TSCAN-based default); Monocle2/Monocle3/Slingshot trajectories are not supported in modules 1 and 2
- Ultra-large matrices (337,191 cells) require hdf5 storage via saveh5() for memory-efficient fitting

## Evidence

- [discussion] Retain only batches with at least one male and one female sample (TB): "Batches with at least one male sample and at least one female sample were retained."
- [discussion] Retain samples with at least 1000 cells and genes expressed in at least 1% of cells (TB): "Samples with at least 1000 cells and genes with expression values > 0.1 in at least 1% of cells were retained."
- [abstract] Lamian adjusts for batch effects when identifying covariate-associated changes: "Lamian can be used to identify changes in a biological process associated with sample covariates, such as different biological conditions while adjusting for batch effects"
- [readme] Lamian takes multiple samples with covariates as input: "Given scRNA-seq data from multiple biological samples with covariates (e.g., age, sex, sample type, disease status, etc.)"
