---
name: high-dimensional-genomic-covariate-regression
description: Use when fitting Cox or logistic regression with thousands of gene-expression/mutation
  covariates (e.g., TCGA mRNA Z-scores) and you need penalized variable selection,
  optional FDR screening, and cross-validated AUC/C-index.
when_to_use_negative:
- Dealing with variables at the magnitude of a hundred thousand, where penalized regression
  models fall short — FDR screening is recommended in that regime
- You need correct p-values for ridge regression — the selectiveInference-based post-selection
  p-values are provided only for the Lasso and adaptive Lasso
- You only need interactive visualization of existing TCGA data rather than regression
  modeling — tools like cBioPortal, HPA, or miRGator cover display of RNA expression,
  mutations, and copy-number alterations
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: HDMAC
  role: Web-based interactive platform (built with Shiny) that fits ridge/Lasso/adaptive
    Lasso Cox and logistic regression, offers FDR first-step screening, and runs 5-fold
    cross validation
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Language in which the platform was written and all statistical analysis performed;
    R scripts on GitHub let RStudio users run HDMAC offline
  repo: https://github.com/chung-R/HD-MAC
- name: selectiveInference
  role: R package implementing the Lee et al. algorithm to compute correct post-selection
    p-values for variables retained by Lasso and adaptive Lasso fits
- name: Shiny
  role: R package used to build the interactive HDMAC web app from R
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:389e2fdd86d903429dff5901acab89687671d03b4c94b5f3360ad6eb2621a881
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# high-dimensional-genomic-covariate-regression

## Summary

Fit penalized regression models (Ridge, Lasso, adaptive Lasso; Cox for survival outcomes, logistic for binary outcomes) to high-dimensional genomic covariates via the HDMAC platform, with optional FDR first-step screening and cross-validated prediction power.

## When to use

Use when fitting Cox or logistic regression with thousands of gene-expression/mutation covariates (e.g., TCGA mRNA Z-scores) and you need penalized variable selection, optional FDR screening, and cross-validated AUC/C-index.

## When NOT to use

- Dealing with variables at the magnitude of a hundred thousand, where penalized regression models fall short — FDR screening is recommended in that regime
- You need correct p-values for ridge regression — the selectiveInference-based post-selection p-values are provided only for the Lasso and adaptive Lasso
- You only need interactive visualization of existing TCGA data rather than regression modeling — tools like cBioPortal, HPA, or miRGator cover display of RNA expression, mutations, and copy-number alterations

## Inputs

- High-dimensional genomic covariate matrix (TCGA mRNA expression Z-scores; e.g., 189 bladder-cancer patients × 8,024 screened entries or 316 ovarian HGSC patients × 9,548 entries)
- Binary outcome labels (e.g., invasive vs. non-invasive bladder cancer subtype) or survival outcomes (overall survival time plus censoring indicators)
- Optional clinical covariates (e.g., gender, age)
- Fitted Lasso/adaptive Lasso model objects (for selectiveInference post-selection p-values)

## Outputs

- Cross-validated prediction-power table (AUC for binary outcomes, e.g., adaptive Lasso 64.300 with FDR; C-index for survival, e.g., 0.591 ridge without FDR), with sensitivity, specificity, and accuracy
- List of selected genes with coefficients (e.g., the 24 adaptive-Lasso FDR-selected abnormally expressed bladder genes, SPTSSA through ZSCAN2)
- Post-selection p-value table for Lasso/adaptive Lasso with fields variable, coefficient, and p-value (e.g., lasso_selective_pvalues.csv)

## How to apply

