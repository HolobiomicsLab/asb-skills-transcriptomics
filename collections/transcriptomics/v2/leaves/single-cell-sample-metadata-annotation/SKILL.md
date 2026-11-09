---
name: single-cell-sample-metadata-annotation
description: Use when scRNA-seq samples already have cell-type annotations and you
  must attach per-sample quantitative metadata (cell-type percentages, composite niche
  scores, biological labels) for downstream group testing.
when_to_use_negative:
- The goal is inferring ligand-receptor cell-cell communication (e.g. enriching CellTalkDB
  LR pairs with scCrossTalk's Z-test) rather than scoring per-sample cell composition
- Single-cell data lack cell type annotations, so the three niche populations (MAIT,
  GZMB+ GZMK+ NK, S100A12+ neutrophils) cannot be identified
- Input is bulk RNA-seq or aggregated data where per-cell population percentages cannot
  be computed
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Analysis environment used to load count matrices/annotations, compute per-sample
    cell-type percentages and niche scores, run Welch's tests, and write the CSV
  repo: https://github.com/multitalk/scCrossTalk
- name: scCrossTalk
  role: R package from the paper whose transplanted-liver scRNA-seq workflow the annotated
    samples and niche populations come from
  repo: https://github.com/multitalk/scCrossTalk
- name: Seurat
  role: Single-cell framework with which scCrossTalk is compatible; source of the
    scRNA-seq objects carrying per-cell annotations
- name: ggplot2
  role: Visualization of the per-sample niche scores and non-EAD vs EAD group comparisons
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:9c8541a22ab94be6292c5d65f1279a11dfd84468cb254c361af93416be92ab3e
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# single-cell-sample-metadata-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate each liver transplant scRNA-seq sample with derived metadata — percentages of MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils, their sum as an EAD-associated immune niche score, plus donor, stage (cold preservation vs 2-hour portal reperfusion), and EAD status — producing a 12-row per-sample table for group comparison.

## When to use

Use when scRNA-seq samples already have cell-type annotations and you must attach per-sample quantitative metadata (cell-type percentages, composite niche scores, biological labels) for downstream group testing.

## When NOT to use

- The goal is inferring ligand-receptor cell-cell communication (e.g. enriching CellTalkDB LR pairs with scCrossTalk's Z-test) rather than scoring per-sample cell composition
- Single-cell data lack cell type annotations, so the three niche populations (MAIT, GZMB+ GZMK+ NK, S100A12+ neutrophils) cannot be identified
- Input is bulk RNA-seq or aggregated data where per-cell population percentages cannot be computed

## Inputs

- scRNA-seq count matrices for the 12 liver transplant samples (75,231 cells total from 7 donors)
- Per-cell cell type annotations including MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils
- Per-sample clinical labels: donor ID, stage (cold preservation vs 2-hour portal reperfusion), EAD status

## Outputs

- 12-row table of per-sample immune niche scores (ead_niche_scores.csv) with group labels
- Per-sample percentages of each of the three niche cell populations
- One-sided Welch's test p-values for non-EAD vs EAD comparison per stage

## How to apply

Load the scRNA-seq count matrices and cell type annotations for the 12 liver samples (7 donors, cold preservation and portal reperfusion stages) in R. For each sample, compute the percentage of cells belonging to each of the three EAD-associated niche populations (mucosal-associated invariant T cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) relative to all annotated cells in that sample. Sum the three percentages per sample to obtain the immune niche score, then label each sample with its donor, stage, and EAD status. Compare niche scores between non-EAD and EAD patients within each stage using a one-sided Welch's test, the same test used in the paper for non-EAD vs EAD comparisons. Write the 12-row per-sample score table with group labels to ead_niche_scores.csv.

## Related tools

- **R** (Analysis environment used to load count matrices/annotations, compute per-sample cell-type percentages and niche scores, run Welch's tests, and write the CSV) — https://github.com/multitalk/scCrossTalk
- **scCrossTalk** (R package from the paper whose transplanted-liver scRNA-seq workflow the annotated samples and niche populations come from) — https://github.com/multitalk/scCrossTalk
- **Seurat** (Single-cell framework with which scCrossTalk is compatible; source of the scRNA-seq objects carrying per-cell annotations)
- **ggplot2** (Visualization of the per-sample niche scores and non-EAD vs EAD group comparisons)

## Examples

```
pct <- sapply(c('MAIT','GZMB+ GZMK+ NK','S100A12+ neutrophil'), function(ct) 100*mean(celltype==ct)); score <- sum(pct); write.csv(scores, 'ead_niche_scores.csv')
```

## Evaluation signals

- Output CSV exists and contains exactly 12 rows, one per sample
- Each niche score is the sum of three cell-type percentages and is bounded between 0 and 300%
- Per-sample percentages for each niche population fall between 0 and 100%
- EAD samples show higher niche scores than non-EAD samples at both stages (cold preservation and portal reperfusion), consistent with the reported pathogenic immune niche association
- Every sample row carries complete labels: donor, stage, and EAD status

## Limitations

- The niche score is defined by exactly three cell populations (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) identified in this specific 12-sample/7-donor liver transplant cohort; other studies would need to re-derive their own constituent populations
- One-sided Welch's test assumes the direction of the non-EAD vs EAD difference specified a priori
- Per-sample percentages depend on the completeness and quality of upstream cell-type annotation across all annotated cells in each sample

## Evidence
