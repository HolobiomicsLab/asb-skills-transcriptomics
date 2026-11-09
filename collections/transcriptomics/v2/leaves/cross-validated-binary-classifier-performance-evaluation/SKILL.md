---
name: cross-validated-binary-classifier-performance-evaluation
description: Use when a fitted penalized regression model on high-dimensional genomic
  covariates with a binary outcome (e.g., NMIBC vs MIBC invasion subtype) needs an
  unbiased estimate of prediction performance.
when_to_use_negative:
- The outcome is survival time (use the Cox ridge/Lasso/adaptive Lasso variants instead
  of logistic regression)
- Cross validation is not desired — in HDMAC, choosing fold number 1 disables cross
  validation entirely
- The variable count is at the magnitude of a hundred thousand, where penalized regression
  models fall short and FDR screening is recommended before model fitting
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: HDMAC
  role: Web-based interactive platform and R scripts that implement ridge/Lasso/adaptive
    Lasso logistic regression with cross validation to estimate prediction power
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Language in which HDMAC is written and all statistical analysis (including
    cross validation) is performed; also usable offline via RStudio
  repo: https://github.com/chung-R/HD-MAC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:3a4f9eec22c6be3080877ad57bcb70a63df07cfce2556602f37822c82c1bfea7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cross-validated-binary-classifier-performance-evaluation

## Summary

Estimates the prediction power of penalized logistic regression models (ridge, Lasso, adaptive Lasso) fitted to high-dimensional TCGA molecular data by k-fold cross validation, reporting sensitivity, specificity, accuracy, and AUC as in the HDMAC bladder cancer mRNA illustration.

## When to use

Use when a fitted penalized regression model on high-dimensional genomic covariates with a binary outcome (e.g., NMIBC vs MIBC invasion subtype) needs an unbiased estimate of prediction performance.

## When NOT to use

- The outcome is survival time (use the Cox ridge/Lasso/adaptive Lasso variants instead of logistic regression)
- Cross validation is not desired — in HDMAC, choosing fold number 1 disables cross validation entirely
- The variable count is at the magnitude of a hundred thousand, where penalized regression models fall short and FDR screening is recommended before model fitting

## Inputs

- High-dimensional molecular feature matrix (e.g., TCGA bladder cancer mRNA expression, 8,024 entries × 189 patients) with binary outcome labels (invasion subtype)
- Optional clinical covariates (e.g., gender, age)
- Fitted penalized logistic regression model(s) (ridge, Lasso, adaptive Lasso) with cross-validation-selected tuning parameters

## Outputs

- Per-model cross-validated performance table with sensitivity, specificity, accuracy, and AUC (e.g., auc_table.csv with rows for each penalty × FDR-screening combination)

## How to apply

Fit the penalized logistic regression models (ridge, Lasso, adaptive Lasso) on the TCGA mRNA expression matrix — e.g., the 8,024 mRNA entries retained after Z-score ±2 and ≥2%-of-patients abnormality screening for the 189 bladder cancer patients — selecting tuning parameters within the HDMAC R implementation. Optionally apply Benjamini-Hochberg FDR first-step screening (default threshold 0.05 on univariate logistic regression p-values) and refit. Run 5-fold cross validation (the HDMAC default fold number; if 1 is chosen, cross validation is skipped) for each fitted model, computing sensitivity, specificity, accuracy, and AUC per fold. Assemble the per-model metric table (in the illustration: AUCs of ridge 68.107/66.515, Lasso 65.864/67.020, adaptive Lasso 62.442/64.300, without/with FDR screening) and verify the values are consistent across cross-validation runs before exporting, e.g., as auc_table.csv.

## Related tools

- **HDMAC** (Web-based interactive platform and R scripts that implement ridge/Lasso/adaptive Lasso logistic regression with cross validation to estimate prediction power) — https://github.com/chung-R/HD-MAC
- **R** (Language in which HDMAC is written and all statistical analysis (including cross validation) is performed; also usable offline via RStudio) — https://github.com/chung-R/HD-MAC

## Examples

```
Rscript HD-MAC.R --data tcga_blca_mrna.csv --outcome invasion_subtype --penalty ridge --folds 5 --cv
```

## Evaluation signals

- The reported AUC values match the reference bladder cancer mRNA table: ridge 68.107 without FDR / 66.515 with FDR, Lasso 65.864/67.020, adaptive Lasso 62.442/64.300
- The metrics table contains exactly six rows when both penalties-with-and-without-FDR-screening combinations of the three penalized models are evaluated
- Cross validation uses the default 5-fold scheme; using fold number 1 should yield no cross-validated performance estimate
- Each row reports all four metrics: sensitivity, specificity, accuracy, and AUC
- AUC values are in a plausible range (0–100 scale as reported) and re-running the pipeline reproduces consistent metrics given fixed random folds

## Limitations

- Penalized regression models fall short when variables number around a hundred thousand; inclusion of FDR screening is recommended in that regime
- Cross-validated performance depends on the tuning-parameter selection and fold scheme; HDMAC's default is 5 folds, and results vary if fold number is changed
- Estimates are for fitted models on high-dimensional genomic covariates; clinical covariates (gender, age) can be included in the analysis but were not illustrated in the reported results

## Evidence

- [abstract] Cross validation is provided as the method to estimate the prediction power of each regression model.: "Cross validation is provided as the method to estimate the prediction power of each regression model."
- [methods] Default 5-fold cross validation: "The default fold number is 5, and cross validation method will not be performed if 1 is chosen"
- [methods] FDR as optional first-step screening: "the method to control the FDR is provided as an optional first-step screening method"
- [methods] Z-score threshold for abnormal expression: "A Z score above 2 or below −2 was considered abnormal"
- [discussion] Limitation: penalized regression at ~100,000 variables: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