Load the high-dimensional genomic matrix (e.g., TCGA bladder cancer mRNA: 189 patients × 8,024 entries retained after Z-score ±2 / ≥2%-abnormality screening; TCGA ovarian HGSC: 316 patients × 9,548 entries) plus the outcome (binary invasion labels or survival time with censoring indicators), using the provided R codes for TCGA download. If the multiple-testing problem is a concern, apply Benjamini-Hochberg FDR control as an optional first-step screening at the default 0.05 threshold, based on the univariate logistic or Cox regression p-value per gene. Fit ridge, Lasso, and adaptive Lasso penalized regression via HDMAC (implemented in R with Shiny), selecting tuning parameters by cross validation. Estimate each model's prediction power with 5-fold cross validation (the default fold number; CV is skipped if 1 is chosen), computing AUC for binary outcomes or C-index for survival, alongside sensitivity, specificity, and accuracy. For post-selection inference on Lasso/adaptive Lasso fits, compute correct p-values using the 'selectiveInference' R package implementing the Lee et al. conditional approach. Assemble the model×screening metric table (e.g., six rows for 3 methods × FDR/no-FDR) and save for validation against reported values.

## Related tools

- **HDMAC** (Web-based interactive platform (built with Shiny) that fits ridge/Lasso/adaptive Lasso Cox and logistic regression, offers FDR first-step screening, and runs 5-fold cross validation) — https://github.com/chung-R/HD-MAC
- **R** (Language in which the platform was written and all statistical analysis performed; R scripts on GitHub let RStudio users run HDMAC offline) — https://github.com/chung-R/HD-MAC
- **selectiveInference** (R package implementing the Lee et al. algorithm to compute correct post-selection p-values for variables retained by Lasso and adaptive Lasso fits)
- **Shiny** (R package used to build the interactive HDMAC web app from R)

## Examples

```
Rscript -e 'shiny::runGitHub("HD-MAC", "chung-R")'
```

## Evaluation signals

- Bladder cancer binary-outcome CV AUC matches reported Table 3 values: ridge 68.107 (no FDR) / 66.515 (FDR), Lasso 65.864/67.020, adaptive Lasso 62.442/64.300
- Ovarian HGSC survival CV C-index matches reported values: 0.591/0.554/0.560 (ridge/Lasso/adaptive Lasso, no FDR) and 0.538/0.538/0.540 (FDR-screened)
- lasso_selective_pvalues.csv exists, contains the fields variable, coefficient, and p-value, and all p-values lie in [0,1]
- The multivariate log-odds score with the reported 0.34 threshold reproduces the predicted-label table, and the selected-gene table matches the Table 4 list (SPTSSA through ZSCAN2, 24 genes)
- Screening filter reproduces the stated data dimensions (e.g., 8,024 bladder entries, 9,548 ovarian entries retained after Z > 2 / Z < -2 and ≥2%-of-patients abnormality screening)

## Limitations

- Penalized regression models fall short when variables number around a hundred thousand; FDR inclusion is recommended in that regime
- Correct p-values are provided only for the Lasso and adaptive Lasso and are not usually available in common statistical software due to variable selection
- Genes identified are preliminary and provide only rational directions for further molecular exploration; genome-wide search for survival-associated changes in ovarian cancer remains outstanding
- Cross validation is not performed if fold number 1 is chosen, and the default is 5 folds
- Downloading TCGA data from cBioPortal allows only limited numbers of genetic entries per download; use the provided R codes instead

## Evidence

- [abstract] Cross validation is provided as the method to estimate the prediction power of each regression model.: "Cross validation is provided as the method to estimate the prediction power of each regression model."
- [methods] the method to control the FDR is provided as an optional first-step screening method: "the method to control the FDR is provided as an optional first-step screening method"
- [methods] Here we used the ‘selectiveInference’ R package29,30 to implement the algorithm by Lee et al. to compute the correct p-values: "Here we used the ‘selectiveInference’ R package29,30 to implement the algorithm by Lee et al. to compute the correct p-values"
- [methods] The default fold number is 5, and cross validation method will not be performed if 1 is chosen: "The default fold number is 5, and cross validation method will not be performed if 1 is chosen"
- [discussion] Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
