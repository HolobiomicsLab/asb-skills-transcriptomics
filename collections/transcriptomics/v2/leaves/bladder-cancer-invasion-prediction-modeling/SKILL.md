---
name: bladder-cancer-invasion-prediction-modeling
description: Use when analyzing high-dimensional TCGA mRNA expression data with a
  binary outcome (e.g., invasion subtype) and you need candidate genes selected by
  penalized regression with correct post-selection p-values.
when_to_use_negative:
- The outcome is a survival time rather than a binary subtype — HDMAC's Cox variants,
  not logistic adaptive Lasso, apply there
- Data dimension is modest and standard (unpenalized) logistic regression is valid
  — selective p-value machinery and FDR pre-screening are unnecessary
- Variables number in the hundred thousands and penalized regression alone is attempted
  without FDR screening — the article recommends FDR in that regime because penalized
  models fall short
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0622
tools:
- name: HDMAC
  role: Web-based/R platform implementing first-step FDR screening, adaptive Lasso
    logistic regression with cross-validation, and post-selection p-value computation
  repo: https://github.com/chung-R/HD-MAC
- name: selectiveInference
  role: R package used to compute correct p-values after Lasso/adaptive Lasso model
    selection via Lee et al.'s algorithm
- name: R
  role: Statistical computing environment in which the platform, data download from
    TCGA, and all analyses were written and performed
- name: Shiny
  role: R package enabling the interactive web-app front end of HDMAC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:8d81ee8820dfdfe7296d0612f60a9651b8aab038dfe20fc2e99a40131792bb95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - Shiny
  - selectiveInference
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# bladder-cancer-invasion-prediction-modeling

## Summary

A penalized-regression workflow for identifying mRNA entries associated with bladder cancer invasion subtype, combining first-step Benjamini-Hochberg FDR screening with adaptive Lasso logistic regression and post-selection inference via HDMAC/selectiveInference.

## When to use

Use when analyzing high-dimensional TCGA mRNA expression data with a binary outcome (e.g., invasion subtype) and you need candidate genes selected by penalized regression with correct post-selection p-values.

## When NOT to use

- The outcome is a survival time rather than a binary subtype — HDMAC's Cox variants, not logistic adaptive Lasso, apply there
- Data dimension is modest and standard (unpenalized) logistic regression is valid — selective p-value machinery and FDR pre-screening are unnecessary
- Variables number in the hundred thousands and penalized regression alone is attempted without FDR screening — the article recommends FDR in that regime because penalized models fall short

## Inputs

- TCGA bladder cancer Z-score mRNA expression matrix for 189 patients (~8,024 entries after filtering)
- Binary invasion subtype outcome per patient (e.g., NMIBC vs MIBC)
- HDMAC R scripts from GitHub (https://github.com/chung-R/HD-MAC)

## Outputs

- Table of 24 selected mRNA entries with estimated coefficients, odds ratios, and p-values (reproducing Table 4, e.g., PSORS1C2 coefficient 1.14, ICOSLG p = 0.004)
- Cross-validated prediction power estimate for the fitted adaptive Lasso model

## How to apply

Load TCGA bladder cancer Z-score mRNA expression for 189 patients and the binary invasion subtype outcome; mark an entry abnormal if Z > 2 or Z < -2 and retain entries abnormal in >= 2% of patients (~8,024 entries). Because penalized regression falls short at ~hundred thousand variables, run first-step screening: fit a univariate logistic regression per gene and control FDR with Benjamini-Hochberg at 0.05. Fit the adaptive Lasso logistic regression (penalty weights w_j = 1/|beta_j^lasso|) on screened entries, selecting the tuning parameter by (default 5-fold) cross-validation. Compute correct post-selection p-values with the selectiveInference R package implementing Lee et al.'s algorithm, then transform coefficients to odds ratios and assemble a table of the 24 selected mRNA entries.

## Related tools

- **HDMAC** (Web-based/R platform implementing first-step FDR screening, adaptive Lasso logistic regression with cross-validation, and post-selection p-value computation) — https://github.com/chung-R/HD-MAC
- **selectiveInference** (R package used to compute correct p-values after Lasso/adaptive Lasso model selection via Lee et al.'s algorithm)
- **R** (Statistical computing environment in which the platform, data download from TCGA, and all analyses were written and performed)
- **Shiny** (R package enabling the interactive web-app front end of HDMAC)

## Examples

```
Rscript -e "source('HDMAC.R'); hdmac(y=invasion_subtype, x=mrna_zscores, family='binomial', penalty='adaptive lasso', screen='FDR', alpha=0.05)"
```

## Evaluation signals

- Output table contains exactly 24 gene rows whose entries and values reproduce Table 4 of the paper
- Spot-check known values: PSORS1C2 estimated coefficient = 1.14 and ICOSLG p-value = 0.004
- Each selected-gene row includes estimated coefficient, odds ratio, and p-value fields
- Filtered input retains only mRNA entries with Z > 2 or Z < -2 in >= 2% of patients (~8,024 entries) before screening
- Reported p-values come from selectiveInference (Lee et al. post-selection inference), not naive coefficient tests

## Limitations

- Selected genes are preliminary candidates that require further in-depth study
- Penalized regression models fall short at ~hundred thousand variables; FDR first-step screening is recommended in that regime
- Correct post-selection p-values for Lasso/adaptive Lasso are not usually available in common statistical software, so selectiveInference is required

## Evidence

- [methods] A Z score above 2 or below −2 was considered abnormal: "A Z score above 2 or below −2 was considered abnormal"
- [methods] preliminary screening 2% threshold: "a preliminary screening was performed so that only the genes whose mutations were found in 1% or more and the mRNA entries whose abnormal expression was found in 2% or more of the patients"
- [methods] FDR first-step screening: "the method to control the FDR is provided as an optional first-step screening method"
- [methods] selectiveInference p-values: "Here we used the ‘selectiveInference’ R package29,30 to implement the algorithm by Lee et al. to compute the correct p-values"
- [discussion] penalized models fall short at large scale: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
- [methods] HDMAC written in R with Shiny: "with the help of Shiny, an R package that facilitates the building of interactive web Apps straight from R"
