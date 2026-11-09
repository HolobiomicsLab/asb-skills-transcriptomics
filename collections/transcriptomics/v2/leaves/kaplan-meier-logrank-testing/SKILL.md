---
name: kaplan-meier-logrank-testing
description: Use when TCGA bulk tumor samples have RSEM-normalized expression for
  a gene signature (e.g., the 73 TMGs) plus matched clinical OS/DSS time-status data,
  and you must test if high vs low signature expression separates survival.
when_to_use_negative:
- The goal is correlating gene-signature expression with continuous covariates (immune
  score, stromal score, tumor purity) — the article uses Spearman correlation for
  those, not KM/log-rank.
- Input is single-cell data without matched clinical survival follow-up (e.g., the
  scRNA-seq training datasets) — no OS/DSS endpoints exist to fit KM models.
- You need to annotate cells as malignant vs non-malignant — that is PreCanCell_classifier()'s
  task, not a survival analysis.
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R survival (survfit)
  role: Fits the Kaplan-Meier models for OS and DSS on median-split TMG expression
    classes; log-rank test evaluates curve separation
- name: PreCanCell
  role: 'Upstream: its ensemble k-NN (k=5) pipeline yields the 73 TMGs whose mean
    TCGA expression defines the survival strata'
  repo: https://github.com/WangX-Lab/PreCanCell
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:1b0023f2c3e9b4429dfbc7c2a5331db686141298a846818e209f048fe9f03d88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - ESTIMATE
  - Kaplan-Meier
  - PreCanCell
  - R
  - R survival (survfit)
  - Student's t-test
  - ssGSEA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# kaplan-meier-logrank-testing

## Summary

Fits Kaplan-Meier survival models with survfit() in the R survival package to test whether TCGA pan-cancer tumors stratified at the median TMG expression level into low- vs high-expression classes show separated overall survival (OS) and disease-specific survival (DSS) curves by log-rank test.

## When to use

Use when TCGA bulk tumor samples have RSEM-normalized expression for a gene signature (e.g., the 73 TMGs) plus matched clinical OS/DSS time-status data, and you must test if high vs low signature expression separates survival.

## When NOT to use

- The goal is correlating gene-signature expression with continuous covariates (immune score, stromal score, tumor purity) — the article uses Spearman correlation for those, not KM/log-rank.
- Input is single-cell data without matched clinical survival follow-up (e.g., the scRNA-seq training datasets) — no OS/DSS endpoints exist to fit KM models.
- You need to annotate cells as malignant vs non-malignant — that is PreCanCell_classifier()'s task, not a survival analysis.

## Inputs

- RSEM-normalized TCGA pan-cancer RNA-seq expression matrix (33 cancer types, log2(x+1)-transformed) from the GDC portal
- TCGA clinical data with OS and DSS time/status per sample
- List of 73 tumor marker genes (TMGs) identified from the five cancer single-cell training datasets

## Outputs

- High- and low-TMG-expression sample class assignments (median split)
- Kaplan-Meier survival curves for OS and DSS stratified by expression class
- Log-rank test P-values for OS (expected ~ 0) and DSS (expected 3.1e-15)

## How to apply

Log2(x+1)-transform the RSEM-normalized TCGA expression values downloaded from the GDC portal for the 33 cancer types, then compute per-sample mean expression of the 73 TMGs. Dichotomize samples at the median TMG expression level into low- and high-expression classes so each class holds roughly half the cohort. Fit Kaplan-Meier models with survfit() from the R survival package, one for OS and one for DSS, stratified by expression class. Evaluate curve separation with the log-rank test, expecting OS P ~ 0 and DSS P = 3.1e-15 for the high-TMG class. The rationale is that if TMGs are authentic cancer-cell markers, their bulk tumor expression should carry prognostic signal, so the reproduction is accepted only when the KM log-rank separation for high-TMG expression is recovered.

## Related tools

- **R survival (survfit)** (Fits the Kaplan-Meier models for OS and DSS on median-split TMG expression classes; log-rank test evaluates curve separation)
- **PreCanCell** (Upstream: its ensemble k-NN (k=5) pipeline yields the 73 TMGs whose mean TCGA expression defines the survival strata) — https://github.com/WangX-Lab/PreCanCell

## Examples

```
fit <- survfit(Surv(OS_time, OS_status) ~ tmg_expr_class, data = tcga); survdiff(Surv(OS_time, OS_status) ~ tmg_expr_class, data = tcga)
```

## Evaluation signals

- Reproduced log-rank P-values: OS P ~ 0 and DSS P = 3.1e-15 for high- vs low-TMG-expression classes
- Median split yields two classes of approximately equal size (each ~50% of TCGA samples)
- Expression values are log2(x+1)-transformed RSEM values and the stratification variable is the per-sample mean over all 73 TMGs
- KM curves for the high-TMG class separate from (show worse survival than) the low-TMG class for both OS and DSS endpoints

## Limitations

- Median-split dichotomization discards the continuous dose-response information in TMG expression; results depend on the chosen cut point.
- Survival associations rely on the completeness and accuracy of TCGA clinical (OS/DSS) annotation and are observational, not causal.

## Evidence

- [methods] Survival analysis workflow: "We implemented survival analyses with the function “survfit()” in the R package “survival.”"
- [other] Median split and log-rank expectation: [evidence span withheld — anchor status: not_found]
- [other] Reproduction acceptance criterion: [evidence span withheld — anchor status: not_found]
