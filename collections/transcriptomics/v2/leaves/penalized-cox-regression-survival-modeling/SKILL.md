---
name: penalized-cox-regression-survival-modeling
description: Use when predicting overall survival (time + censoring indicator) from
  thousands of genetic features per patient — e.g., Z-scored mRNA entries or mutated-gene
  matrices — and you need penalized Cox models with cross-validated prediction power.
when_to_use_negative:
- The outcome is binary rather than a survival time + censoring indicator — use the
  logistic regression variants (ridge/Lasso/adaptive Lasso logistic) instead
- The feature count approaches ~100,000 and no FDR screening is applied — the article
  states penalized regression models fall short at that magnitude and recommends including
  FDR first-step screening
- The data are low-dimensional (few candidate predictors) — an unpenalized standard
  Cox regression is the appropriate tool, not penalized variants with cross-validated
  tuning
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: HDMAC
  role: Web-based interactive R/Shiny platform implementing the Cox ridge, Lasso,
    and adaptive Lasso fits, FDR first-step screening, and cross-validated C-index
    estimation
  repo: https://github.com/chung-R/HD-MAC
- name: selectiveInference
  role: R package implementing the Lee et al. algorithm to compute correct post-selection
    p-values for variables retained by Lasso and adaptive Lasso Cox fits
- name: R
  role: Language in which HDMAC and all statistical analysis were written and performed;
    scripts run offline in RStudio
  repo: https://github.com/chung-R/HD-MAC
- name: Shiny
  role: R package used to expose the penalized Cox regression workflow as an interactive
    web app within HDMAC
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:173c6cf02a26b0b1bf252334407abb611f56ce89f5cacdd09ab77ca6fa865c08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - Shiny
  - selectiveInference
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# penalized-cox-regression-survival-modeling

## Summary

Fits Ridge, Lasso, and adaptive Lasso penalized Cox regression models to high-dimensional genomic covariates (e.g., TCGA ovarian HGSC mRNA or somatic mutation matrices) with an optional Benjamini-Hochberg FDR first-step screening and 5-fold cross-validated C-index estimation, via the HDMAC R/Shiny platform.

## When to use

Use when predicting overall survival (time + censoring indicator) from thousands of genetic features per patient — e.g., Z-scored mRNA entries or mutated-gene matrices — and you need penalized Cox models with cross-validated prediction power.

## When NOT to use

- The outcome is binary rather than a survival time + censoring indicator — use the logistic regression variants (ridge/Lasso/adaptive Lasso logistic) instead
- The feature count approaches ~100,000 and no FDR screening is applied — the article states penalized regression models fall short at that magnitude and recommends including FDR first-step screening
- The data are low-dimensional (few candidate predictors) — an unpenalized standard Cox regression is the appropriate tool, not penalized variants with cross-validated tuning

## Inputs

- TCGA patient x feature genomic matrix (e.g., 9,548 Z-scored mRNA expression entries for 316 ovarian HGSC patients, or a 670-gene somatic mutation matrix retained after a ≥1% mutation-frequency screen)
- Overall survival times and censoring indicators for the same patients
- Optional clinical covariates (e.g., gender, age)

## Outputs

- Summary table of C-index values per penalized Cox model (ridge/Lasso/adaptive Lasso) x screening condition (no-FDR vs. BH-FDR at 0.05), e.g., cindex_table.csv
- FDR-selected candidate gene list with hazard ratios (e.g., ZSWIM8 HR 7.493, PABPC3 HR 5.635)
- Post-selection p-values (variable, coefficient, p-value) for Lasso/adaptive Lasso selected variables via selectiveInference

## How to apply

Load a TCGA patient-by-feature matrix (e.g., 316 ovarian HGSC patients with 9,548 Z-scored mRNA entries, abnormal if Z > 2 or Z < −2 and abnormal in ≥2% of patients; or 670 genes mutated in ≥1% of patients) plus overall survival times and censoring indicators. Optionally apply Benjamini-Hochberg FDR control (threshold 0.05, via univariate Cox regression per gene/mRNA entry) as a first-step screening to address the multiple-comparison issue — recommended when variables approach the magnitude of a hundred thousand, where penalized regression falls short. Fit Cox ridge (negative log partial likelihood plus ridge penalty), Cox Lasso, and Cox adaptive Lasso (adaptive weights from the Lasso estimates) with tuning parameters chosen by cross-validation. Evaluate all model-screening combinations with 5-fold cross validation (the HDMAC default fold number is 5), computing the concordance index (C-index) for each. Optionally compute post-selection p-values for retained Lasso/adaptive Lasso variables using the Lee et al. selective inference framework via the selectiveInference R package.

## Related tools

- **HDMAC** (Web-based interactive R/Shiny platform implementing the Cox ridge, Lasso, and adaptive Lasso fits, FDR first-step screening, and cross-validated C-index estimation) — https://github.com/chung-R/HD-MAC
- **selectiveInference** (R package implementing the Lee et al. algorithm to compute correct post-selection p-values for variables retained by Lasso and adaptive Lasso Cox fits)
- **R** (Language in which HDMAC and all statistical analysis were written and performed; scripts run offline in RStudio) — https://github.com/chung-R/HD-MAC
- **Shiny** (R package used to expose the penalized Cox regression workflow as an interactive web app within HDMAC)

## Examples

```
git clone https://github.com/chung-R/HD-MAC && Rscript -e 'source("HD-MAC.R")'  # run HDMAC penalized Cox analysis offline in RStudio
```

## Evaluation signals

- The mRNA C-index table reproduces the reported values: no-FDR ridge/Lasso/adaptive Lasso 0.591/0.554/0.560 and FDR-screened 0.538/0.538/0.540
- The mutation-data C-index table reproduces Table 1 values (0.529/0.501/0.501 without FDR; 0.502/0.502/0.497 with FDR) and the FDR screen selects exactly ZSWIM8 and PABPC3 (HR 7.493 and 5.635)
- Cross validation uses 5 folds (the HDMAC default); validation is skipped only if fold number is set to 1
- Post-selection p-value output contains the fields variable, coefficient, and p-value, with all p-values within [0, 1]
- The summary table contains exactly 6 rows (3 penalized methods x 2 screening conditions) with the computed C-indices

## Limitations

- Penalized regression models fall short when variables are at the magnitude of a hundred thousand; inclusion of FDR first-step screening is recommended in that regime
- Genes identified are preliminary and provide rational directions for further study rather than validated biomarkers (e.g., ASAP3 for ovarian cancer survival)
- Selecting one fold number via cross validation is required; setting folds to 1 disables cross validation entirely
- Clinical variables such as gender and age can be included in the analysis but were not illustrated in the paper's results

## Evidence

- [abstract] Cross validation to estimate prediction power of each regression model: "Cross validation is provided as the method to estimate the prediction power of each regression model."
- [methods] FDR first-step screening: "the method to control the FDR is provided as an optional first-step screening method"
- [methods] 5-fold cross validation default: "The default fold number is 5, and cross validation method will not be performed if 1 is chosen"
- [methods] Platform implemented in R: "The platform was written and all statistical analysis was performed with the statistical computing and graphic drawing language, R"
- [methods] Z-score abnormality threshold: "A Z score above 2 or below −2 was considered abnormal"
- [discussion] Scale limitation of penalized regression: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
