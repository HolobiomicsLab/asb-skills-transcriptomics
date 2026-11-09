---
name: two-cluster-gaussian-simulation-analysis
description: Use when analyzing a simulated two-cluster perturbation/control parameter
  grid varying cluster distance d and ratio change r, to verify that infected proportion
  bias on PCs scales linearly with d and r as derived in Equations 14-15.
when_to_use_negative:
- Real scCRISPR-seq data with more than two clusters or unknown cluster structure
  — the exact linear factorization (Equations 14-15) only holds in the two-cluster
  synthetic scenario; use the full scDecouple GMM/EM decoupling pipeline instead
- Systems where the Gaussian mixture assumption may not hold, such as intricate systems
  or disease tissues like tumors, or when rare cell types are completely absent from
  infection
- Analysis of genuine biological perturbation responses rather than validating the
  mathematical bias model — downstream gene ranking and pathway enrichment (scDecouple
  step 4) is the appropriate skill
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: scDecouple
  role: R package whose preprocessing (data_preprocessing) and decoupling framework
    (Gaussian mixture model + EM maximum likelihood) provides the log-normalized data
    and PC-level outputs on which the two-cluster bias factorization is computed
  repo: https://github.com/MengQiuchen/scDecouple
- name: tidyverse
  role: dplyr/tidyverse used to assemble the per-parameter bias values, d, r, and
    correlation coefficients into a tidy table; ggplot2 used for the regression plots
- name: R
  role: Implementation language for the bias computation, Pearson correlation testing,
    and figure generation
  repo: https://github.com/MengQiuchen/scDecouple
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:1b2cb1e42c1487a5db2497b81a68741f7d1c398a2b49361c9f097816b609d604
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# two-cluster-gaussian-simulation-analysis

## Summary

Validates the linear factorization of infected proportion bias in the paper's synthetic two-cluster scCRISPR-seq scenario, where the bias on any PC equals (lambda_1 - delta_1)(mu_1 - mu_2) — the product of the cluster-proportion ratio change r and the mean-expression cluster distance d (Equations 14-15).

## When to use

Use when analyzing a simulated two-cluster perturbation/control parameter grid varying cluster distance d and ratio change r, to verify that infected proportion bias on PCs scales linearly with d and r as derived in Equations 14-15.

## When NOT to use

- Real scCRISPR-seq data with more than two clusters or unknown cluster structure — the exact linear factorization (Equations 14-15) only holds in the two-cluster synthetic scenario; use the full scDecouple GMM/EM decoupling pipeline instead
- Systems where the Gaussian mixture assumption may not hold, such as intricate systems or disease tissues like tumors, or when rare cell types are completely absent from infection
- Analysis of genuine biological perturbation responses rather than validating the mathematical bias model — downstream gene ranking and pathway enrichment (scDecouple step 4) is the appropriate skill

## Inputs

- Synthetic perturbation group matrix Y (cells x genes) spanning the simulated d and r parameter grid
- Synthetic control group matrix Z (cells x genes) from the upstream two-cluster d/r simulation
- Simulated parameter grid of cluster distance d and ratio change r values

## Outputs

- bias_vs_dr_table.csv — one tidy row per simulated (d, r) combination with the computed bias term (lambda_1 - delta_1)(mu_1 - mu_2) and Pearson correlations against d and r
- bias_vs_dr_correlation.png — ggplot2 figure of bias versus d and bias versus r with fitted linear regression lines annotated with correlation coefficients

## How to apply

Load the synthetic perturbation (Y) and control (Z) cell-by-gene expression matrices spanning the simulated (d, r) grid, preprocess them with scDecouple's data_preprocessing (log-normalization and highly variable gene detection) to obtain log-normalized matrices and PCA results. For each (d, r) combination, compute the infected proportion bias term (lambda_1 - delta_1)(mu_1 - mu_2) as the product of the cluster-proportion difference between perturbation and control groups and the mean-expression difference between the two clusters (mu_1 - mu_2), per the paper's exact factorization. Compute Pearson correlation coefficients of the bias term against d and against r, assemble per-parameter bias values, d, r, and correlations into a tidy dplyr table (bias_vs_dr_table.csv), and fit linear regressions of bias versus d and bias versus r in ggplot2, annotating the plots with the correlations (bias_vs_dr_correlation.png). The rationale is that scDecouple decouples true cellular response from infected proportion bias via maximum likelihood with a Gaussian mixture model and EM algorithm, so the two-cluster Gaussian setting is the controlled case where the linear bias decomposition (Equations 14-15) can be verified analytically and empirically.

## Related tools

- **scDecouple** (R package whose preprocessing (data_preprocessing) and decoupling framework (Gaussian mixture model + EM maximum likelihood) provides the log-normalized data and PC-level outputs on which the two-cluster bias factorization is computed) — https://github.com/MengQiuchen/scDecouple
- **tidyverse** (dplyr/tidyverse used to assemble the per-parameter bias values, d, r, and correlation coefficients into a tidy table; ggplot2 used for the regression plots)
- **R** (Implementation language for the bias computation, Pearson correlation testing, and figure generation) — https://github.com/MengQiuchen/scDecouple

## Examples

```
res <- run_scDecouple(Y, Z)
```

## Evaluation signals

- Pearson correlation of the bias term against d and against r is high and positive, consistent with the paper's linear derivation (Equations 14-15)
- bias_vs_dr_table.csv contains exactly one row per simulated (d, r) combination with no missing bias values
- Computed bias values match the product (lambda_1 - delta_1)(mu_1 - mu_2) reconstructed from cluster proportions and cluster mean expressions
- ggplot2 fitted regression lines show approximately linear trends of bias with both d and r in the saved figure

## Limitations

- The exact linear factorization is demonstrated only in the two-cluster synthetic scenario and may not extend to heterogeneous multi-cluster samples
- The Gaussian mixture model assumption might not hold in intricate systems or disease tissues, such as tumors
- The assumption of the same cell clusters between NT and perturbation groups fails when some rare cell types are completely absent from infection
- The scDecouple R package version number and the simulation setup details are not fully specified in the discussion section

## Evidence

- [abstract] Iteratively find the maximum likelihood of cell cluster proportions as well as the cellular response for each gRNA: "and then iteratively finds the maximum likelihood of cell cluster proportions as well as the cellular response for each gRNA."
- [results] two-cluster bias factorization: [evidence span withheld — anchor status: not_found]
- [intro] GMM + MLE approach: "Here, we developed scDecouple as a solution to solve this problem by employing maximum likelihood estimation. scDecouple leverages a Gaussian mixture model to depict the distribution of gene"
- [readme] run_scDecouple invocation: "## step3: decouple cellular response and infection bias
res.step3 <- scDecouple(Z.pca.mat = res.step1$Z.pca.results$x"
- [readme] preprocessing step: "## step1: preprocessing, to get log-normalized data and highly variable genes"
