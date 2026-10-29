---
name: tcga-clinical-survival-analysis
description: Use when TCGA RNA-seq expression (RSEM-normalized, log2(x+1)) plus clinical
  OS/DSS data are available and you need to test whether a gene-set expression signature
  stratifies patient survival.
when_to_use_negative:
- Input is single-cell data without TCGA clinical survival annotations — PreCanCell
  cell classification, not survival testing, is the right skill
- The gene signature is defined per-patient rather than as a per-sample mean expression
  score, so a median split is not meaningful
- Only normal (non-tumor) samples are available, since KM/log-rank stratification
  requires tumor cohorts with OS/DSS follow-up
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3577
tools:
- name: R survival
  role: Provides survfit() for fitting Kaplan-Meier OS/DSS models on the median-split
    TMG classes
- name: Kaplan-Meier
  role: Survival curve estimator compared between low- and high-TMG expression classes
- name: PreCanCell
  role: Source of the 73 TMGs whose TCGA expression signature drives the survival
    stratification
  repo: https://github.com/WangX-Lab/PreCanCell
- name: R
  role: Implementation environment for the survival analysis pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:9179af73b64905cae3dcb97cd40197e39ce6c08e78a698cae9152bdb0b49bb59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - ESTIMATE
  - Kaplan-Meier
  - PreCanCell
  - R
  - R survival
  - Student's t-test
  - ssGSEA
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tcga-clinical-survival-analysis

## Summary

Reproduces the TCGA pan-cancer survival analysis from PreCanCell: tumor samples are split at the median expression level of the 73 tumor marker genes (TMGs) into low- and high-expression classes, Kaplan-Meier curves for overall survival (OS) and disease-specific survival (DSS) are fit, and separation is tested by log-rank.

## When to use

Use when TCGA RNA-seq expression (RSEM-normalized, log2(x+1)) plus clinical OS/DSS data are available and you need to test whether a gene-set expression signature stratifies patient survival.

## When NOT to use

- Input is single-cell data without TCGA clinical survival annotations — PreCanCell cell classification, not survival testing, is the right skill
- The gene signature is defined per-patient rather than as a per-sample mean expression score, so a median split is not meaningful
- Only normal (non-tumor) samples are available, since KM/log-rank stratification requires tumor cohorts with OS/DSS follow-up

## Inputs

- RSEM-normalized RNA-seq expression matrix for 33 TCGA cancer types (from GDC portal)
- TCGA clinical data with overall survival (OS) and disease-specific survival (DSS) time and event fields
- Gene signature list (e.g., the 73 TMGs identified from the five cancer single-cell datasets)

## Outputs

- Kaplan-Meier survival curves for low- vs high-TMG-expression classes (OS and DSS)
- Log-rank test P-values for OS and DSS separation
- Median-split sample classification (low vs high TMG expression)

## How to apply

Download RSEM-normalized RNA-seq and clinical data for the 33 TCGA cancer types from the GDC portal and log2(x+1)-transform expression values. Compute per-sample mean expression of the 73 TMGs (or 186 NMGs for the negative-control signature), then split TCGA samples at the median TMG expression level into low- and high-expression classes. Fit Kaplan-Meier models with survfit() from the R survival package and evaluate OS and DSS separation between the two classes with the log-rank test. The reproduction succeeds when high-TMG expression shows significant log-rank separation (expected OS P ~ 0 and DSS P = 3.1e-15).

## Related tools

- **R survival** (Provides survfit() for fitting Kaplan-Meier OS/DSS models on the median-split TMG classes)
- **Kaplan-Meier** (Survival curve estimator compared between low- and high-TMG expression classes)
- **PreCanCell** (Source of the 73 TMGs whose TCGA expression signature drives the survival stratification) — https://github.com/WangX-Lab/PreCanCell
- **R** (Implementation environment for the survival analysis pipeline)

## Examples

```
library(survival); expr <- log2(rsem_expr + 1); grp <- ifelse(colMeans(expr[tmg_list, ]) > median(colMeans(expr[tmg_list, ])), 'high', 'low'); km <- survfit(Surv(OS_time, OS_event) ~ grp, data = clin); survdiff(Surv(OS_time, OS_event) ~ grp, data = clin)
```

## Evaluation signals

- High-TMG-expression class shows significant log-rank separation with OS P ~ 0
- DSS log-rank P = 3.1e-15 is recovered
- Per-sample TMG mean expression computed from log2(x+1)-transformed RSEM values and split exactly at the median produces two equal-sized classes
- Survival analysis is implemented via survfit() from the R survival package, matching the published workflow

## Limitations

- The median split is a binary stratification; it does not capture dose-response or nonlinear expression–survival relationships
- Clinical covariates (stage, age) are not adjusted in the published log-rank analysis, so confounding is not controlled
- TMG signatures come from five cancer-type training datasets, and the reported training-set performance may be overestimated due to feature-selection leakage

## Evidence

- [methods] Median split and KM workflow: "We implemented survival analyses with the function “survfit()” in the R package “survival.”"
- [results] Expected survival results: "TMGs displayed significantly higher expression levels in TCGA pan-cancer (P = 2.22 × 10−39) and in 25 of the 30 individual cancer types with normal sample’s size not less than five (FDR < 0.05)"
- [methods] TCGA data acquisition: "we downloaded RNA-seq gene expression profiling (RSEM normalized) and clinical data for 33 TCGA cancer types"
- [results] TMG definition: "The DEGs included 73 TMGs and 186 NMGs."
