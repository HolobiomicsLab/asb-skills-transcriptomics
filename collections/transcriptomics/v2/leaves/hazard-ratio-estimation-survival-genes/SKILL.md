---
name: hazard-ratio-estimation-survival-genes
description: Use when high-dimensional genomic covariates (e.g., TCGA somatic mutation
  matrix) must be related to a survival outcome, and penalized Cox selection with
  FDR screening, hazard ratios, and C-index evaluation is needed.
when_to_use_negative:
- The outcome is binary rather than a survival endpoint — use HDMAC's Ridge/Lasso/adaptive
  Lasso logistic regression path instead of Cox regression
- The covariate matrix already contains only a handful of pre-screened genes, making
  penalized regression and FDR first-step screening unnecessary
- The data contain variables at the magnitude of a hundred thousand, where the article
  notes penalized regression models fall short and FDR screening is explicitly recommended
  first
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3316
tools:
- name: HDMAC
  role: Web-based/R platform that implements Ridge, Lasso, and adaptive Lasso Cox
    regression with optional FDR first-step screening and cross-validated prediction
    power
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Language in which the platform and all penalized Cox statistical analysis
    were written and performed
  repo: https://github.com/chung-R/HD-MAC
- name: Shiny
  role: R package used to build the interactive HDMAC web app exposing the penalized
    Cox workflow
  repo: https://github.com/chung-R/HD-MAC
- name: selectiveInference
  role: R package used to compute correct post-selection p-values for Lasso and adaptive
    Lasso selected variables
- name: RStudio
  role: Offline environment in which users can run the HDMAC R scripts instead of
    the web app
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:f077d2e94c9600d9024e0f80833502e29df8651a01dd328872d1b780c8fe5d24
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
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Hazard ratio estimation for survival-associated genes

## Summary

Fits penalized Cox regression models (Ridge, Lasso, adaptive Lasso) to high-dimensional somatic mutation data with overall survival outcomes, optionally preceded by Benjamini-Hochberg FDR first-step screening, and reports selected genes with hazard ratios and cross-validated C-index prediction power.

## When to use

Use when high-dimensional genomic covariates (e.g., TCGA somatic mutation matrix) must be related to a survival outcome, and penalized Cox selection with FDR screening, hazard ratios, and C-index evaluation is needed.

## When NOT to use

- The outcome is binary rather than a survival endpoint — use HDMAC's Ridge/Lasso/adaptive Lasso logistic regression path instead of Cox regression
- The covariate matrix already contains only a handful of pre-screened genes, making penalized regression and FDR first-step screening unnecessary
- The data contain variables at the magnitude of a hundred thousand, where the article notes penalized regression models fall short and FDR screening is explicitly recommended first

## Inputs

- High-dimensional somatic mutation matrix (genes x patients; e.g., 670 mutated genes after ≥1% frequency screening from 316 TCGA ovarian HGSC patients)
- Overall survival times and censoring indicators for the same patients
- Optional clinical covariates (e.g., gender, age) that may be included in the regression
- FDR threshold for first-step screening (default 0.05)

## Outputs

- Summary table (methods x with/without FDR screening) of selected gene counts and cross-validated C-index values (e.g., ovarian_mutation_cox_cindex_table.csv)
- List of FDR-screened candidate survival-associated genes with hazard ratios (e.g., ZSWIM8 HR 7.493, PABPC3 HR 5.635)
- Correct post-selection p-values for Lasso and adaptive Lasso selected variables

## How to apply

Load the mutation matrix (genes retained after ≥1% mutation-frequency screening) together with patients' survival times and censoring indicators. Optionally apply Benjamini-Hochberg FDR first-step screening (default threshold 0.05) to reduce the multiple-testing burden before penalized fitting. Fit Ridge, Lasso, and adaptive Lasso Cox regressions by minimizing negative log partial likelihood plus the penalty, with adaptive weights taken from the Lasso estimates. Evaluate each fit with 5-fold cross validation (the HDMAC default fold number) using the concordance index (C-index) as prediction power, and report selected genes with their hazard ratios. Validate by checking that the FDR-screened fits select a small, stable gene set (e.g., ZSWIM8 HR 7.493 and PABPC3 HR 5.635 in the TCGA ovarian HGSC example) and that C-indices match expected values (0.529/0.501/0.501 without FDR; 0.502/0.502/0.497 with FDR).

## Related tools

- **HDMAC** (Web-based/R platform that implements Ridge, Lasso, and adaptive Lasso Cox regression with optional FDR first-step screening and cross-validated prediction power) — https://github.com/chung-R/HD-MAC
- **R** (Language in which the platform and all penalized Cox statistical analysis were written and performed) — https://github.com/chung-R/HD-MAC
- **Shiny** (R package used to build the interactive HDMAC web app exposing the penalized Cox workflow) — https://github.com/chung-R/HD-MAC
- **selectiveInference** (R package used to compute correct post-selection p-values for Lasso and adaptive Lasso selected variables)
- **RStudio** (Offline environment in which users can run the HDMAC R scripts instead of the web app)

## Examples

```
Rscript -e 'source("HDMAC.R"); HDMAC::run_app()'  # or run offline in RStudio via https://github.com/chung-R/HD-MAC
```

## Evaluation signals

- The FDR-screened gene list contains ZSWIM8 (HR 7.493) and PABPC3 (HR 5.635) for all three penalized methods in the TCGA ovarian HGSC example
- Reported 5-fold cross-validated C-indices match Table 1: 0.529 (Ridge), 0.501 (Lasso), 0.501 (adaptive Lasso) without FDR; 0.502/0.502/0.497 with FDR screening
- The emitted summary table has 6 rows (3 methods x no-FDR/FDR) with gene counts and C-indices per row
- Input mutation matrix is restricted to genes mutated in ≥1% of patients (670 genes for the 316-patient ovarian cohort)
- Correct post-selection p-values are reported for Lasso and adaptive Lasso selections via the selectiveInference algorithm

## Limitations

- Penalized regression models fall short when variables number around a hundred thousand; FDR first-step screening is recommended in that regime
- Selected genes are preliminary and require further in-depth study; a genome-wide search for genetic changes associated with ovarian cancer survival is still lacking
- Clinical covariates such as gender and age may be included but were not illustrated in the article's results

## Evidence

- [methods] the method to control the FDR is provided as an optional first-step screening method: "the method to control the FDR is provided as an optional first-step screening method"
- [methods] The default fold number is 5, and cross validation method will not be performed if 1 is chosen: "The default fold number is 5, and cross validation method will not be performed if 1 is chosen"
- [methods] a preliminary screening was performed so that only the genes whose mutations were found in 1% or more: "a preliminary screening was performed so that only the genes whose mutations were found in 1% or more"
- [abstract] Cross validation is provided as the method to estimate the prediction power of each regression model.: "Cross validation is provided as the method to estimate the prediction power of each regression model."
- [methods] with the help of Shiny, an R package that facilitates the building of interactive web Apps straight from R: "with the help of Shiny, an R package that facilitates the building of interactive web Apps straight from R"
