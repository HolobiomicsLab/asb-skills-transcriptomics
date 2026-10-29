---
name: linear-regression-of-cell-proportions-on-clinical-trait
description: Use when estimated cell type proportions from deconvolution (e.g., MuSiC
  music_prop output) must be tested for association with a clinical trait such as
  HbA1c via single-variable linear regression.
when_to_use_negative:
- Cell type proportions have not been estimated yet — run MuSiC music_prop deconvolution
  first; this skill operates on the proportion output, not raw counts
- TPM-normalized expression is the deconvolution input — cell type proportions cannot
  be estimated with TPM, so downstream regression would be on invalid proportions
- Multi-variable adjustment is required rather than the single-variable model (lm
  with one predictor, as drawn by geom_smooth(method = 'lm')) specified by the reference
  workflow
edam_operation: http://edamontology.org/operation_0370
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MuSiC
  role: music_prop provides the weighted cell type proportion estimates (Est.prop.weighted)
    that serve as the regression response variable
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: lm() fits the single-variable linear model of beta cell proportion on HbA1c;
    results written to beta_hba1c_regression_results.csv
- name: ggplot2
  role: geom_smooth(method = 'lm') draws the regression fit used for validation against
    the reported association
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:b4698d083be638cb12ffbabec717e3d224d74dac8dd3b6be9e52765c6558ccb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - MuSiC
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_0370
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# linear-regression-of-cell-proportions-on-clinical-trait

## Summary

Fit a single-variable linear model (R lm, visualized with ggplot2 geom_smooth(method='lm')) of MuSiC-estimated beta cell proportion on HbA1c in the 89-donor Fadista et al. bulk pancreatic islet dataset, to test whether deconvolved cell composition tracks a clinical trait.

## When to use

Use when estimated cell type proportions from deconvolution (e.g., MuSiC music_prop output) must be tested for association with a clinical trait such as HbA1c via single-variable linear regression.

## When NOT to use

- Cell type proportions have not been estimated yet — run MuSiC music_prop deconvolution first; this skill operates on the proportion output, not raw counts
- TPM-normalized expression is the deconvolution input — cell type proportions cannot be estimated with TPM, so downstream regression would be on invalid proportions
- Multi-variable adjustment is required rather than the single-variable model (lm with one predictor, as drawn by geom_smooth(method = 'lm')) specified by the reference workflow

## Inputs

- Est.prop.weighted proportion table from MuSiC music_prop (89 subjects x 6 islet cell types)
- Subject phenotype table with age, bmi, hba1c, and gender columns
- MuSiC R package installed via devtools::install_github('xuranw/MuSiC')

## Outputs

- beta_hba1c_regression_results.csv containing regression slope, standard error, and p-value
- Per-subject fitted regression of beta cell proportion on HbA1c (e.g., ggplot2 scatter with geom_smooth(method = 'lm'))

## How to apply

Run MuSiC deconvolution with music_prop(bulk.eset = GSE50244.bulk.eset, sc.eset = EMTAB.eset, clusters = 'cellType', samples = 'sampleID', select.ct = c('alpha','beta','delta','gamma','acinar','ductal')) to obtain weighted proportions over the six major islet cell types (>90% of the islet). Assemble a data frame joining the estimated beta cell proportion (second column of Est.prop.weighted) with subject phenotypes age, bmi, hba1c, and gender, and remove rows with missing HbA1c. Fit the single-variable linear model lm(beta_proportion ~ hba1c) as drawn by geom_smooth(method = 'lm'), then extract the slope, its standard error, and p-value into beta_hba1c_regression_results.csv.

## Related tools

- **MuSiC** (music_prop provides the weighted cell type proportion estimates (Est.prop.weighted) that serve as the regression response variable) — https://github.com/xuranw/MuSiC
- **R** (lm() fits the single-variable linear model of beta cell proportion on HbA1c; results written to beta_hba1c_regression_results.csv)
- **ggplot2** (geom_smooth(method = 'lm') draws the regression fit used for validation against the reported association)

## Examples

```
Est.prop.weighted <- music_prop(bulk.eset = GSE50244.bulk.eset, sc.eset = EMTAB.eset, clusters = 'cellType', samples = 'sampleID', select.ct = c('alpha','beta','delta','gamma','acinar','ductal')); summary(lm(beta_prop ~ hba1c, data = props))
```

## Evaluation signals

- The fitted lm must reproduce the reported p-value of 0.00126 for beta cell proportion vs HbA1c
- The regression must be consistent with an adjusted estimate of a 3.07% ± 2.49% beta cell proportion drop per 0.5% HbA1c increase
- The six estimated proportions for each of the 89 subjects must be non-negative and normalized over the included cell types
- Rows with missing HbA1c must be excluded before fitting, and slope, standard error, and p-value must all be present in beta_hba1c_regression_results.csv

## Limitations

- Under assumptions A1–A3, only trends of cell type proportion change across subjects can be recovered, not absolute cell type proportions, so regression tests association rather than true composition
- Collinearity among closely related cell types can make proportion estimates unreliable, especially for rare cell types, weakening the downstream association
- MuSiC requires count-level input: cell type proportions cannot be estimated with TPM as the input

## Evidence

- [methods] deconvolution output feeding the regression: "The cell type proportions is estimated by function `music_prop`"
- [methods] regression model specification: [evidence span withheld — anchor status: not_found]
- [results] expected association finding: [evidence span withheld — anchor status: not_found]
- [abstract] proportion estimation constraint: "By appropriate weighting of genes showing cross-subject and cross-cell consistency, MuSiC enables the transfer of cell type-specific gene expression information from one dataset to another."
- [readme] tool installation: "devtools::install_github('xuranw/MuSiC')"
