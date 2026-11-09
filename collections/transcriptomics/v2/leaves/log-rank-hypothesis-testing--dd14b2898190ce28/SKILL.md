---
name: log-rank-hypothesis-testing
description: Use when you have per-sample viral or HERV positivity calls (e.g., from
  viRNAtrap contigs) paired with TCGA clinical overall survival times, and the goal
  is to test survival differences between virus-positive and virus-negative patient
  groups.
when_to_use_negative:
- The endpoint is a continuous non-survival variable (TMB, chromosome-level CNA burden,
  CIBERSORT CD8+/Treg proportions) — use Wilcoxon rank-sum tests instead
- The question is enrichment of virus-positive samples among Trichomonas vaginalis-positive
  or TP53/KRAS/ARID1A/PTEN-mutant samples — use Fisher's exact test or scipy.stats.hypergeom,
  not log-rank
- Input FASTQ reads have not yet been processed by viRNAtrap to produce per-sample
  viral contig calls, so positivity status cannot be defined
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: viRNAtrap
  role: Upstream alignment-free deep learning pipeline that generates the per-sample
    predicted viral contigs used to define HERV/virus-positive vs -negative survival
    groups
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: lifelines
  role: Python library used to fit Kaplan-Meier overall survival estimators per group
    and compute two-sided log-rank test p-values within each cancer type
- name: scipy.stats.hypergeom
  role: Companion test in the same workflow for hypergeometric enrichment of HERV-positive
    samples among driver-gene (TP53, KRAS, ARID1A, PTEN) mutant samples
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:68abd68b979d8057b80ed9dcdde0429f9b5249c5a9eebf8b7c3a565150d503f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - lifelines
  - scipy.stats.hypergeom
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# log-rank-hypothesis-testing

## Summary

Hypothesis-testing skill for comparing overall survival between virus/HERV-positive and virus-negative tumor sample groups using Kaplan-Meier estimators and the log-rank test, as applied in the viRNAtrap TCGA viral-expression study to link HERV-K/HERV-H and divergent virus presence to poor survival.

## When to use

Use when you have per-sample viral or HERV positivity calls (e.g., from viRNAtrap contigs) paired with TCGA clinical overall survival times, and the goal is to test survival differences between virus-positive and virus-negative patient groups.

## When NOT to use

- The endpoint is a continuous non-survival variable (TMB, chromosome-level CNA burden, CIBERSORT CD8+/Treg proportions) — use Wilcoxon rank-sum tests instead
- The question is enrichment of virus-positive samples among Trichomonas vaginalis-positive or TP53/KRAS/ARID1A/PTEN-mutant samples — use Fisher's exact test or scipy.stats.hypergeom, not log-rank
- Input FASTQ reads have not yet been processed by viRNAtrap to produce per-sample viral contig calls, so positivity status cannot be defined

## Inputs

- Per-sample HERV-K/HERV-H or divergent virus (e.g., IIV31) positivity calls derived from viRNAtrap-predicted viral contigs
- TCGA clinical survival data (overall survival times and censoring status) for the relevant cohorts (COAD, LUSC, LUAD, LIHC, UCEC)
- Deposited Source Data files with per-sample virus status, survival, TMB, CNA, and CIBERSORT fractions (e.g., Source Data file 4)

## Outputs

- Log-rank test p-values per cancer type comparing virus-positive vs virus-negative overall survival
- Kaplan-Meier survival estimators (fit objects/curves) per group
- A single CSV results table (e.g., log-rank p-values per cohort, association statistics per gene/virus)

## How to apply

Dichotomize patients in each TCGA cohort (e.g., COAD, LUSC, LUAD, LIHC for HERV-K/HERV-H; UCEC for IIV31) into HERV/virus-positive and -negative groups based on the presence of predicted viral contigs in the matching sample. Fit Kaplan-Meier overall survival estimators per group with the lifelines library and compare groups with a two-sided log-rank test within each cancer type. Treat p < 0.05 as the significance threshold for lower overall survival in the HERV-positive group, as required to reproduce the paper's reported associations. Keep survival testing separate from other endpoint tests: continuous variables such as tumor mutation burden, chromosome-level CNA burden, and CIBERSORT CD8+ T-cell/Treg fractions use Wilcoxon rank-sum tests, and enrichment of virus-positive samples among Trichomonas vaginalis-positive or TP53/KRAS/ARID1A/PTEN-mutant samples uses Fisher's exact or hypergeometric tests. Write per-cohort log-rank p-values into a single results CSV alongside the other test statistics.

## Related tools

- **viRNAtrap** (Upstream alignment-free deep learning pipeline that generates the per-sample predicted viral contigs used to define HERV/virus-positive vs -negative survival groups) — https://github.com/AuslanderLab/virnatrap.git
- **lifelines** (Python library used to fit Kaplan-Meier overall survival estimators per group and compute two-sided log-rank test p-values within each cancer type)
- **scipy.stats.hypergeom** (Companion test in the same workflow for hypergeometric enrichment of HERV-positive samples among driver-gene (TP53, KRAS, ARID1A, PTEN) mutant samples)

## Examples

```
from lifelines.statistics import logrank_test; lr = logrank_test(durations_pos, durations_neg, events_pos, events_neg); print(lr.p_value)
```

## Evaluation signals

- HERV-K/HERV-H-positive patients must show significantly lower overall survival (log-rank p < 0.05) in COAD, LUSC, LUAD, and LIHC, reproducing the paper's reported associations
- The same workflow applied to UCEC must return finite log-rank statistics consistent with the reported Fig. 4b-e results
- Per-cohort group sizes should be non-empty: dichotomization must yield both virus-positive and virus-negative samples before fitting Kaplan-Meier estimators
- Companion enrichment tests must reproduce reported values, e.g., Fisher exact p = 1.4e-8 for TV-positive UCEC enrichment among IIV31-positive samples
- Output CSV should contain one log-rank p-value per cancer type plus hypergeometric enrichment p-values per gene, with no missing/NaN entries for the validated cohorts

## Limitations

- Significance is a between-group association, not evidence that viral expression drives poor survival; the paper reports only that HERV expression is associated with poor survival rates
- viRNAtrap sensitivity is only guaranteed when more than five viral reads are present, so virus-negative groups may contain undetected positives that bias survival comparisons
- Sample-level contamination (e.g., phiX phage and other laboratory contaminants) cannot be fully modeled and may misclassify samples as viral-positive

## Evidence

- [abstract] disclose human endogenous viruses whose expression is associated with poor overall survival: "disclose human endogenous viruses whose expression is associated with poor overall survival"
- [intro] analyze HERV expression across cancers for survival association: "we curate a database of potentially functional human endogenous retroviruses (HERVs) and analyze expression patterns of different HERVs across human cancers to find that HERV expression is associated"
- [discussion] viRNAtrap applied across 14 TCGA cancer tissues with genomic and survival correlates: "We employ viRNAtrap for exploratory data analysis and characterize viruses that are expressed across 14 cancer tissues from TCGA and analyze their genomic and survival correlates"
- [readme] README description of model-based viral read prediction feeding per-sample positivity: "Use the Tensorflow model to predict which of the unmapped reads are viral; this is performed by the included software."
