---
name: ead-group-statistical-comparison
description: Use when validating a single-cell-defined cell-population signature in
  an independent bulk RNA-seq cohort with a binary phenotype group (e.g., EAD vs non-EAD),
  requiring per-sample scoring and group-comparison statistics.
when_to_use_negative:
- Input is already a per-sample feature/signature score table with no raw counts to
  normalize
- The cohort is single-cell data rather than bulk RNA-seq — use the single-cell niche
  identification workflow instead
- Groups have very small and equal variances where a standard t-test rather than Welch's
  correction is already appropriate and explicitly specified by the study design
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0625
tools:
- name: R
  role: Environment for GEO data retrieval, median-1000 normalization, log transformation,
    niche signature scoring, and Welch's t-test
  repo: https://github.com/multitalk/scCrossTalk
- name: Seurat
  role: Compatible single-cell framework from which the niche cell-population marker
    genes are derived
  repo: https://github.com/multitalk/scCrossTalk
- name: ggplot2
  role: Rendering the EAD vs non-EAD niche-score comparison boxplot
  repo: https://github.com/multitalk/scCrossTalk
- name: scCrossTalk
  role: Cell-cell communication method whose one-sided Welch's test group comparisons
    (non-EAD vs EAD LR-pair number and score) establish the paper's statistical-comparison
    paradigm
  repo: https://github.com/multitalk/scCrossTalk
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:527b13a5dc856a474a807b9bc67fd6563a777c9ebbdb73f239ac27494904ef24
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/multitalk/scCrossTalk
  repo_url: https://github.com/multitalk/scCrossTalk
  tools:
  - R
  - Seurat
  - ggplot2
  - scCrossTalk
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ead-group-statistical-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistically validates whether the EAD-associated immune niche (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) identified in the single-cell landscape distinguishes EAD from non-EAD samples, by comparing per-sample niche signature scores with Welch's t-test in the independent GSE23649 bulk RNA-seq DBD cohort (8 EAD vs 8 non-EAD).

## When to use

Use when validating a single-cell-defined cell-population signature in an independent bulk RNA-seq cohort with a binary phenotype group (e.g., EAD vs non-EAD), requiring per-sample scoring and group-comparison statistics.

## When NOT to use

- Input is already a per-sample feature/signature score table with no raw counts to normalize
- The cohort is single-cell data rather than bulk RNA-seq — use the single-cell niche identification workflow instead
- Groups have very small and equal variances where a standard t-test rather than Welch's correction is already appropriate and explicitly specified by the study design

## Inputs

- GSE23649 bulk RNA-seq raw count matrix (16 DBD liver transplant samples: 8 EAD, 8 non-EAD)
- Sample metadata with EAD status and sampling stage (Sample A pre-cold-perfusion, Sample B 2h post-portal-reperfusion)
- Marker gene lists for MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils from the single-cell landscape

## Outputs

- niche_ead_validation.csv containing per-sample niche signature scores and Welch's t-test statistic and p-value (16 rows)
- EAD vs non-EAD niche-score comparison boxplot rendered with ggplot2

## How to apply

Retrieve the GSE23649 bulk RNA-seq count matrix and metadata (8 DBD EAD vs 8 DBD non-EAD; Sample A pre-cold-perfusion, Sample B 2h post-portal-reperfusion) from GEO using R. Normalize by scaling each sample to a median value of 1000 followed by log transformation. Compute a per-sample niche signature score using the marker genes of the niche cell populations (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) defined in the single-cell landscape of 75,231 cells. Compare scores between EAD and non-EAD with Welch's t-test (the same one-sided Welch's test framework the paper used to compare LR-pair number and score between non-EAD and EAD patients), recording the test statistic and p-value. Validation succeeds if the 16-sample output table is complete and Welch's test shows a significant niche-score difference with EAD samples scoring higher, matching the reported direction.

## Related tools

- **R** (Environment for GEO data retrieval, median-1000 normalization, log transformation, niche signature scoring, and Welch's t-test) — https://github.com/multitalk/scCrossTalk
- **Seurat** (Compatible single-cell framework from which the niche cell-population marker genes are derived) — https://github.com/multitalk/scCrossTalk
- **ggplot2** (Rendering the EAD vs non-EAD niche-score comparison boxplot) — https://github.com/multitalk/scCrossTalk
- **scCrossTalk** (Cell-cell communication method whose one-sided Welch's test group comparisons (non-EAD vs EAD LR-pair number and score) establish the paper's statistical-comparison paradigm) — https://github.com/multitalk/scCrossTalk

## Examples

```
scores <- data.frame(sample, ead, niche_score); wt <- t.test(niche_score ~ ead, data = scores, alternative = "greater"); write.csv(scores, "niche_ead_validation.csv")
```

## Evaluation signals

- The output table niche_ead_validation.csv contains exactly 16 per-sample rows with no missing values
- Welch's t-test p-value indicates a significant niche-score difference between EAD and non-EAD groups
- The direction of the difference matches the published finding: EAD samples score higher on the niche signature
- Normalization yields each sample scaled to a median value of 1000 before log transformation
- The reported test statistic and p-value are reproducible from the stored per-sample scores

## Limitations

- Validation relies on a small 16-sample DBD cohort (8 vs 8), limiting statistical power
- The niche signature is defined from the single-cell landscape of 75,231 cells from 12 liver samples of 7 donors; marker-gene transfer to bulk data may dilute cell-type-specific signal
- No changelog found for the source repository, so tooling version drift cannot be tracked

## Evidence

- [abstract] Comparison of the 75 231 cells of non-EAD and EAD patients revealed an EAD-associated immune niche comprising mucosal-associated invariant T cells, granzyme B+ (GZMB+) granzyme K (GZMK+) natural: "Comparison of the 75 231 cells of non-EAD and EAD patients revealed an EAD-associated immune niche comprising mucosal-associated invariant T cells, granzyme B+ (GZMB+) granzyme K (GZMK+) natural"
- [abstract] we verified this immune niche and its association with EAD occurrence in two independent cohorts: "we verified this immune niche and its association with EAD occurrence in two independent cohorts"
- [readme] Comparison of the number and score of LR pairs between non-EAD and EAD patients with one-sided Welch¡¯s test.: "Comparison of the number and score of LR pairs between non-EAD and EAD patients with one-sided Welch's test."
