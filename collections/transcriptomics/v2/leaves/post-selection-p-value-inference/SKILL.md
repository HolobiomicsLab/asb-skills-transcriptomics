---
name: post-selection-p-value-inference
description: Use when p-values are needed for variables selected by Lasso or adaptive
  Lasso penalized regression on high-dimensional genomic covariates, since standard
  software p-values are invalid after variable selection.
when_to_use_negative:
- The model was fit by ridge or ordinary (unpenalized) regression — selection-based
  p-value correction targets Lasso/adaptive Lasso selection events
- The p-values are needed for prediction assessment rather than variable-level inference
  — cross-validation prediction power is the HDMAC mechanism for that
- The number of variables is around a hundred thousand where penalized regression
  falls short — HDMAC recommends FDR screening first rather than relying on the penalized
  fit directly
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: selectiveInference
  role: R package implementing Lee et al.'s algorithm to compute correct p-values
    after Lasso/adaptive Lasso model selection
- name: HDMAC
  role: Web-based/Shiny platform that fits the Lasso and adaptive Lasso models with
    cross-validation and provides the correct post-selection p-values
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Statistical computing environment in which HDMAC and selectiveInference analysis
    runs, including offline use via RStudio
  repo: https://github.com/chung-R/HD-MAC
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:953c680951111b862614c5e048dd9b70584fcc70ebe283f31879f484854ddcc2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - Shiny
  - selectiveInference
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# post-selection-p-value-inference

## Summary

Computes valid conditional post-selection p-values for variables retained by Lasso and adaptive Lasso penalized regression (Cox and logistic variants) using the selectiveInference R package implementing Lee et al.'s algorithm, as provided by the HDMAC platform for high-dimensional TCGA genomic analyses.

## When to use

Use when p-values are needed for variables selected by Lasso or adaptive Lasso penalized regression on high-dimensional genomic covariates, since standard software p-values are invalid after variable selection.

## When NOT to use

- The model was fit by ridge or ordinary (unpenalized) regression — selection-based p-value correction targets Lasso/adaptive Lasso selection events
- The p-values are needed for prediction assessment rather than variable-level inference — cross-validation prediction power is the HDMAC mechanism for that
- The number of variables is around a hundred thousand where penalized regression falls short — HDMAC recommends FDR screening first rather than relying on the penalized fit directly

## Inputs

- Fitted Lasso or adaptive Lasso logistic/Cox regression model object (tuning parameter selected by cross-validation)
- High-dimensional genomic covariate matrix the model was fit on (e.g., TCGA Z-score mRNA entries, binary invasion subtype or survival outcome)
- Set of retained variables from the penalized fit (optionally after FDR first-step screening)

## Outputs

- Table with columns variable, coefficient, and p-value (post-selection p-values in [0,1])
- For logistic models, odds ratios (exp(coefficient)) alongside estimated coefficients and p-values

## How to apply

Fit the penalized model first (Lasso or adaptive Lasso logistic/Cox regression on the high-dimensional covariate matrix, e.g., the 8,024 Z-score mRNA entries from 189 TCGA bladder cancer patients, optionally after Benjamini-Hochberg FDR first-step screening at 0.05), with the tuning parameter selected by cross-validation (default 5-fold in HDMAC). Then apply the selectiveInference framework of Lee et al. to compute conditional post-selection p-values for each retained variable, which correct for the selection step that ordinary software ignores. For adaptive Lasso, penalty weights are w_j = 1/|beta_j^lasso|. Assemble results per variable with coefficient and p-value columns (p-values must lie in [0,1]); coefficients from logistic fits can additionally be exponentiated to odds ratios, as in the 24-entry bladder cancer invasion table (e.g., PSORS1C2 coefficient 1.14, ICOSLG p = 0.004).

## Related tools

- **selectiveInference** (R package implementing Lee et al.'s algorithm to compute correct p-values after Lasso/adaptive Lasso model selection)
- **HDMAC** (Web-based/Shiny platform that fits the Lasso and adaptive Lasso models with cross-validation and provides the correct post-selection p-values) — https://github.com/chung-R/HD-MAC
- **R** (Statistical computing environment in which HDMAC and selectiveInference analysis runs, including offline use via RStudio) — https://github.com/chung-R/HD-MAC

## Evaluation signals

- Output table contains exactly the columns variable, coefficient, and p-value, with every p-value in [0,1]
- For the bladder cancer invasion case study, the table contains exactly 24 gene rows reproducing Table 4 (e.g., PSORS1C2 coefficient 1.14, ICOSLG p = 0.004), with coefficient, odds ratio, and p-value fields present
- The R script runs end-to-end and produces the expected p-value CSV (script_runs and field_present checks)
- p-values are only computed for variables actually retained by the penalized fit (post-selection scope, not all candidate entries)

## Limitations

- Correct p-values for Lasso/adaptive Lasso are not usually available in common statistical software due to the methods' involvement in variable selection, so bespoke selective-inference tooling is required
- When variable counts reach the magnitude of a hundred thousand, penalized regression models fall short; FDR first-step screening is recommended before applying this inference step
- Penalty weights for adaptive Lasso (w_j = 1/|beta_j^lasso|) depend on an initial Lasso fit, so an incorrectly tuned first-stage Lasso propagates into the inference

## Evidence

- [methods] selectiveInference used for Lee et al. algorithm: "Here we used the ‘selectiveInference’ R package29,30 to implement the algorithm by Lee et al. to compute the correct p-values"
- [methods] post-selection p-value computation approach: "developed a general approach to compute the correct p-values after model selection"
- [intro] cross-validation with correct p-values for Lasso and adaptive Lasso: "it would estimate the prediction power for each regression model using cross validation with the correct p-values for the Lasso and adaptive Lasso provided"
- [discussion] why standard software p-values are unavailable: "which are not usually available in common statistical software due to the methods’ involvement in variable selection"
- [methods] adaptive Lasso penalty weights: "The adaptive Lasso logistic regression estimator"
