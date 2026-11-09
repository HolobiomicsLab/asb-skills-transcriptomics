---
name: cross-validated-prediction-evaluation
description: Use when a multivariate logistic or Cox model has been built from genes
  selected by first-step screening (e.g., adaptive Lasso with FDR) and you need an
  unbiased estimate of prediction power via cross validation.
when_to_use_negative:
- Only a single candidate variable or a low-dimensional model is being evaluated —
  cross-validated high-dimensional penalized regression is unnecessary
- The user is still at the first-step screening stage (FDR/mutation/expression frequency
  filtering) and has not yet fitted a multivariate regression model to screen output
- Fold number 1 is chosen — the article states cross validation will not be performed,
  so no prediction-power estimate is produced
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: HDMAC
  role: Web-based interactive platform that performs Ridge, Lasso, and adaptive Lasso
    penalized logistic/Cox regression with built-in 5-fold cross validation to estimate
    each model's prediction power
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Language in which HDMAC and all statistical analyses were implemented; R scripts
    on GitHub enable offline use with RStudio and downloading TCGA molecular variables
  repo: https://github.com/chung-R/HD-MAC
- name: selectiveInference
  role: R package used to compute correct p-values after Lasso/adaptive Lasso model
    selection alongside the cross-validated performance estimates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:b9ff49f89fa38c4362534f4871bba8f125b368c2a1f14251896a2f3621a5d5ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - selectiveInference
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-validated-prediction-evaluation

## Summary

Estimates the predictive performance (e.g., cross-validated AUC, sensitivity, specificity, accuracy) of a multivariate regression model built from high-dimensionally screened genes using HDMAC's built-in cross validation, so that penalized-regression-selected models can be assessed rather than just reported.

## When to use

Use when a multivariate logistic or Cox model has been built from genes selected by first-step screening (e.g., adaptive Lasso with FDR) and you need an unbiased estimate of prediction power via cross validation.

## When NOT to use

- Only a single candidate variable or a low-dimensional model is being evaluated — cross-validated high-dimensional penalized regression is unnecessary
- The user is still at the first-step screening stage (FDR/mutation/expression frequency filtering) and has not yet fitted a multivariate regression model to screen output
- Fold number 1 is chosen — the article states cross validation will not be performed, so no prediction-power estimate is produced

## Inputs

- TCGA bladder cancer mRNA expression matrix (189 patients, 8,024 pre-screened expressed entries)
- Coefficient list of the 24 abnormally expressed genes selected by adaptive Lasso logistic regression after FDR penalty (SPTSSA through ZSCAN2, Table 4)
- Patient outcome labels (binary invasion status or survival) joined to the expression matrix

## Outputs

- Cross-validated prediction-power estimates (AUC) for the regression model
- Sensitivity, specificity, and accuracy from 5-fold cross validation
- Predicted-label table classifying each patient as invasive (score > 0.34) or non-invasive
- Correct p-values for Lasso/adaptive Lasso selected coefficients

## How to apply

Build the multivariate log-odds score for each patient as the coefficient-weighted linear combination of the selected genes' abnormal-expression Z-scores (a Z score above 2 or below −2 was considered abnormal), then classify samples against a decision threshold (e.g., 0.34 for bladder cancer invasion). Run HDMAC's cross validation — the default fold number is 5, so use 5-fold cross validation unless a specific fold count is justified — to estimate the prediction power of the regression model; note cross validation is skipped entirely if fold number 1 is chosen. For binary outcomes use the adaptive Lasso logistic regression branch and collect the cross-validated AUC along with sensitivity, specificity, and accuracy; HDMAC also supplies correct post-selection p-values for Lasso and adaptive Lasso via the selectiveInference R package. Validate by checking that the selected gene list and cross-validated AUC reproduce the article's reported values (e.g., adaptive-Lasso-with-FDR AUC of 64.300 in Table 3).

## Related tools

- **HDMAC** (Web-based interactive platform that performs Ridge, Lasso, and adaptive Lasso penalized logistic/Cox regression with built-in 5-fold cross validation to estimate each model's prediction power) — https://github.com/chung-R/HD-MAC
- **R** (Language in which HDMAC and all statistical analyses were implemented; R scripts on GitHub enable offline use with RStudio and downloading TCGA molecular variables) — https://github.com/chung-R/HD-MAC
- **selectiveInference** (R package used to compute correct p-values after Lasso/adaptive Lasso model selection alongside the cross-validated performance estimates)

## Evaluation signals

- Cross-validated AUC matches the adaptive-Lasso-with-FDR value of 64.300 reported in Table 3
- Predicted-label table reproduces the reported Table 4 gene list (24 genes, SPTSSA through ZSCAN2)
- 5-fold cross validation is used (HDMAC default fold number), not leave-one-out or no validation
- Abnormal-expression scoring uses the Z > 2 or Z < −2 threshold before classification at the 0.34 cutoff
- Both sensitivity and specificity are reported alongside the AUC, not AUC alone

## Limitations

- Cross validation will not be performed if fold number 1 is chosen
- Penalized regression models fall short at ~hundred thousand variables; FDR first-step screening is recommended at that scale
- Correct p-values for Lasso/adaptive Lasso are not usually available in common statistical software outside HDMAC's selectiveInference-based approach
- Genes identified are preliminary and require further in-depth study

## Evidence

- [abstract] Cross validation is provided as the method to estimate the prediction power of each regression model.: "Cross validation is provided as the method to estimate the prediction power of each regression model."
- [methods] The default fold number is 5, and cross validation method will not be performed if 1 is chosen: "The default fold number is 5, and cross validation method will not be performed if 1 is chosen"
- [methods] A Z score above 2 or below −2 was considered abnormal: "A Z score above 2 or below −2 was considered abnormal"
- [intro] it would estimate the prediction power for each regression model using cross validation with the correct p-values for the Lasso and adaptive Lasso provided: "it would estimate the prediction power for each regression model using cross validation with the correct p-values for the Lasso and adaptive Lasso provided"
- [methods] The platform was written and all statistical analysis was performed with the statistical computing and graphic drawing language, R: "The platform was written and all statistical analysis was performed with the statistical computing and graphic drawing language, R"
