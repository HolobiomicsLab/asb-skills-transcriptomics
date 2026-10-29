---
name: pancreatic-islet-cell-type-biology
description: Use when deconvolving bulk pancreatic islet RNA-seq (e.g., Fadista/GSE50244
  read counts) into alpha, beta, delta, gamma, acinar and ductal proportions with
  a healthy-human-islet scRNA-seq reference (E-MTAB-5060) and testing proportion–phenotype
  associations such as beta cell proportion vs HbA1c.
when_to_use_negative:
- When the bulk or single-cell input contains TPM-normalized values — cell type proportions
  cannot be estimated with TPM as the input; use raw read counts.
- When bulk samples come from conditions very different from the scRNA-seq reference
  (e.g., multi-condition bulk data) — MuSiC2 is the intended iterative alternative
  for such cases.
- When the goal is absolute cell type proportions rather than trends across subjects
  — under the method's assumptions only trends of cell type proportion change across
  subjects can be recovered.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MuSiC
  role: Performs the weighted non-negative least squares (W-NNLS) deconvolution of
    bulk islet read counts via music_prop using the scRNA-seq reference
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: Language in which MuSiC is implemented as an R package installed from GitHub
    via devtools; also used for the lm regression of beta proportion on HbA1c
- name: ggplot2
  role: Visualizes beta cell proportion vs HbA1c with geom_smooth(method = 'lm') as
    drawn in the workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:7da58e8ba9b5becb2c232f89a43d6b9b6639bce2fccd834dcaef5905043b1b8e
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pancreatic-islet-cell-type-biology

## Summary

Reproduces the MuSiC deconvolution analysis in which bulk pancreatic islet RNA-seq from 89 Fadista et al. donors (GSE50244) is deconvolved using the Segerstolpe et al. scRNA-seq reference (E-MTAB-5060) and the estimated beta cell proportion is regressed on HbA1c, recovering the reported p-value of 0.00126.

## When to use

Use when deconvolving bulk pancreatic islet RNA-seq (e.g., Fadista/GSE50244 read counts) into alpha, beta, delta, gamma, acinar and ductal proportions with a healthy-human-islet scRNA-seq reference (E-MTAB-5060) and testing proportion–phenotype associations such as beta cell proportion vs HbA1c.

## When NOT to use

- When the bulk or single-cell input contains TPM-normalized values — cell type proportions cannot be estimated with TPM as the input; use raw read counts.
- When bulk samples come from conditions very different from the scRNA-seq reference (e.g., multi-condition bulk data) — MuSiC2 is the intended iterative alternative for such cases.
- When the goal is absolute cell type proportions rather than trends across subjects — under the method's assumptions only trends of cell type proportion change across subjects can be recovered.

## Inputs

- GSE50244bulkeset.rds — bulk pancreatic islet read counts as ExpressionSet (32581 features, 89 Fadista donors)
- EMTABesethealthy.rds — Segerstolpe et al. healthy human islet scRNA-seq reference (E-MTAB-5060; 25453 genes, 1097 cells) as ExpressionSet
- Subject phenotype table with age, bmi, hba1c, gender for the 89 donors

## Outputs

- music_prop output: Est.prop.weighted matrix of six non-negative, normalized cell type proportions per subject
- beta_hba1c_regression_results.csv with regression slope, standard error, and p-value
- ggplot2 scatter of beta cell proportion vs HbA1c with geom_smooth(method = 'lm')

## How to apply

Load the bulk read counts GSE50244bulkeset.rds (32581 features, 89 samples) and the single-cell reference EMTABesethealthy.rds (25453 genes, 1097 cells) as Biobase ExpressionSet objects. Run MuSiC deconvolution with music_prop using clusters='cellType', samples='sampleID', and select.ct = c('alpha','beta','delta','gamma','acinar','ductal') to get weighted proportions constrained to the six major islet cell types that make up over 90% of the islet. MuSiC uses weighted non-negative least squares (W-NNLS) with gene weights initialized by NNLS and estimated iteratively, so no pre-selected marker genes are needed. Join the estimated beta cell proportion (second column of Est.prop.weighted) with subject phenotypes age, bmi, hba1c, and gender, drop rows with missing HbA1c, and fit a single-variable linear model (lm) of beta cell proportion on HbA1c. Report the slope, standard error, and p-value, expecting an adjusted ~3.07% ± 2.49% drop in beta cell proportion per 0.5% HbA1c increase.

## Related tools

- **MuSiC** (Performs the weighted non-negative least squares (W-NNLS) deconvolution of bulk islet read counts via music_prop using the scRNA-seq reference) — https://github.com/xuranw/MuSiC
- **R** (Language in which MuSiC is implemented as an R package installed from GitHub via devtools; also used for the lm regression of beta proportion on HbA1c)
- **ggplot2** (Visualizes beta cell proportion vs HbA1c with geom_smooth(method = 'lm') as drawn in the workflow)

## Examples

```
music_prop(bulk.eset = GSE50244.bulk.eset, sc.eset = EMTAB.eset, clusters = 'cellType', samples = 'sampleID', select.ct = c('alpha','beta','delta','gamma','acinar','ductal'))
```

## Evaluation signals

- The fitted linear model of beta cell proportion on HbA1c must reproduce the reported p-value of 0.00126.
- The adjusted slope must match a ~3.07% ± 2.49% beta cell proportion drop per 0.5% HbA1c increase.
- All six estimated proportions for each of the 89 subjects must be non-negative and normalized over the included cell types.
- The proportions come from Est.prop.weighted (MuSiC's weighted W-NNLS estimates), not the unweighted NNLS columns.
- Rows with missing HbA1c must be removed before fitting; the regression must be single-variable (beta proportion ~ hba1c).

## Limitations

- Cell type proportions cannot be estimated with TPM as the input; raw read counts are required (RPKM is usable only if cell type-specific total RNA abundance estimates are provided).
- Under assumptions A1–A3 only trends of cell type proportion change across subjects can be recovered, not absolute cell type proportions.
- Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions, especially for less frequent and rare cell types.
- Most scRNA-seq data do not reflect true cell type proportions in intact tissues; they provide cell type-specific gene expression information only.

## Evidence

- [results] MuSiC deconvolution of the Fadista et al. bulk islet samples detects a significant association of beta cell proportion with HbA1c level, with a reported p-value of 0.00126.: [evidence span withheld — anchor status: not_found]
- [methods] music_prop call with clusters, samples, and cell type selection: "The cell type proportions is estimated by function `music_prop`"
- [discussion] MuSiC is a weighted non-negative least squares method without pre-selected marker genes: "MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes."
- [readme] MuSiC description in the README: [evidence span withheld — anchor status: external_doc]
- [abstract] MuSiC abstract claim: "We present MuSiC, a method that utilizes cell-type specific gene expression from single-cell RNA sequencing (RNA-seq) data to characterize cell type compositions from bulk RNA-seq data"
