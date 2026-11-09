---
name: adaptive-lasso-logistic-regression
description: Use when a binary outcome (e.g., invasive vs non-invasive bladder cancer
  subtype) must be regressed on thousands of molecular variables (e.g., ~8,024 mRNA
  Z-score entries) where ordinary logistic regression fails and variable selection
  with correct p-values is needed.
when_to_use_negative:
- The outcome is a survival time rather than a binary label — use the Cox ridge/Lasso/adaptive
  Lasso regression variants instead
- The data contain variables at the magnitude of a hundred thousand where penalized
  regression models fall short — FDR screening is recommended, or the problem may
  be infeasible
- The covariate set is small and standard (unpenalized) logistic regression with conventional
  p-values is applicable
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: HDMAC
  role: Web-based interactive platform that implements adaptive Lasso logistic regression
    with optional first-step FDR screening and cross-validation for high-dimensional
    molecular data
  repo: https://github.com/chung-R/HD-MAC
- name: selectiveInference
  role: R package used to compute correct post-selection p-values for the adaptive
    Lasso coefficients via Lee et al.'s algorithm
- name: R
  role: Language in which the platform was written and all statistical analysis performed;
    R scripts allow offline use via RStudio
  repo: https://github.com/chung-R/HD-MAC
- name: Shiny
  role: R package facilitating the building of the interactive HDMAC web app
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:5a1f4c17c37f84d538b37037b3aa1e957be8bf94a59e078a312020397ef787c1
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
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# adaptive-lasso-logistic-regression

## Summary

Fits adaptive Lasso penalized logistic regression (with optional first-step FDR screening) to high-dimensional genomic covariates for a binary outcome, via the HDMAC platform, and computes correct post-selection p-values with selectiveInference. Used to identify candidate genes associated with abnormal expression in TCGA bladder cancer data.

## When to use

Use when a binary outcome (e.g., invasive vs non-invasive bladder cancer subtype) must be regressed on thousands of molecular variables (e.g., ~8,024 mRNA Z-score entries) where ordinary logistic regression fails and variable selection with correct p-values is needed.

## When NOT to use

- The outcome is a survival time rather than a binary label — use the Cox ridge/Lasso/adaptive Lasso regression variants instead
- The data contain variables at the magnitude of a hundred thousand where penalized regression models fall short — FDR screening is recommended, or the problem may be infeasible
- The covariate set is small and standard (unpenalized) logistic regression with conventional p-values is applicable

## Inputs

- TCGA bladder cancer mRNA expression matrix (189 patients, Z-score values)
- Binary outcome vector (invasion subtype: invasive vs non-invasive)
- Pre-screened expressed entries (8,024 mRNA entries, abnormal if Z > 2 or Z < -2 in >= 2% of patients)
- Penalty type choice (adaptive Lasso) and first-step FDR screening threshold (Benjamini-Hochberg, 0.05)

## Outputs

- Table of 24 selected mRNA entries with estimated coefficients, odds ratios, and post-selection p-values (Table 4)
- Cross-validated prediction power of the fitted model (e.g., AUC)
- Fitted multivariate regression model based on the identified genes

## How to apply

Load TCGA bladder cancer data for 189 patients using Z-score mRNA expression values, defining abnormal expression as Z > 2 or Z < -2 and retaining only entries abnormal in >= 2% of patients (8,024 entries), with the binary invasion subtype outcome. Perform first-step screening by fitting univariate logistic regressions per gene and controlling the FDR with Benjamini-Hochberg at 0.05 to address the multiple-comparison problem. Fit the adaptive Lasso logistic regression with penalty weights w_j = 1/|beta_j^lasso| on the screened entries, choosing the tuning parameter by cross-validation (HDMAC default is 5-fold). Compute correct post-selection p-values using the selectiveInference R package implementing Lee et al.'s algorithm, then transform coefficients into odds ratios and assemble a table of the selected entries (24 mRNA entries in the paper's Table 4, e.g., PSORS1C2 coefficient 1.14, ICOSLG p = 0.004).

## Related tools

- **HDMAC** (Web-based interactive platform that implements adaptive Lasso logistic regression with optional first-step FDR screening and cross-validation for high-dimensional molecular data) — https://github.com/chung-R/HD-MAC
- **selectiveInference** (R package used to compute correct post-selection p-values for the adaptive Lasso coefficients via Lee et al.'s algorithm)
- **R** (Language in which the platform was written and all statistical analysis performed; R scripts allow offline use via RStudio) — https://github.com/chung-R/HD-MAC
- **Shiny** (R package facilitating the building of the interactive HDMAC web app)

## Examples

```
Rscript -e "library(HDMAC); hdmac(x = expr_zscores, y = invasion_subtype, penalty = 'adaptive.lasso', family = 'binomial', screen = 'FDR', fdr = 0.05, nfolds = 5)"
```

## Evaluation signals

- The output contains exactly 24 gene rows reproducing Table 4 of the paper, with fields for estimated coefficient, odds ratio, and p-value present
- Spot-check values against Table 4: PSORS1C2 coefficient = 1.14 and ICOSLG p-value = 0.004
- The number of candidate entries entering the adaptive Lasso matches the Benjamini-Hochberg FDR screen at 0.05 applied to per-gene univariate logistic regressions
- Cross-validated prediction power (e.g., AUC) of the adaptive-Lasso-with-FDR model matches the value reported in Table 3 (e.g., 64.300 for the multivariate invasion model with threshold 0.34)
- Post-selection p-values are produced by selectiveInference (Lee et al. algorithm), not naive summary statistics

## Limitations

- Penalized regression models fall short at around a hundred thousand variables; inclusion of FDR screening is recommended at that scale
- Correct p-values for Lasso/adaptive Lasso are not usually available in common statistical software and require specialized methods (selectiveInference)
- Genes identified are preliminary and require further in-depth study; clinical variables such as gender and age may also be included although not illustrated in the paper's results

## Evidence

- [abstract] Adaptive Lasso logistic regression with FDR screening on TCGA bladder data: "Choice of a first-step screening is provided to address the multiple-comparison issue that often arises with large-volume genomic data."
- [abstract] Cross-validation for prediction power: "Cross validation is provided as the method to estimate the prediction power of each regression model."
- [methods] selectiveInference for correct p-values: "Here we used the ‘selectiveInference’ R package29,30 to implement the algorithm by Lee et al. to compute the correct p-values"
- [methods] Z-score abnormality threshold and 2% frequency screening: "a preliminary screening was performed so that only the genes whose mutations were found in 1% or more and the mRNA entries whose abnormal expression was found in 2% or more of the patients were"
- [methods] FDR as optional first-step screening: "the method to control the FDR is provided as an optional first-step screening method"
- [methods] Default 5-fold cross validation: "The default fold number is 5, and cross validation method will not be performed if 1 is chosen"
- [discussion] Scale limitation of penalized regression: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
