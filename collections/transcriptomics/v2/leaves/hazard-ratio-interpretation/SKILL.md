---
name: hazard-ratio-interpretation
description: Use when patient-level survival metadata (vital status, days to death,
  days to follow-up) is available alongside per-gene expression sets, and you need
  to rank gene panels (e.g. IC vs DE genes) by prognostic association via hazard ratios
  and C-statistics.
when_to_use_negative:
- The cohort lacks per-patient survival endpoints (vital status and time-to-event);
  a classification or enrichment evaluation (e.g. normal/cancer AUC comparison, as
  done elsewhere in this paper) is the appropriate validation instead
- The candidate gene lists are not the fused-network IC genes or the DESeq2 DE genes
  used as the paper's comparison pair, so the 44 vs 6 and C-statistic reference values
  do not apply as expected outcomes
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Used to fit the Cox proportional-hazards models, extract hazard ratios and
    p-values, compute Concordance statistics, and write cox_survival_results.csv
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: R/Bioconductor
  role: Provides the DESeq2 baseline DE gene sets (adjusted p-value < 0.05, |log2FC|
    > 2.0) against which the Cox-based IC gene prognostic evaluation is compared
- name: Jupyter Notebook
  role: Hosts the evaluation notebooks containing the systematic and statistical evaluation
    of the extracted IC genes
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:8889314f867894b19a6f8b37027550446336e4c1e4661ec3ec9efb982a3989c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - Jupyter Notebook
  - Python
  - R/Bioconductor
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# hazard-ratio-interpretation

## Summary

Fits univariate and multivariate Cox proportional-hazards models (h(t) = h0(t) * exp(b*x)) on gene-expression predictors to test whether IC genes from fused co-expression networks are more prognostic than DESeq2-derived DE genes in TCGA cohorts (KIRC, LIHC, PRAD), comparing hazard ratios, significant-gene counts, and Concordance statistics.

## When to use

Use when patient-level survival metadata (vital status, days to death, days to follow-up) is available alongside per-gene expression sets, and you need to rank gene panels (e.g. IC vs DE genes) by prognostic association via hazard ratios and C-statistics.

## When NOT to use

- The cohort lacks per-patient survival endpoints (vital status and time-to-event); a classification or enrichment evaluation (e.g. normal/cancer AUC comparison, as done elsewhere in this paper) is the appropriate validation instead
- The candidate gene lists are not the fused-network IC genes or the DESeq2 DE genes used as the paper's comparison pair, so the 44 vs 6 and C-statistic reference values do not apply as expected outcomes

## Inputs

- TCGA gene expression matrices for KIRC, LIHC, and PRAD
- Patient clinical metadata: vital status, days to death, days to follow-up
- IC gene lists per cancer type (Extracted/IC_<tumor>.csv from the fused co-expression network pipeline)
- Baseline DE gene sets from DESeq2 (adjusted p-value < 0.05, |log2 fold change| > 2.0)

## Outputs

- Per-gene hazard-ratio table (hazard ratio exp(b) and p-value for each IC and DE gene)
- Counts of genes significantly associated with survival per gene set (IC vs DE)
- Concordance statistics for multivariate Cox models of significant IC genes vs DE gene models
- cox_survival_results.csv consolidating all of the above

## How to apply

Load the TCGA gene expression matrices and clinical metadata (vital status, days to death, days to follow-up) for KIRC, LIHC, and PRAD, plus the IC gene lists (Extracted/IC_<tumor>.csv) and the DESeq2 baseline DE gene sets (adjusted p-value < 0.05, |log2 fold change| > 2.0). Fit a univariate Cox proportional-hazards model per gene under h(t) = h0(t) * exp(b*x), extracting exp(b) (the hazard ratio) and its p-value for every IC and DE gene. Count genes significantly associated with the survival event probability in each set — the reference outcome is 44 significant IC genes vs 6 significant DE genes. Then fit a multivariate Cox model on the significant IC genes per cancer type and compute the Concordance statistic; the reference comparison is C = 0.745 vs 0.666 for KIRC and C = 0.923 vs 0.797 for PRAD, both favoring IC genes. Write hazard-ratio tables, significant-gene counts, and the C-statistic comparison to cox_survival_results.csv. Note the article does not name the specific survival-analysis software or package versions, so any standard Cox implementation (Python lifelines or R survival) is acceptable but must be documented.

## Related tools

- **Python** (Used to fit the Cox proportional-hazards models, extract hazard ratios and p-values, compute Concordance statistics, and write cox_survival_results.csv) — https://github.com/DEIB-GECO/GeneNetFusion
- **R/Bioconductor** (Provides the DESeq2 baseline DE gene sets (adjusted p-value < 0.05, |log2FC| > 2.0) against which the Cox-based IC gene prognostic evaluation is compared)
- **Jupyter Notebook** (Hosts the evaluation notebooks containing the systematic and statistical evaluation of the extracted IC genes) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb

## Evaluation signals

- Row counts of significant genes must match the reported 44 IC genes vs 6 DE genes significantly associated with survival
- Concordance statistics must reproduce C = 0.745 (IC) vs 0.666 (DE) for KIRC and C = 0.923 (IC) vs 0.797 (DE) for PRAD
- The hazard-ratio table must contain exp(b) and a p-value for every gene in both the IC and DE sets per cancer type
- cox_survival_results.csv must exist and contain the hazard-ratio table, significant-gene counts, and C-statistic comparison sections
- Cox proportionality and multivariate fit diagnostics should be checked since the article reports no survival-software package or version, leaving the implementation unverified

## Limitations

- No statistical software, packages, or versions are named for the survival analyses (Cox proportional-hazards models), so exact reproducibility of the C-statistics is not guaranteed
- No thresholds or parameters are reported in the article for declaring survival association significance, so the 44 vs 6 gene counts cannot be reproduced without an assumed cutoff
- Only KIRC and PRAD showed a higher Concordance statistic for IC genes; the LIHC comparison did not yield the reported advantage, so the IC-over-DE conclusion is cancer-type dependent

## Evidence

- [discussion] 44 of all IC genes are significantly associated with the survival event probability, a relevantly higher number than the 6 differentially expressed genes significantly associated with survival.: "44 of all IC genes are significantly associated with the survival event probability, a relevantly higher number than the 6 differentially expressed genes significantly associated with survival."
- [discussion] the multivariate Cox proportional-hazards model of the significant IC genes for each cancer type gives a better Concordance statistic: "the multivariate Cox proportional-hazards model of the significant IC genes for each cancer type gives a better Concordance statistic"
- [other] Fit a univariate Cox proportional-hazards model for each IC gene and each DE gene using the hazard function h(t) = h0(t) * exp(b*x), extracting the hazard ratio exp(b) and its p-value per gene.: [evidence span withheld — anchor status: not_found]
- [readme] contains systematic and statistic evaluation of the extracted genes (here called IC genes).: "contains systematic and statistic evaluation of the extracted genes (here called IC genes)."
- [results] Multivariate Cox models built with significant IC genes achieve higher Concordance statistics than DE gene models for KIRC and PRAD: [evidence span withheld — anchor status: not_found]
