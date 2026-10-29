---
name: tcga-clinical-survival-data-analysis
description: Use when you have per-sample binary exposure labels (e.g., viral/HERV
  contig presence) for TCGA tumor samples and clinical overall survival data, and
  need to test whether the marker is associated with survival in specific cancer cohorts.
when_to_use_negative:
- Exposure labels are continuous rather than a per-sample binary presence/absence
  call — dichotomization into HERV-positive/negative groups is the assumed design
- The analysis goal is de novo viral detection or contig assembly from RNAseq reads
  (that is viRNAtrap's role, upstream of this skill)
- Cancer types outside the cohorts with adequate survival follow-up, or clinical endpoints
  other than overall survival, without re-validating the survival model assumptions
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: viRNAtrap
  role: Upstream generator of the predicted viral/HERV contigs whose per-sample presence
    defines the HERV-positive vs HERV-negative patient groups used for survival testing
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: keras
  role: Deep learning backend underlying viRNAtrap's viral-read discrimination model
    that produced the contig calls feeding this survival analysis
  repo: https://github.com/fchollet/keras
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:7dfebbacb9a3c2cbc2d8d02c05eaabcf7a319639911005f6e60b1f54971d577c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - keras
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# tcga-clinical-survival-data-analysis

## Summary

Associates per-sample molecular markers (here, HERV-K/HERV-H contig presence from viRNAtrap) with TCGA overall survival by dichotomizing patients into marker-positive and marker-negative groups, fitting Kaplan-Meier estimators, and testing differences with log-rank tests, plus hypergeometric enrichment of marker-positive samples among somatically mutated driver genes.

## When to use

Use when you have per-sample binary exposure labels (e.g., viral/HERV contig presence) for TCGA tumor samples and clinical overall survival data, and need to test whether the marker is associated with survival in specific cancer cohorts.

## When NOT to use

- Exposure labels are continuous rather than a per-sample binary presence/absence call — dichotomization into HERV-positive/negative groups is the assumed design
- The analysis goal is de novo viral detection or contig assembly from RNAseq reads (that is viRNAtrap's role, upstream of this skill)
- Cancer types outside the cohorts with adequate survival follow-up, or clinical endpoints other than overall survival, without re-validating the survival model assumptions

## Inputs

- Per-sample HERV-K/HERV-H contig presence/absence calls (from viRNAtrap-predicted viral contigs)
- TCGA clinical overall survival data for COAD, LUSC, LUAD, and LIHC cohorts
- TCGA somatic mutation calls (mutation/wild-type status per sample for genes such as TP53, KRAS, ARID1A, PTEN)

## Outputs

- Kaplan-Meier overall survival curves per marker-positive and marker-negative group per cancer type
- Log-rank test p-values comparing HERV-positive vs HERV-negative patients per cancer type
- Hypergeometric enrichment p-values of HERV-positive samples among driver-gene mutant samples
- A single CSV results table containing per-cancer-type log-rank p-values and per-gene hypergeometric p-values

## How to apply

Load the per-sample HERV contig calls (HERV-K/HERV-H-positive samples derived from viRNAtrap-predicted viral contigs) and the TCGA clinical survival data for COAD, LUSC, LUAD, and LIHC. Dichotomize patients within each cancer type into HERV-positive versus HERV-negative groups based on presence of HERV-K/HERV-H contigs in the corresponding sample. Fit Kaplan-Meier overall survival estimators per group using lifelines and compute log-rank test p-values comparing the two groups within each cancer type, treating log-rank p < 0.05 as significant. Optionally, load TCGA somatic mutation calls, label each sample as mutated or wild-type for genes such as TP53, KRAS, ARID1A, and PTEN, and compute scipy.stats.hypergeom tests for enrichment of HERV-positive samples among mutant samples. Write per-cancer-type log-rank p-values and per-gene hypergeometric p-values into a single CSV results table.

## Related tools

- **viRNAtrap** (Upstream generator of the predicted viral/HERV contigs whose per-sample presence defines the HERV-positive vs HERV-negative patient groups used for survival testing) — https://github.com/AuslanderLab/virnatrap.git
- **keras** (Deep learning backend underlying viRNAtrap's viral-read discrimination model that produced the contig calls feeding this survival analysis) — https://github.com/fchollet/keras

## Examples

```
from lifelines import KaplanMeierFitter; kmf = KaplanMeierFitter(); kmf.fit(T_herv_pos, E_herv_pos, label='HERV+'); kmf.plot(); from lifelines.statistics import logrank_test; p = logrank_test(T_pos, T_neg, E_pos, E_neg).p_value
```

## Evaluation signals

- HERV-K/HERV-H-positive patients show significantly lower overall survival (log-rank p < 0.05) in COAD, LUSC, LUAD, and LIHC
- HERV-positive group shows significant hypergeometric enrichment among TP53, KRAS, ARID1A, and PTEN mutant samples, reproducing the paper's reported associations
- Kaplan-Meier curves are fitted per dichotomized group within each cancer type, and the CSV results table contains one log-rank p-value per cancer type and one hypergeometric p-value per gene
- Group assignment is a one-to-one match between contig-call files and TCGA sample identifiers, with no samples dropped or duplicated during the join

## Limitations

- HERV expression is associated with poor survival rates, but the associations are correlational from TCGA cohort data; no causal claim follows from the log-rank tests
- Results depend on the quality of upstream viRNAtrap contig calls and on reads unaligned to hg19 and phiX; misclassification of samples as HERV-positive/negative propagates into survival statistics

## Evidence

- [abstract] dichotomize patients and test survival: "disclose human endogenous viruses whose expression is associated with poor overall survival"
- [discussion] HERV expression strictly associated with poor survival: "we found that the expression of human endogenous viruses is strictly associated with poor survival rates"
- [intro] TCGA as principal survival data source: "The Cancer Genome Atlas (TCGA) has become a principal resource for the identification of viral sequences in cancer tissues"
- [readme] viRNAtrap produces the contigs underlying group labels: "viRNAtrap is a package to generate predicted viral contigs from unmapped RNAseq reads"
- [intro] HERV expression analysis goal: "we curate a database of potentially functional human endogenous retroviruses (HERVs) and analyze expression patterns of different HERVs across human cancers to find that HERV expression is associated"
