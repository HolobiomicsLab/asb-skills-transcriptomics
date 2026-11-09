---
name: ead-vs-nonead-group-comparison-statistics
description: Use when per-sample scalar summaries (cell-type percentages, niche scores,
  LR-pair counts or scores) must be compared between non-EAD and EAD patient groups
  within a transplantation stage, mirroring the paper's one-sided Welch's test design.
when_to_use_negative:
- Input is already an aggregated comparison result or summary table — apply the test
  only to per-sample values with known EAD labels
- The analysis goal is cell-cell communication inference itself (LR-pair enrichment);
  use scCrossTalk's create_scCrossTalk/find_lrpairs workflow first, then apply this
  skill only for the non-EAD vs EAD comparison of resulting scores
- Sample sizes or group structure differ from a two-group (non-EAD vs EAD) within-stage
  design; the one-sided Welch's test as parameterized here assumes that structure
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: R
  role: Environment in which per-sample percentage computation and one-sided Welch's
    test are implemented
  repo: https://github.com/multitalk/scCrossTalk
- name: scCrossTalk
  role: Produces the LR-pair counts/scores that, like the immune niche scores, are
    compared between non-EAD and EAD patients with one-sided Welch's test
  repo: https://github.com/multitalk/scCrossTalk
- name: ggplot2
  role: Visualization of group comparisons between non-EAD and EAD patients
  repo: https://github.com/multitalk/scCrossTalk
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:44effd9e6349676f430e66a8efd03a984e5a8b2549cc471f018188999741f27a
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
  - ggplot2
  - scCrossTalk
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ead-vs-nonead-group-comparison-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical comparison of per-sample group summaries (e.g., EAD-associated immune niche scores) between non-EAD and EAD liver transplant patients using one-sided Welch's test, as used in the paper to compare LR-pair counts/scores and niche scores across cold preservation and portal reperfusion stages.

## When to use

Use when per-sample scalar summaries (cell-type percentages, niche scores, LR-pair counts or scores) must be compared between non-EAD and EAD patient groups within a transplantation stage, mirroring the paper's one-sided Welch's test design.

## When NOT to use

- Input is already an aggregated comparison result or summary table — apply the test only to per-sample values with known EAD labels
- The analysis goal is cell-cell communication inference itself (LR-pair enrichment); use scCrossTalk's create_scCrossTalk/find_lrpairs workflow first, then apply this skill only for the non-EAD vs EAD comparison of resulting scores
- Sample sizes or group structure differ from a two-group (non-EAD vs EAD) within-stage design; the one-sided Welch's test as parameterized here assumes that structure

## Inputs

- Per-sample scRNA-seq count matrices and cell type annotations for 12 liver samples (7 donors, cold perfusion and portal reperfusion stages)
- Cell-type labels for the three niche populations (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils)
- Per-sample metadata: donor ID, stage (cold preservation vs 2-hour portal reperfusion), EAD status (non-EAD vs EAD)

## Outputs

- 12-row table of per-sample immune niche scores (sum of three cell-type percentages) with group labels, e.g., ead_niche_scores.csv
- One-sided Welch's test results comparing non-EAD vs EAD within each stage (statistic, p-value)

## How to apply

First compute a per-sample scalar from the scRNA-seq data: for the immune niche score, sum the percentages of MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils relative to all annotated cells in each of the 12 liver samples (7 donors). Label each sample with donor, stage (cold preservation vs 2-hour portal reperfusion), and EAD status. Then apply a one-sided Welch's test comparing non-EAD vs EAD within each stage, as the paper used for non-EAD vs EAD comparisons of LR-pair number and score. The one-sided alternative encodes the prior hypothesis that EAD samples show higher niche/LR values, consistent with the reported pathogenic immune niche association. Report per-sample values in a table (e.g., ead_niche_scores.csv) alongside the test result per stage.

## Related tools

- **R** (Environment in which per-sample percentage computation and one-sided Welch's test are implemented) — https://github.com/multitalk/scCrossTalk
- **scCrossTalk** (Produces the LR-pair counts/scores that, like the immune niche scores, are compared between non-EAD and EAD patients with one-sided Welch's test) — https://github.com/multitalk/scCrossTalk
- **ggplot2** (Visualization of group comparisons between non-EAD and EAD patients) — https://github.com/multitalk/scCrossTalk

## Examples

```
score <- rowSums(sapply(c('MAIT','GZMB+_GZMK+_NK','S100A12+_neutrophil'), function(ct) 100*mean(celltype==ct))); wilcox.test(score ~ ead, alternative='greater')
```

## Evaluation signals

- The per-sample table contains exactly 12 rows, one per liver sample
- Each immune niche score is a sum of three cell-type percentages bounded between 0% and 300%
- EAD samples show higher niche scores than non-EAD samples at both stages (cold preservation and 2-hour portal reperfusion), consistent with the reported pathogenic immune niche association
- The test applied is a one-sided Welch's test performed separately within each stage, not pooled across stages
- Each score is decomposable into its three constituent cell-type percentages (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils)

## Limitations

- Only 12 samples from 7 donors are available, so statistical power per stage is limited and donors may contribute multiple correlated samples
- The one-sided test presupposes the direction of the EAD effect; it should not be reused where no directional prior exists
- The niche score is defined strictly from the three populations identified in this cohort (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils); generalization rests on the paper's verification in two independent cohorts

## Evidence

- [full_text] Workflow: compute percentage of each niche population per sample, sum to niche score, one-sided Welch's test per stage: [evidence span withheld — anchor status: not_found]
