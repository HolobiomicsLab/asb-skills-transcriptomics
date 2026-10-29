---
name: penalized-logistic-regression-variable-selection
description: Use when predicting a binary outcome (e.g., muscle-invasive vs non-muscle-invasive
  bladder cancer subtype) from thousands of genetic variables, and penalized regression
  with optional FDR screening is needed for variable selection and performance estimation.
when_to_use_negative:
- Outcome is a survival time with censoring indicator — use the HDMAC Cox penalized
  regression routines instead
- Variables are at the magnitude of a hundred thousand where penalized regression
  models fall short — include FDR screening first or the models cannot handle the
  dimensionality
- Only a handful of predictors are present; unpenalized standard logistic regression
  is more appropriate
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: HDMAC
  role: Web-based platform / R scripts implementing ridge, Lasso, and adaptive Lasso
    logistic regression with FDR first-step screening and cross validation
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Statistical computing environment in which the platform and all penalized
    regression analyses were written and performed
  repo: https://github.com/chung-R/HD-MAC
- name: selectiveInference
  role: R package implementing the Lee et al. algorithm to compute correct post-selection
    p-values for variables selected by Lasso and adaptive Lasso logistic regression
- name: Shiny
  role: R package enabling the interactive web-app interface of HDMAC
- name: RStudio
  role: Environment for running HDMAC R scripts offline
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:2f561091bf1d6c08b7cce4ee2e2e59c1ea514f8ccaef9b9a53ba222f4c949afd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - RStudio
  - Shiny
  - selectiveInference
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# penalized-logistic-regression-variable-selection

## Summary

Fit ridge, Lasso, and adaptive Lasso logistic regression to high-dimensional genomic covariates (e.g., TCGA mRNA entries) for binary outcomes, with optional Benjamini-Hochberg FDR first-step screening and cross-validated sensitivity/specificity/accuracy/AUC to estimate prediction power.

## When to use

Use when predicting a binary outcome (e.g., muscle-invasive vs non-muscle-invasive bladder cancer subtype) from thousands of genetic variables, and penalized regression with optional FDR screening is needed for variable selection and performance estimation.

## When NOT to use

- Outcome is a survival time with censoring indicator — use the HDMAC Cox penalized regression routines instead
- Variables are at the magnitude of a hundred thousand where penalized regression models fall short — include FDR screening first or the models cannot handle the dimensionality
- Only a handful of predictors are present; unpenalized standard logistic regression is more appropriate

## Inputs

- High-dimensional covariate matrix of genetic entries (e.g., TCGA mRNA expression with Z-score ±2 abnormality threshold, ≥2% of patients)
- Binary outcome labels per patient (e.g., invasion subtype for bladder cancer)
- Optional clinical covariates (e.g., gender, age)

## Outputs

- Fitted penalized logistic regression models (ridge, Lasso, adaptive Lasso) with selected variables
- Cross-validated metrics table with sensitivity, specificity, accuracy, and AUC per model (e.g., ridge AUC 68.107 without FDR, Lasso 65.864, adaptive Lasso 62.442)
- Post-selection p-values table (variable, coefficient, p-value) for Lasso/adaptive Lasso selected variables

## How to apply

Load the high-dimensional covariate matrix (e.g., 8,024 TCGA bladder cancer mRNA entries retained after Z-score ±2 abnormality screening in ≥2% of 189 patients) with binary outcome labels. Optionally apply first-step screening via Benjamini-Hochberg FDR on univariate logistic regression p-values per gene at the default threshold of 0.05 to address the multiple-testing problem. Fit ridge, Lasso, and adaptive Lasso logistic regression using the HDMAC R implementation, selecting tuning parameters by cross validation. Estimate each model's prediction power with 5-fold cross validation (the default fold number), computing sensitivity, specificity, accuracy, and AUC. Optionally compute correct post-selection p-values for Lasso/adaptive Lasso selected variables via the selectiveInference R package implementing the Lee et al. framework.

## Related tools

- **HDMAC** (Web-based platform / R scripts implementing ridge, Lasso, and adaptive Lasso logistic regression with FDR first-step screening and cross validation) — https://github.com/chung-R/HD-MAC
- **R** (Statistical computing environment in which the platform and all penalized regression analyses were written and performed) — https://github.com/chung-R/HD-MAC
- **selectiveInference** (R package implementing the Lee et al. algorithm to compute correct post-selection p-values for variables selected by Lasso and adaptive Lasso logistic regression)
- **Shiny** (R package enabling the interactive web-app interface of HDMAC)
- **RStudio** (Environment for running HDMAC R scripts offline)

## Examples

```
Rscript -e "source('HD-MAC.R'); hdmac_logistic(X, y, penalty='adaptive_lasso', screen='FDR', fdr=0.05, nfolds=5)"
```

## Evaluation signals

- 5-fold cross-validation AUC values fall in expected ranges (bladder cancer mRNA: ridge ≈68.1 without FDR / 66.5 with FDR; Lasso ≈65.9/67.0; adaptive Lasso ≈62.4/64.3)
- Cross-validated AUC, sensitivity, specificity, and accuracy are present for all six model × screening combinations (3 penalties × {FDR, no FDR})
- Post-selection p-value output exists with fields variable, coefficient, p-value, and all p-values lie in [0,1]
- Tuning parameters chosen by cross validation and FDR screening applied at the default threshold of 0.05
- Screening matrix dimension matches expectations (e.g., 8,024 mRNA entries × 189 patients after Z-score ±2 / ≥2% screening)

## Limitations

- Penalized regression models fall short at ~hundred thousand variables; FDR first-step screening is recommended at that scale
- Correct p-values for Lasso/adaptive Lasso selected variables are not usually available in common statistical software due to variable selection, requiring the selectiveInference package
- Genes identified are preliminary and require further in-depth study
- Cross validation will not be performed if fold number 1 is chosen (default fold number is 5)

## Evidence

- [abstract] Cross validation is provided as the method to estimate the prediction power of each regression model.: "Cross validation is provided as the method to estimate the prediction power of each regression model."
- [methods] Z-score threshold: "A Z score above 2 or below −2 was considered abnormal"
- [methods] selectiveInference package: "Here we used the ‘selectiveInference’ R package29,30 to implement the algorithm by Lee et al. to compute the correct p-values"
- [intro] First-step screening: [evidence span withheld — anchor status: not_found]
- [methods] Default fold number: "The default fold number is 5, and cross validation method will not be performed if 1 is chosen"
