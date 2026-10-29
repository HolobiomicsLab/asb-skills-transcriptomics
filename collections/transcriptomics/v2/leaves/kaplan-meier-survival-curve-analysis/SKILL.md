---
name: kaplan-meier-survival-curve-analysis
description: Use when per-sample viral or HERV presence calls (from viRNAtrap contigs)
  must be tested for association with TCGA overall survival by Kaplan-Meier curves
  and log-rank tests.
when_to_use_negative:
- Samples lack per-sample viral/HERV status calls, so no virus-positive/negative dichotomization
  can be defined
- The question is differential expression or mutation enrichment rather than survival
  association (use Wilcoxon or hypergeometric tests instead)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0199
tools:
- name: viRNAtrap
  role: Generates predicted viral contigs from unmapped RNAseq reads used to dichotomize
    patients into virus-positive and virus-negative survival groups
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: lifelines
  role: Python library used to fit Kaplan-Meier estimators and compute log-rank test
    p-values
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:48c64eb5bdf0ddc68d907ef152f2de87d577ea7763c05efdab84b4b97f6379b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - lifelines
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# kaplan-meier-survival-curve-analysis

## Summary

Dichotomize TCGA patients by viral-status (e.g., HERV-K/HERV-H contig-positive vs -negative from viRNAtrap, or IIV31-positive vs -negative in UCEC) and compare overall survival between groups using Kaplan-Meier estimators with a two-sided log-rank test.

## When to use

Use when per-sample viral or HERV presence calls (from viRNAtrap contigs) must be tested for association with TCGA overall survival by Kaplan-Meier curves and log-rank tests.

## When NOT to use

- Samples lack per-sample viral/HERV status calls, so no virus-positive/negative dichotomization can be defined
- The question is differential expression or mutation enrichment rather than survival association (use Wilcoxon or hypergeometric tests instead)

## Inputs

- Per-sample viRNAtrap-predicted viral contig calls (HERV-K/HERV-H-positive samples)
- TCGA clinical overall survival data per patient (e.g., Source Data file 4 for UCEC)
- Cohort/cancer-type labels (COAD, LUSC, LUAD, LIHC, UCEC)

## Outputs

- Kaplan-Meier survival curves per virus-positive and virus-negative group
- Two-sided log-rank test p-values per cancer type
- Results CSV of test statistics and p-values (e.g., fig4b_e_association_statistics.csv)

## How to apply

Load per-sample viral/HERV contig calls (e.g., HERV-K/HERV-H-positive samples derived from viRNAtrap-predicted viral contigs, or IIV31 status) together with TCGA clinical survival data for the relevant cancer cohort. Dichotomize patients into virus-positive and virus-negative groups within each cancer type. Fit Kaplan-Meier overall survival estimators for each group with lifelines and compute two-sided log-rank test p-values comparing the groups. In the HERV analysis, significant associations (log-rank p < 0.05) with lower overall survival were expected in COAD, LUSC, LUAD, and LIHC; in the UCEC analysis, the IIV31 survival comparison is reported alongside Wilcoxon tests on TMB, CNA burden, and CIBERSORT CD8+/Treg fractions. Write all test statistics and p-values to a results CSV (e.g., fig4b_e_association_statistics.csv).

## Related tools

- **viRNAtrap** (Generates predicted viral contigs from unmapped RNAseq reads used to dichotomize patients into virus-positive and virus-negative survival groups) — https://github.com/AuslanderLab/virnatrap.git
- **lifelines** (Python library used to fit Kaplan-Meier estimators and compute log-rank test p-values)

## Examples

```
from lifelines.statistics import logrank_test; results = logrank_test(T_pos, T_neg, E_pos, E_neg)
```

## Evaluation signals

- Log-rank test returns a finite p-value for each cancer type (COAD, LUSC, LUAD, LIHC)
- HERV-K/HERV-H-positive patients show significantly lower overall survival with log-rank p < 0.05 in all four cancer types
- For UCEC, the IIV31 survival test reproduces the reported Fig. 4b-e statistics consistent with deposited Source Data file 4
- All computed test statistics and p-values are written to the output CSV with finite values

## Limitations

- HERV expression is associated with poor survival but the paper does not establish causality; associations are exploratory across 14 TCGA cancer types
- viral-status dichotomization depends on viRNAtrap sensitivity, which is only guaranteed if more than five viral reads are present in a sample

## Evidence

- [results] KM/log-rank for HERV survival: [evidence span withheld — anchor status: not_found]
- [abstract] HERV survival associated finding: "disclose human endogenous viruses whose expression is associated with poor overall survival"
- [methods] KM/log-rank for IIV31 survival: [evidence span withheld — anchor status: not_found]
- [discussion] viRNAtrap sensitivity limitation: "while not compromising sensitivity if more than five viral reads are present"
