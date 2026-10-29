---
name: fdr-multiple-testing-control-survival-genomics
description: Use when a regression analysis with survival or binary outcomes has high-dimensional
  genetic covariates (on the order of thousands of variables, e.g. ~9,548 mRNA entries)
  and the multiple-testing problem must be addressed before Ridge/Lasso/adaptive Lasso
  penalized regression.
when_to_use_negative:
- The covariate dimension is small (low hundreds) and standard univariate testing
  with the multiple-testing problem under control suffices — HDMAC's first-step screening
  is designed for large-volume genomic data.
- The input is not regression-ready (e.g., raw reads or expression matrices without
  per-gene abnormality Z scores or survival endpoints) — preliminary screening by
  expression frequency and survival/censoring data must be assembled first.
- 'The goal is post-selection inference alone: correct p-values for Lasso/adaptive
  Lasso are computed separately via the selectiveInference package and are not provided
  by the FDR screening step itself.'
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: HDMAC
  role: Web-based platform implementing the FDR first-step screening option followed
    by Ridge, Lasso, and adaptive Lasso Cox regression with cross-validation
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Language in which the platform and all statistical analysis (including FDR
    screening and Cox penalized regression) were written and performed
  repo: https://github.com/chung-R/HD-MAC
- name: selectiveInference
  role: R package used to compute correct post-selection p-values after Lasso/adaptive
    Lasso variable selection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:431e13c4997793b2e1d5e6d8f91763d5b1708cda6ff49d8e671d22f786780f0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - selectiveInference
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# fdr-multiple-testing-control-survival-genomics

## Summary

An optional first-step screening procedure in the HDMAC workflow that applies Benjamini-Hochberg FDR control to univariate Cox regression p-values so that only survival-associated genes (e.g., the 9,548 pre-screened TCGA ovarian cancer mRNA entries narrowed further by FDR < 0.05) advance to penalized regression.

## When to use

Use when a regression analysis with survival or binary outcomes has high-dimensional genetic covariates (on the order of thousands of variables, e.g. ~9,548 mRNA entries) and the multiple-testing problem must be addressed before Ridge/Lasso/adaptive Lasso penalized regression.

## When NOT to use

- The covariate dimension is small (low hundreds) and standard univariate testing with the multiple-testing problem under control suffices — HDMAC's first-step screening is designed for large-volume genomic data.
- The input is not regression-ready (e.g., raw reads or expression matrices without per-gene abnormality Z scores or survival endpoints) — preliminary screening by expression frequency and survival/censoring data must be assembled first.
- The goal is post-selection inference alone: correct p-values for Lasso/adaptive Lasso are computed separately via the selectiveInference package and are not provided by the FDR screening step itself.

## Inputs

- High-dimensional genetic covariate matrix (e.g., TCGA ovarian cancer mRNA expression entries pre-screened to 9,548 genes with |Z| > 2 in ≥2% of the 316 patients)
- Patient overall survival times with censoring indicators
- Optional clinical covariates (e.g., gender, age)

## Outputs

- List of FDR-passed gene candidates (Benjamini-Hochberg adjusted p-values below the 0.05 threshold)
- Downstream intersection gene list across Ridge/Lasso/adaptive Lasso Cox models (e.g., fdr_selected_mrna_ovarian.csv with 6 entries)
- Cross-validation concordance indices for each penalized model (expected 0.538, 0.538, 0.540 for the ovarian cancer case)

## How to apply

Fit a univariate Cox regression on each gene using the patients' overall survival times and censoring indicators, then apply the Benjamini-Hochberg FDR procedure to the resulting p-values as an optional first-step screening with the default FDR threshold of 0.05. Prior to this, restrict inputs to entries passing the paper's preliminary frequency screen — mRNA entries whose abnormal expression (|Z| > 2) was found in 2% or more of the patients. Only FDR-passed genes enter the subsequent Ridge, Lasso, and adaptive Lasso Cox models, whose tuning parameters are chosen by cross-validation (default fold number 5). The rationale is that penalized regression models fall short at very high variable counts, so FDR screening removes unrelated variables first; survivors are then intersected across the three penalized models (in the ovarian cancer case yielding 6 genes: ASAP3, C10ORF113, TIGAR, KIAA0100, REPL4B, ZFHX4).

## Related tools

- **HDMAC** (Web-based platform implementing the FDR first-step screening option followed by Ridge, Lasso, and adaptive Lasso Cox regression with cross-validation) — https://github.com/chung-R/HD-MAC
- **R** (Language in which the platform and all statistical analysis (including FDR screening and Cox penalized regression) were written and performed) — https://github.com/chung-R/HD-MAC
- **selectiveInference** (R package used to compute correct post-selection p-values after Lasso/adaptive Lasso variable selection)

## Examples

```
pvals_adj <- p.adjust(univariate_cox_pvalues, method = "BH"); fdr_selected_genes <- names(pvals_adj)[pvals_adj < 0.05]
```

## Evaluation signals

- The FDR-passed variable count is substantially smaller than the input dimension (9,548 mRNA entries screened down before penalized fitting).
- The downstream intersection CSV (fdr_selected_mrna_ovarian.csv) exists, is valid CSV, has exactly 6 rows, and its gene entries match ASAP3, C10ORF113, TIGAR, KIAA0100, REPL4B, ZFHX4 from Table 2.
- 5-fold cross-validation concordance indices for the three FDR-screened models fall at approximately 0.538 (Ridge), 0.538 (Lasso), and 0.540 (adaptive Lasso).
- Default parameters honored: FDR threshold 0.05 and default cross-validation fold number of 5.
- All genes entering the penalized stage satisfy the preliminary screen of abnormal expression (|Z| > 2) in at least 2% of patients.

## Limitations

- Genes identified are preliminary and require further in-depth study; they provide rational directions rather than definitive findings.
- Penalized regression models fall short at variables on the magnitude of a hundred thousand — FDR screening is recommended at that scale but itself does not guarantee discovery.
- Correct p-values for Lasso/adaptive Lasso are generally unavailable in common statistical software and must be computed via specialized methods (selectiveInference).

## Evidence

- [abstract] Choice of a first-step screening is provided to address the multiple-comparison issue that often arises with large-volume genomic data.: "Choice of a first-step screening is provided to address the multiple-comparison issue that often arises with large-volume genomic data."
- [methods] the method to control the FDR is provided as an optional first-step screening method: "the method to control the FDR is provided as an optional first-step screening method"
- [methods] a preliminary screening was performed so that only the genes whose mutations were found in 1% or more and the mRNA entries whose abnormal expression was found in 2% or more of the patients were: "a preliminary screening was performed so that only the genes whose mutations were found in 1% or more and the mRNA entries whose abnormal expression was found in 2% or more of the patients were"
- [methods] A Z score above 2 or below −2 was considered abnormal: "A Z score above 2 or below −2 was considered abnormal"
- [discussion] Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
- [methods] The default fold number is 5, and cross validation method will not be performed if 1 is chosen: "The default fold number is 5, and cross validation method will not be performed if 1 is chosen"
