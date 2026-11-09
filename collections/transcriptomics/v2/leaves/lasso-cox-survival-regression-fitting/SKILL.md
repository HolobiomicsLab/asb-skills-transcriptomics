---
name: lasso-cox-survival-regression-fitting
description: Use when regressing a survival outcome (survival time plus censoring
  indicator) on high-dimensional molecular covariates (e.g. TCGA gene-level features
  far outnumbering patients) and a sparsity-inducing penalty is needed for variable
  selection.
when_to_use_negative:
- The outcome is binary rather than a survival time/censoring pair — use the penalized
  logistic regression routine instead
- Variables are at the magnitude of a hundred thousand, where penalized regression
  models fall short — apply the optional FDR first-step screening instead
- Only a small number of variables with standard low-dimensional regression needs;
  the article targets high-dimensional genomic data for which existing platforms were
  lacking
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: HDMAC
  role: Web-based interactive platform that implements the Cox Lasso regression estimator
    with penalty-type choice and cross validation
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Language in which the platform and all statistical analysis (Cox Lasso fitting,
    cross validation) were written and performed
- name: Shiny
  role: R package that exposes the Cox Lasso regression fitting as an interactive
    web app within HDMAC
- name: selectiveInference
  role: R package used to compute correct p-values for Lasso model selection after
    fitting
- name: RStudio
  role: Environment for offline use of the HDMAC R scripts to run the Cox Lasso analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:8fe25d51eafacfb11402bb30259a158756d786ebd959282eab9128d18d59c6a1
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
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lasso-cox-survival-regression-fitting

## Summary

Fit a Lasso-penalized Cox regression (as implemented in HDMAC, an R/Shiny platform) to high-dimensional genomic covariates with a survival outcome, using 5-fold cross validation to estimate prediction power and optionally the selectiveInference package for correct post-selection p-values.

## When to use

Use when regressing a survival outcome (survival time plus censoring indicator) on high-dimensional molecular covariates (e.g. TCGA gene-level features far outnumbering patients) and a sparsity-inducing penalty is needed for variable selection.

## When NOT to use

- The outcome is binary rather than a survival time/censoring pair — use the penalized logistic regression routine instead
- Variables are at the magnitude of a hundred thousand, where penalized regression models fall short — apply the optional FDR first-step screening instead
- Only a small number of variables with standard low-dimensional regression needs; the article targets high-dimensional genomic data for which existing platforms were lacking

## Inputs

- High-dimensional molecular covariate matrix (e.g. TCGA gene mutation/expression features), optionally with clinical covariates such as gender and age
- Survival outcome: survival time plus censoring indicator per sample
- Penalty-type choice (Lasso among Ridge/Lasso/adaptive Lasso) and 5-fold cross-validation setting

## Outputs

- Fitted Lasso Cox regression model with selected important genetic alterations
- Cross-validated prediction-power estimate for the model
- Correct post-selection p-values for Lasso-selected variables (via selectiveInference)
- List of candidate genes potentially associated with survival (e.g. ASAP3 in ovarian cancer)

## How to apply

Supply a covariate matrix of high-dimensional molecular variables together with survival time and censoring indicator, optionally including clinical covariates such as gender and age. HDMAC offers a penalty-type choice among Ridge, Lasso, and adaptive Lasso; select the Cox Lasso regression estimator for the survival outcome, keeping the default 5-fold cross validation to estimate prediction power (cross validation is skipped if fold number 1 is chosen). If the multiple-testing problem is of concern, apply the optional first-step FDR screening to screen out unrelated variables before penalized fitting — this is recommended when variables reach the magnitude of a hundred thousand, where penalized regression models fall short. After model selection, correct p-values for the Lasso (via the selectiveInference R package implementing the Lee et al. algorithm) are provided, since these are not usually available in common statistical software. The workflow runs either through the Shiny web interface or offline in RStudio using the HDMAC R scripts on GitHub.

## Related tools

- **HDMAC** (Web-based interactive platform that implements the Cox Lasso regression estimator with penalty-type choice and cross validation) — https://github.com/chung-R/HD-MAC
- **R** (Language in which the platform and all statistical analysis (Cox Lasso fitting, cross validation) were written and performed)
- **Shiny** (R package that exposes the Cox Lasso regression fitting as an interactive web app within HDMAC)
- **selectiveInference** (R package used to compute correct p-values for Lasso model selection after fitting)
- **RStudio** (Environment for offline use of the HDMAC R scripts to run the Cox Lasso analysis)

## Evaluation signals

- Cross validation (default 5-fold) reports a finite prediction-power estimate for the fitted Cox Lasso model; setting fold number to 1 correctly disables cross validation
- The fitted model identifies a sparse subset of genes rather than retaining all high-dimensional covariates
- Correct p-values are returned for Lasso-selected variables (via the Lee et al. algorithm in selectiveInference)
- Survival analysis computing time exceeds logistic-regression time for the largest observation count (e.g. 12.8 s vs 4.3 s for 1000 observations x 50 variables; 128.2 s vs 16.4 s for 1000 x 5000)
- Timing increases monotonically with both the number of observations and the number of variables

## Limitations

- Penalized regression models fall short at variables on the magnitude of a hundred thousand; FDR first-step screening is recommended there
- Correct p-values for Lasso/adaptive Lasso are not usually available in common statistical software and rely on the selectiveInference implementation
- Genes identified are a preliminary list requiring further in-depth study; genome-wide search for genetic changes associated with ovarian cancer survival is still lacking

## Evidence

- [methods] The Cox Lasso regression estimator: "The Cox Lasso regression estimator"
- [abstract] Cross validation: "Cross validation is provided as the method to estimate the prediction power of each regression model."
- [discussion] FDR recommendation: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
- [methods] Shiny: "with the help of Shiny, an R package that facilitates the building of interactive web Apps straight from R"
- [methods] GitHub repo: "the codes are available at GitHub (https://github.com/chung-R/HD-MAC)"
