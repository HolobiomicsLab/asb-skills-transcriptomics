---
name: high-dimensional-omics-cross-model-gene-intersection
description: Use when fitting regression models with survival (or binary) outcomes
  against ~thousands to hundred-thousands of genomic covariates (e.g., 9,548 TCGA
  ovarian mRNA entries) and a robust, intersection-based candidate gene list with
  concordance estimates is needed.
when_to_use_negative:
- The covariate dimension is low (tens of variables); penalized regression with first-step
  FDR screening is designed for high-dimensional genomic data.
- The data are already pre-reduced to a small feature table without a multiple-testing
  problem, making FDR screening unnecessary.
- The goal is differential expression per gene rather than selecting genes predictive
  of a survival (Cox) or binary (logistic) outcome.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3053
tools:
- name: HDMAC
  role: Web-based interactive platform implementing FDR first-step screening plus
    Ridge/Lasso/adaptive Lasso Cox and logistic penalized regression with cross validation
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Statistical computing language in which the platform is written and all analysis
    (Cox penalized regressions, FDR screening, cross validation) is performed; R scripts
    run offline in RStudio
  repo: https://github.com/chung-R/HD-MAC
- name: Shiny
  role: R package providing the interactive web interface for running the FDR-screened
    penalized regression workflow
- name: selectiveInference
  role: R package used to compute correct post-selection p-values for Lasso and adaptive
    Lasso variable selection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:d73aabc8bb8eda792bb439691f219764ded59572e7ce9ec7cf713d52af8d51c4
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
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# high-dimensional-omics-cross-model-gene-intersection

## Summary

A penalized-regression workflow, implemented in HDMAC (R/Shiny), that first screens high-dimensional mRNA covariates by Benjamini-Hochberg FDR via univariate Cox regression, then fits Ridge, Lasso, and adaptive Lasso Cox models and intersects their selected genes to yield a robust candidate survival-gene list for TCGA ovarian cancer data.

## When to use

Use when fitting regression models with survival (or binary) outcomes against ~thousands to hundred-thousands of genomic covariates (e.g., 9,548 TCGA ovarian mRNA entries) and a robust, intersection-based candidate gene list with concordance estimates is needed.

## When NOT to use

- The covariate dimension is low (tens of variables); penalized regression with first-step FDR screening is designed for high-dimensional genomic data.
- The data are already pre-reduced to a small feature table without a multiple-testing problem, making FDR screening unnecessary.
- The goal is differential expression per gene rather than selecting genes predictive of a survival (Cox) or binary (logistic) outcome.

## Inputs

- TCGA ovarian cancer mRNA expression matrix (9,548 entries pre-screened for abnormal Z-score expression in ≥2% of 316 patients)
- Patient overall survival times with censoring indicators
- FDR threshold (default 0.05) and cross-validation fold number (default 5)

## Outputs

- Intersection gene list of the Ridge/Lasso/adaptive-Lasso Cox selections (expected 6 genes: ASAP3, C10ORF113, TIGAR, KIAA0100, REPL4B, ZFHX4), e.g., fdr_selected_mrna_ovarian.csv
- 5-fold cross-validation concordance indices per model (~0.538, 0.538, 0.540)
- FDR-screened variable set from the first-step screening

## How to apply

Load the pre-screened molecular dataset — for TCGA ovarian cancer this is 9,548 mRNA entries whose abnormal expression (Z score above 2 or below −2) is observed in ≥2% of the 316 patients — together with overall survival times and censoring indicators. Apply FDR control as the optional first-step screening (default threshold 0.05) to remove unrelated variables, which is recommended when covariates reach the magnitude of a hundred thousand where penalized regression alone falls short. Fit Ridge, Lasso, and adaptive Lasso Cox proportional hazards regressions on the FDR-passed genes, selecting optimal tuning parameters via 5-fold cross validation (the default fold number). Intersect the gene sets selected by the three penalized Cox models to obtain the common candidate list — for the ovarian survival analysis this yields ASAP3, C10ORF113, TIGAR, KIAA0100, REPL4B, and ZFHX4 — and report each model's cross-validated concordance index (expected ~0.538–0.540) to judge prediction power.

## Related tools

- **HDMAC** (Web-based interactive platform implementing FDR first-step screening plus Ridge/Lasso/adaptive Lasso Cox and logistic penalized regression with cross validation) — https://github.com/chung-R/HD-MAC
- **R** (Statistical computing language in which the platform is written and all analysis (Cox penalized regressions, FDR screening, cross validation) is performed; R scripts run offline in RStudio) — https://github.com/chung-R/HD-MAC
- **Shiny** (R package providing the interactive web interface for running the FDR-screened penalized regression workflow)
- **selectiveInference** (R package used to compute correct post-selection p-values for Lasso and adaptive Lasso variable selection)

## Evaluation signals

- fdr_selected_mrna_ovarian.csv exists in CSV format and contains exactly 6 rows.
- The gene entries match the six Table 2 genes: ASAP3, C10ORF113, TIGAR, KIAA0100, REPL4B, ZFHX4.
- The intersection equals the overlap of the three individual penalized-model gene selections (Ridge ∩ Lasso ∩ adaptive Lasso).
- 5-fold cross-validation concordance indices fall near the reported values 0.538, 0.538, and 0.540.
- Genes removed by first-step FDR screening at threshold 0.05 are consistent with the Benjamini-Hochberg-adjusted univariate Cox p-values.

## Limitations

- FDR screening is recommended when variables reach the magnitude of a hundred thousand because penalized regression models fall short at that scale; without screening, very high dimensions can break the models.
- The identified genes (e.g., ASAP3) are a preliminary list requiring further in-depth study; HDMAC provides candidate discovery, not validated biomarkers.
- Penalized Cox selection alone can yield unstable gene lists, which is why the three-model intersection is used for robustness.

## Evidence

- [methods] FDR as optional first-step screening: "the method to control the FDR is provided as an optional first-step screening method"
- [methods] Abnormal expression Z-score cutoff: "A Z score above 2 or below −2 was considered abnormal"
- [methods] Default cross-validation folds: "The default fold number is 5, and cross validation method will not be performed if 1 is chosen"
- [discussion] When FDR screening is needed: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
- [abstract] Cross validation for prediction power: "Cross validation is provided as the method to estimate the prediction power of each regression model."
