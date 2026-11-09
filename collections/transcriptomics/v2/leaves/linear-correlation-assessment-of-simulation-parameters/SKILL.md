---
name: linear-correlation-assessment-of-simulation-parameters
description: Use when you have simulated perturbation (Y) and control (Z) scCRISPR-seq
  matrices spanning a synthetic (d, r) parameter grid and need to verify that a derived
  quantity depends linearly on both simulation parameters, per the paper's Equations
  14-15.
when_to_use_negative:
- Input is real scCRISPR-seq data rather than the synthetic two-cluster matrices spanning
  a simulated (d, r) parameter grid — the factorized bias term (lambda_1 - delta_1)(mu_1
  - mu_2) is only exact in the two-cluster case
- Goal is estimating cellular response or infected proportion bias itself via the
  scDecouple GMM/EM pipeline (run_scDecouple) rather than validating the paper's linear
  derivation of Equations 14-15
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDecouple
  role: Provides the theoretical framework (Equations 14-15) whose linear bias dependence
    on d and r is being assessed on the synthetic data
  repo: https://github.com/MengQiuchen/scDecouple
- name: R
  role: Implementation language for computing the bias term, Pearson correlations,
    and parameter tables
- name: tidyverse
  role: Used with dplyr to assemble per-parameter bias values, d, r, and correlation
    coefficients into a tidy table, and ggplot2 for the regression plots
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:f7d3b32b5f81c47efcdcdaa091bf8278d7ffde25c9dbc2daa37af53be884f4ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# linear-correlation-assessment-of-simulation-parameters

## Summary

Validates theoretical linear relationships in scDecouple's synthetic two-cluster simulation by computing the infected proportion bias term (lambda_1 - delta_1)(mu_1 - mu_2) across a grid of cluster distance (d) and ratio change (r) parameters and testing its Pearson correlation with each, as asserted by Equations 14-15.

## When to use

Use when you have simulated perturbation (Y) and control (Z) scCRISPR-seq matrices spanning a synthetic (d, r) parameter grid and need to verify that a derived quantity depends linearly on both simulation parameters, per the paper's Equations 14-15.

## When NOT to use

- Input is real scCRISPR-seq data rather than the synthetic two-cluster matrices spanning a simulated (d, r) parameter grid — the factorized bias term (lambda_1 - delta_1)(mu_1 - mu_2) is only exact in the two-cluster case
- Goal is estimating cellular response or infected proportion bias itself via the scDecouple GMM/EM pipeline (run_scDecouple) rather than validating the paper's linear derivation of Equations 14-15

## Inputs

- Synthetic two-cluster perturbation expression matrix Y (cells x genes) from the d/r simulation card
- Synthetic two-cluster control expression matrix Z (cells x genes) from the d/r simulation card
- Simulated parameter grid of cluster distances d and ratio changes r

## Outputs

- bias_vs_dr_table.csv — tidy table with one row per (d, r) combination containing bias values, d, r, and Pearson correlation coefficients
- bias_vs_dr_correlation.png — ggplot2 scatter plots of bias versus d and bias versus r with fitted linear regression lines annotated with correlation coefficients

## How to apply

Load the synthetic two-cluster perturbation (Y) and control (Z) expression matrices produced by the upstream d/r simulation card. For each (d, r) combination, compute the infected proportion bias on each PC factor as the product of the cluster-proportion difference (lambda_1 - delta_1) and the mean-expression difference between perturbation and control groups (mu_1 - mu_2) in R. Compute Pearson correlation coefficients of the bias term against d and against r to test the linear dependence defined in Equations 14-15. Assemble bias values, d, r, and correlations into a tidy table with tidyverse/dplyr, write it to bias_vs_dr_table.csv, and use ggplot2 to plot bias versus d and bias versus r with fitted linear regression lines annotated with the correlation coefficients (bias_vs_dr_correlation.png). Correctness is judged by approximately linear trends with high positive Pearson correlation for both parameters and a complete table with one row per simulated (d, r) combination and no missing bias values.

## Related tools

- **scDecouple** (Provides the theoretical framework (Equations 14-15) whose linear bias dependence on d and r is being assessed on the synthetic data) — https://github.com/MengQiuchen/scDecouple
- **R** (Implementation language for computing the bias term, Pearson correlations, and parameter tables)
- **tidyverse** (Used with dplyr to assemble per-parameter bias values, d, r, and correlation coefficients into a tidy table, and ggplot2 for the regression plots)

## Examples

```
res <- run_scDecouple(Y, Z); cor(as.data.frame(bias_table)$d, as.data.frame(bias_table)$bias, method="pearson")
```

## Evaluation signals

- Bias values show an approximately linear trend with both d and r, with high positive Pearson correlation consistent with the paper's linear derivation
- bias_vs_dr_table.csv contains one row per simulated (d, r) combination
- No missing bias values in the output table
- ggplot2 figure shows fitted linear regression lines annotated with the correlation coefficients for both bias-vs-d and bias-vs-r panels

## Limitations

- The exact factorization of the infected proportion bias as (lambda_1 - delta_1)(mu_1 - mu_2) holds only in the two-cluster synthetic case; more complex cluster structures may not exhibit this linear dependence
- scDecouple's Gaussian mixture model assumption may not hold in intricate systems or disease tissues such as tumors
- The linear relationship assumes the same cell clusters are present between NT and perturbation groups, which can fail when rare cell types are completely absent from infection

## Evidence

- [results] in the two-cluster case the infected proportion bias on any PC factorizes exactly as the product of the ratio change (lambda_1-delta_1) and the cluster distance (mu_1-mu_2): [evidence span withheld — anchor status: not_found]
- [results] Compute Pearson correlation coefficients of the bias term against d and against r to test the linear relationship asserted by Equations 14-15.: [evidence span withheld — anchor status: not_found]
- [results] the bias values must show an approximately linear trend with both d and r (high positive Pearson correlation, consistent with the paper's linear derivation): [evidence span withheld — anchor status: not_found]
- [discussion] We have built an R package for scDecouple, offering a comprehensive suite of functionalities for streamlined scCRISPR-seq data analysis.: "We have built an R package for scDecouple, offering a comprehensive suite of functionalities for streamlined scCRISPR-seq data analysis."
