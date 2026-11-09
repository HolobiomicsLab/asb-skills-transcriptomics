---
name: bladder-cancer-invasion-subtype-classification
description: Use when classifying bladder cancer patients into invasive vs non-invasive
  subtypes from TCGA mRNA expression Z-scores, or when validating a penalized-regression
  gene panel (e.g., adaptive Lasso with FDR screening) as a multivariate predictor
  with cross-validated AUC.
when_to_use_negative:
- When the feature space is on the order of a hundred thousand variables, where penalized
  regression models fall short and FDR screening alone is recommended instead
- When the outcome is survival rather than a binary invasive/non-invasive label —
  HDMAC's Cox penalized regression branch, not the logistic model with a 0.34 score
  threshold, applies
- When no pre-screened gene coefficient list exists — the classifier requires genes
  already selected by adaptive Lasso with FDR, not raw genome-wide expression
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: HDMAC
  role: Web-based platform (and R scripts) that performs FDR first-step screening,
    adaptive Lasso logistic regression, and 5-fold cross validation to estimate prediction
    power of the multivariate invasion model
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Statistical computing language in which HDMAC is written and through which
    the multivariate log-odds scoring and TCGA data download are performed
- name: selectiveInference
  role: R package used to compute correct post-selection p-values for the Lasso/adaptive
    Lasso gene selection step
- name: Shiny
  role: R package enabling HDMAC's interactive web interface for the classification
    workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:4c7df13882b17941c9f12848602b54a186e302c8b9d9a7329332acb767925cf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - Shiny
  - selectiveInference
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bladder-cancer-invasion-subtype-classification

## Summary

A reproducible workflow for classifying TCGA bladder cancer patients as muscle-invasive vs non-invasive using a multivariate log-odds score built from 24 FDR-selected abnormally expressed genes (adaptive Lasso, Table 4), with a 0.34 decision threshold and 5-fold cross-validated performance (AUC) assessed via HDMAC.

## When to use

Use when classifying bladder cancer patients into invasive vs non-invasive subtypes from TCGA mRNA expression Z-scores, or when validating a penalized-regression gene panel (e.g., adaptive Lasso with FDR screening) as a multivariate predictor with cross-validated AUC.

## When NOT to use

- When the feature space is on the order of a hundred thousand variables, where penalized regression models fall short and FDR screening alone is recommended instead
- When the outcome is survival rather than a binary invasive/non-invasive label — HDMAC's Cox penalized regression branch, not the logistic model with a 0.34 score threshold, applies
- When no pre-screened gene coefficient list exists — the classifier requires genes already selected by adaptive Lasso with FDR, not raw genome-wide expression

## Inputs

- TCGA bladder cancer mRNA expression matrix (189 patients, 8,024 pre-screened expressed entries)
- Coefficient list of the 24 FDR-selected abnormally expressed genes from adaptive Lasso logistic regression (Table 4, SPTSSA through ZSCAN2)
- Decision threshold value (0.34)

## Outputs

- Per-patient multivariate log-odds scores and binary invasive/non-invasive predicted labels
- Cross-validated performance metrics (sensitivity, specificity, accuracy, AUC) from 5-fold CV
- Predicted-label table comparable to Table 4 gene list

## How to apply

Load the TCGA bladder cancer mRNA expression matrix (189 patients, 8,024 pre-screened expressed entries) and the coefficient list of the 24 abnormally expressed genes selected by adaptive Lasso logistic regression after FDR penalty (SPTSSA through ZSCAN2, Table 4). Compute each patient's multivariate log-odds score as the coefficient-weighted linear combination of the selected genes' abnormal-expression Z-scores, treating a Z above 2 or below −2 as abnormal. Classify patients as invasive when the score exceeds the reported threshold of 0.34, and non-invasive otherwise, to build the predicted-label table. Assess predictive performance with HDMAC's default 5-fold cross validation, computing sensitivity, specificity, accuracy, and cross-validated AUC. Correct p-values for the Lasso/adaptive Lasso selection step are computed with the selectiveInference R package (Lee et al. algorithm).

## Related tools

- **HDMAC** (Web-based platform (and R scripts) that performs FDR first-step screening, adaptive Lasso logistic regression, and 5-fold cross validation to estimate prediction power of the multivariate invasion model) — https://github.com/chung-R/HD-MAC
- **R** (Statistical computing language in which HDMAC is written and through which the multivariate log-odds scoring and TCGA data download are performed)
- **selectiveInference** (R package used to compute correct post-selection p-values for the Lasso/adaptive Lasso gene selection step)
- **Shiny** (R package enabling HDMAC's interactive web interface for the classification workflow)

## Examples

```
Rscript hdmac_adaptive_lasso_cv.R --expression tcga_blca_mrna.csv --genes table4_24genes.csv --threshold 0.34 --folds 5
```

## Evaluation signals

- Predicted-label table reproduces the reported Table 4 gene list (SPTSSA through ZSCAN2, 24 genes)
- Cross-validated AUC matches the adaptive-Lasso-with-FDR value of 64.300 reported in Table 3
- Patient classification is consistent: log-odds score > 0.34 → invasive, otherwise non-invasive
- Abnormal-expression calls match the Z-score rule (Z > 2 or Z < −2) applied to the 8,024 pre-screened entries
- 5-fold cross validation uses HDMAC's default fold number; CV results should be stable across fold assignments

## Limitations

- The 24 identified genes are preliminary and provide rational directions for further in-depth study rather than a validated clinical classifier
- Penalized regression models fall short at ~hundred thousand variables; FDR screening is recommended at that scale
- Clinical variables such as gender and age may be included in the analysis but were not illustrated in the reported results

## Evidence

- [abstract] HDMAC estimates the prediction power of each regression model by cross validation: "Cross validation is provided as the method to estimate the prediction power of each regression model."
- [methods] Z-score abnormality threshold: "A Z score above 2 or below −2 was considered abnormal"
- [methods] Default 5-fold cross validation: "The default fold number is 5, and cross validation method will not be performed if 1 is chosen"
- [methods] selectiveInference for correct p-values: "Here we used the ‘selectiveInference’ R package29,30 to implement the algorithm by Lee et al. to compute the correct p-values"
- [discussion] FDR screening for very large variable counts: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
