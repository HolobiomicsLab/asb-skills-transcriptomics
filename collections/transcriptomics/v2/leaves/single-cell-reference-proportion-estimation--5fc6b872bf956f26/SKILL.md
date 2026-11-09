---
name: single-cell-reference-proportion-estimation
description: Use when you have bulk RNA-seq read counts from complex tissue (e.g.,
  89 Fadista pancreatic islet donors) and a multi-subject scRNA-seq reference (e.g.,
  E-MTAB-5060 Segerstolpe data), and need cell type proportions to test trait associations.
when_to_use_negative:
- Bulk expression input is TPM/normalized values without raw counts — cell type proportions
  cannot be estimated with TPM as the input
- Target cell types are extremely rare or highly collinear without a cluster-aware
  strategy — MuSiC's basic W-NNLS struggles with collinearity among closely related
  cell types
- The scRNA-seq reference is a single-subject library without pre-selected markers
  — the cross-subject MuSiC weighting cannot be exploited and existing marker-gene
  pitfalls apply
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MuSiC
  role: Performs weighted non-negative least squares deconvolution (music_prop / music_prop.cluster)
    of bulk proportions using cross-subject scRNA-seq weighting
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: Implementation environment; MuSiC is installed as a GitHub R/Bioconductor-style
    package and lm is used for the HbA1c regression
- name: ggplot2
  role: Visualizes the proportion-HbA1c association with geom_smooth(method = 'lm')
    for regression inspection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:d6eb848cdb4acaa5b7813113135ddbadd05341b9033f7d272e7f7300c7cbc9f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - MuSiC
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# single-cell-reference-proportion-estimation

## Summary

Estimates cell type proportions in bulk RNA-seq samples by MuSiC weighted non-negative least squares (W-NNLS) deconvolution against a multi-subject scRNA-seq reference, enabling association analyses of cell composition with clinical phenotypes such as HbA1c in pancreatic islet data.

## When to use

Use when you have bulk RNA-seq read counts from complex tissue (e.g., 89 Fadista pancreatic islet donors) and a multi-subject scRNA-seq reference (e.g., E-MTAB-5060 Segerstolpe data), and need cell type proportions to test trait associations.

## When NOT to use

- Bulk expression input is TPM/normalized values without raw counts — cell type proportions cannot be estimated with TPM as the input
- Target cell types are extremely rare or highly collinear without a cluster-aware strategy — MuSiC's basic W-NNLS struggles with collinearity among closely related cell types
- The scRNA-seq reference is a single-subject library without pre-selected markers — the cross-subject MuSiC weighting cannot be exploited and existing marker-gene pitfalls apply

## Inputs

- GSE50244bulkeset.rds — bulk pancreatic islet read counts ExpressionSet (32581 features, 89 samples)
- EMTABesethealthy.rds — scRNA-seq reference ExpressionSet (25453 genes, 1097 cells) with cellType and sampleID metadata
- Subject phenotype table (age, bmi, hba1c, gender)

## Outputs

- Est.prop.weighted — per-subject weighted cell type proportion estimates (six non-negative proportions per donor, normalized over included cell types)
- beta_hba1c_regression_results.csv — regression slope, standard error, and p-value for beta cell proportion vs HbA1c

## How to apply

Load the bulk read counts (GSE50244bulkeset.rds, 32581 features x 89 samples) and single-cell reference (EMTABesethealthy.rds, 25453 genes x 1097 cells) as Biobase ExpressionSet objects. Run music_prop with clusters='cellType', samples='sampleID', and select.ct limited to the six major islet cell types (alpha, beta, delta, gamma, acinar, ductal) that make up over 90% of the islet, obtaining weighted proportions constrained to sum to one. MuSiC works by W-NNLS without pre-selected marker genes, weighting genes for cross-subject and cross-cell consistency, with an optional gene- and subject-specific intercept to adjust protocol bias; input must be raw counts (TPM is not estimable, RPKM only with external RNA abundance estimates). Join the estimated beta proportion with phenotypes (age, bmi, hba1c, gender), drop rows with missing HbA1c, and fit a single-variable lm of beta cell proportion on HbA1c to extract slope, standard error, and p-value. Use the tree-guided variant music_prop.cluster (hierarchical clustering then W-NNLS within clusters) when closely related cell types cause collinearity.

## Related tools

- **MuSiC** (Performs weighted non-negative least squares deconvolution (music_prop / music_prop.cluster) of bulk proportions using cross-subject scRNA-seq weighting) — https://github.com/xuranw/MuSiC
- **R** (Implementation environment; MuSiC is installed as a GitHub R/Bioconductor-style package and lm is used for the HbA1c regression)
- **ggplot2** (Visualizes the proportion-HbA1c association with geom_smooth(method = 'lm') for regression inspection)

## Examples

```
Est.prop.GSE50244 <- music_prop(bulk.eset = GSE50244.bulk.eset, sc.eset = EMTAB.eset, clusters = 'cellType', samples = 'sampleID', select.ct = c('alpha','beta','delta','gamma','acinar','ductal'))
```

## Evaluation signals

- Regression reproduces the reported p-value of 0.00126 for beta cell proportion vs HbA1c
- Slope corresponds to a 3.07% +/- 2.49% drop in beta cell proportion per 0.5% HbA1c increase
- All six estimated proportions per subject are non-negative and normalized over the included cell types
- Output covers all 89 bulk samples with rows containing missing HbA1c removed before regression
- Reference datasets load with expected dimensions (25453 genes x 1097 cells for EMTAB; 32581 features x 89 samples for bulk)

## Limitations

- Cell type proportions cannot be estimated with TPM as the input; RPKM is only usable if cell-type-specific total RNA abundance estimates are provided from another dataset
- Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate proportions of closely related or rare cell types
- Under assumptions A1-A3 only trends of cell type proportion change across subjects are recoverable, not absolute cell type proportions; when the efficiency assumption fails only the RNA-molecule fraction per cell type is estimable
- Most scRNA-seq references do not reflect true cell type proportions in intact tissues, though they provide cell type-specific gene expression

## Evidence

- [discussion] MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes.: "MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes."
- [methods] The cell type proportions is estimated by function music_prop: "The cell type proportions is estimated by function `music_prop`"
- [discussion] However, cell type proportions cannot be estimated with TPM as the input.: "However, cell type proportions cannot be estimated with TPM as the input."
- [discussion] MuSiC is available on Github (https://github.com/xuranw/MuSiC). Tutorial and examples are provided.: "MuSiC is available on Github (https://github.com/xuranw/MuSiC). Tutorial and examples are provided."
- [discussion] Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions: "Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions, especially for less frequent and rare cell types."
