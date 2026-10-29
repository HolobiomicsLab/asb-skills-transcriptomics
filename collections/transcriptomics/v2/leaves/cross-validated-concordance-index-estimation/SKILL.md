---
name: cross-validated-concordance-index-estimation
description: Use when evaluating the predictive performance of penalized Cox regression
  (Ridge, Lasso, adaptive Lasso) fitted to high-dimensional genomic covariates (e.g.,
  9,548 mRNA entries or 670 mutated genes vs. 316 patients' survival times) with a
  5-fold cross-validated concordance index.
when_to_use_negative:
- 'Outcome is binary rather than time-to-event: HDMAC pairs the same penalized regression
  framework with logistic regression, where a survival C-index is not the correct
  metric'
- 'Covariate dimension is around a hundred thousand variables: penalized regression
  models fall short, and HDMAC itself recommends including FDR screening first'
- Low-dimensional data with few covariates, where a standard (unpenalized) Cox model
  does not require cross-validated tuning or this evaluation scheme
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: HDMAC
  role: Web-based interactive platform implementing ridge, Lasso, and adaptive Lasso
    Cox regression with cross validation to estimate prediction power and optional
    FDR first-step screening
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Language in which HDMAC is written and all statistical analysis (penalized
    Cox fits, 5-fold cross validation, C-index computation) is performed; R scripts
    for offline use are on GitHub
  repo: https://github.com/chung-R/HD-MAC
- name: Shiny
  role: R package used to build HDMAC's interactive web interface through which the
    cross-validated penalized Cox analysis is run
- name: RStudio
  role: Environment for analyzing data offline with the HDMAC R scripts
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:488fc663a051de58c078f80a1c9ee212cce019d5836ddd813e15fa6e3b8a9366
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - RStudio
  - Shiny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-validated-concordance-index-estimation

## Summary

HDMAC's method for estimating the prediction power of penalized Cox regression models (Ridge, Lasso, adaptive Lasso) by computing the concordance index (C-index) via cross-validation on high-dimensional TCGA survival data, optionally after a first-step FDR screening.

## When to use

Use when evaluating the predictive performance of penalized Cox regression (Ridge, Lasso, adaptive Lasso) fitted to high-dimensional genomic covariates (e.g., 9,548 mRNA entries or 670 mutated genes vs. 316 patients' survival times) with a 5-fold cross-validated concordance index.

## When NOT to use

- Outcome is binary rather than time-to-event: HDMAC pairs the same penalized regression framework with logistic regression, where a survival C-index is not the correct metric
- Covariate dimension is around a hundred thousand variables: penalized regression models fall short, and HDMAC itself recommends including FDR screening first
- Low-dimensional data with few covariates, where a standard (unpenalized) Cox model does not require cross-validated tuning or this evaluation scheme

## Inputs

- TCGA ovarian HGSC Z-scored mRNA expression matrix (316 patients x 9,548 entries, |Z|>2 abnormal, retained if abnormal in >=2% of patients)
- TCGA ovarian HGSC somatic mutation matrix (670 genes retained after >=1% mutation-frequency screening)
- Overall survival times and censoring indicators for the 316 patients
- Optional FDR-screened covariate subset (Benjamini-Hochberg, threshold 0.05) from first-step univariate Cox regression

## Outputs

- Concordance index (C-index) per model-screening combination (3 penalized methods x 2 screening conditions), e.g., saved as cindex_table.csv
- Cross-validated prediction-power estimates for ridge, Lasso, and adaptive Lasso Cox fits

## How to apply

Fit each penalized Cox model (ridge, Lasso, adaptive Lasso) on the genomic covariate matrix — e.g., the TCGA ovarian HGSC mRNA matrix of 316 patients x 9,548 Z-scored entries (|Z|>2 deemed abnormal, entries abnormal in >=2% of patients retained) or the 670-gene somatic mutation matrix (genes mutated in >=1% of patients) — with tuning parameters chosen by cross-validation. Optionally apply Benjamini-Hochberg FDR control (threshold 0.05) as a first-step univariate-Cox screening, then refit the penalized models on the retained entries. Estimate prediction power of every model-screening combination with 5-fold cross validation (HDMAC's default fold number), computing the concordance index (C-index) per fold-validated fit. Assemble the C-index values into a summary table such as cindex_table.csv; for the ovarian mRNA data the reported values are 0.591/0.554/0.560 (ridge/Lasso/adaptive Lasso, no FDR) and 0.538/0.538/0.540 (FDR-screened), and for the mutation data 0.529/0.501/0.501 and 0.502/0.502/0.497. Rationale: cross validation is HDMAC's designated method to estimate each regression model's prediction power, and FDR screening is recommended when variable counts reach the magnitude of a hundred thousand where penalized regression models fall short.

## Related tools

- **HDMAC** (Web-based interactive platform implementing ridge, Lasso, and adaptive Lasso Cox regression with cross validation to estimate prediction power and optional FDR first-step screening) — https://github.com/chung-R/HD-MAC
- **R** (Language in which HDMAC is written and all statistical analysis (penalized Cox fits, 5-fold cross validation, C-index computation) is performed; R scripts for offline use are on GitHub) — https://github.com/chung-R/HD-MAC
- **Shiny** (R package used to build HDMAC's interactive web interface through which the cross-validated penalized Cox analysis is run)
- **RStudio** (Environment for analyzing data offline with the HDMAC R scripts)

## Evaluation signals

- The 6-row C-index table for the ovarian mRNA analysis must equal the reported indices: no-FDR ridge/Lasso/adaptive Lasso 0.591/0.554/0.560 and FDR-screened 0.538/0.538/0.540
- The 6-row table for the ovarian mutation analysis must reproduce 0.529/0.501/0.501 without FDR and 0.502/0.502/0.497 with FDR, with ZSWIM8 and PABPC3 in the FDR-selected gene list
- cindex_table.csv must exist in CSV format with 6 rows (3 methods x 2 screening conditions)
- Cross validation must use the default 5 folds; setting fold number to 1 should skip cross validation entirely
- Tuning parameters for each penalized Cox fit must be selected by cross validation, not fixed arbitrarily

## Limitations

- Penalized regression models fall short at variables on the magnitude of a hundred thousand, so inclusion of FDR screening is recommended for such very high-dimensional inputs
- Genes identified through this workflow are preliminary and require further in-depth study; the genome-wide search for genetic changes associated with ovarian cancer survival is still ongoing in the field
- Cross validation is not performed if the fold number is set to 1; the default is 5 folds
- Results are dataset-specific: the C-index values above are validated only for the TCGA ovarian HGSC mRNA (316 patients, 9,548 entries) and mutation (670 genes) matrices

## Evidence

- [abstract] Cross validation is provided as the method to estimate the prediction power of each regression model.: "Cross validation is provided as the method to estimate the prediction power of each regression model."
- [abstract] Choice of a first-step screening is provided to address the multiple-comparison issue that often arises with large-volume genomic data.: "Choice of a first-step screening is provided to address the multiple-comparison issue that often arises with large-volume genomic data."
- [methods] The default fold number is 5, and cross validation method will not be performed if 1 is chosen: "The default fold number is 5, and cross validation method will not be performed if 1 is chosen"
- [discussion] Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
- [methods] a preliminary screening was performed so that only the genes whose mutations were found in 1% or more and the mRNA entries whose abnormal expression was found in 2% or more of the patients were: "a preliminary screening was performed so that only the genes whose mutations were found in 1% or more and the mRNA entries whose abnormal expression was found in 2% or more of the patients were"
