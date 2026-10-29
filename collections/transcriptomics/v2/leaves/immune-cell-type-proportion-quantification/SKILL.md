---
name: immune-cell-type-proportion-quantification
description: Use when scRNA-seq count matrices with per-cell cell-type annotations
  are available and a per-sample composite score of specific immune populations (e.g.
  an EAD-associated niche score) is needed to compare patient groups across stages.
when_to_use_negative:
- Input is already a per-sample cell-type composition/feature table rather than raw
  count matrices with per-cell annotations
- The goal is ligand-receptor cell-cell communication inference (use the scCrossTalk
  find_lrpairs/get_top_lrpairs workflow instead)
- Data lacks cell-type annotations for MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+
  neutrophils, so the three niche populations cannot be identified
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scCrossTalk
  role: R package whose create_scCrossTalk interface consumes the scRNA-seq count
    matrix and per-cell cell-type vector; the paper uses it to dissect cell-cell communications
    in the transplanted livers of EAD vs non-EAD patients alongside the niche-score
    analysis
  repo: https://github.com/multitalk/scCrossTalk
- name: R
  role: Analysis environment for loading count matrices, computing per-sample cell-type
    percentages, summing the three niche percentages, and running one-sided Welch's
    tests
- name: ggplot2
  role: Visualization of per-sample niche scores and non-EAD vs EAD differences
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:d0731ae6bda6a01c60a7b6e20109cbd3a0a577a633cc1e2d7ba0220167979d01
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# immune-cell-type-proportion-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies per-sample proportions of specific immune cell populations (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) from annotated scRNA-seq data and aggregates them into an EAD-associated immune niche score for group comparison in liver transplantation samples.

## When to use

Use when scRNA-seq count matrices with per-cell cell-type annotations are available and a per-sample composite score of specific immune populations (e.g. an EAD-associated niche score) is needed to compare patient groups across stages.

## When NOT to use

- Input is already a per-sample cell-type composition/feature table rather than raw count matrices with per-cell annotations
- The goal is ligand-receptor cell-cell communication inference (use the scCrossTalk find_lrpairs/get_top_lrpairs workflow instead)
- Data lacks cell-type annotations for MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils, so the three niche populations cannot be identified

## Inputs

- scRNA-seq count matrices for 12 liver samples (7 donors, cold perfusion and portal reperfusion stages)
- Per-cell cell-type annotations distinguishing MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils from all other annotated cells
- Sample-level metadata: donor ID, stage (cold preservation vs 2-hour portal reperfusion), EAD status

## Outputs

- ead_niche_scores.csv: a 12-row table of per-sample immune niche scores with group labels
- One-sided Welch's test statistics comparing non-EAD vs EAD niche scores within each stage
- Optional ggplot2 visualizations of score differences between EAD groups and stages

## How to apply

Load the scRNA-seq count matrices and cell-type annotations for the 12 liver samples (7 donors, cold preservation vs 2-hour portal reperfusion stages) in R, using Seurat-compatible objects. For each sample, compute the percentage of cells belonging to each niche population — mucosal-associated invariant T (MAIT) cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils — relative to all annotated cells in that sample. Sum the three percentages per sample to obtain the immune niche score, and label each sample with donor, stage, and EAD status. Compare scores between non-EAD and EAD patients within each stage using a one-sided Welch's test, the same test used in the paper for non-EAD vs EAD LR-pair comparisons. Write the 12-row per-sample score table to ead_niche_scores.csv and optionally visualize with ggplot2.

## Related tools

- **scCrossTalk** (R package whose create_scCrossTalk interface consumes the scRNA-seq count matrix and per-cell cell-type vector; the paper uses it to dissect cell-cell communications in the transplanted livers of EAD vs non-EAD patients alongside the niche-score analysis) — https://github.com/multitalk/scCrossTalk
- **R** (Analysis environment for loading count matrices, computing per-sample cell-type percentages, summing the three niche percentages, and running one-sided Welch's tests)
- **ggplot2** (Visualization of per-sample niche scores and non-EAD vs EAD differences)

## Examples

```
Rscript -e 'anno <- read.csv("cell_annotations.csv"); score <- function(d,ct) 100*mean(ct[d]=="MAIT cells")+100*mean(ct[d]=="GZMB+ GZMK+ NK cells")+100*mean(ct[d]=="S100A12+ neutrophils"); write.csv(data.frame(sample=unique(anno$sample), niche_score=sapply(split(anno$celltype, anno$sample), score)), "ead_niche_scores.csv")'
```

## Evaluation signals

- ead_niche_scores.csv exists with exactly 12 rows, one per sample (7 donors across cold preservation and 2-hour portal reperfusion stages)
- Each niche score is a sum of three cell-type percentages and is therefore bounded between 0% and 300%
- Each percentage is computed relative to all annotated cells in that sample, not a subset
- EAD samples show higher niche scores than non-EAD samples at both stages, consistent with the reported pathogenic immune niche association
- Group comparison uses a one-sided Welch's test performed within each stage, mirroring the paper's non-EAD vs EAD test setup

## Limitations

- The niche score depends entirely on upstream cell-type annotation quality; misannotation of MAIT cells, GZMB+ GZMK+ NK cells, or S100A12+ neutrophils propagates into the score
- Cohort size is small — 12 liver samples from 7 donors — and stage comparisons are limited to cold preservation vs 2-hour portal reperfusion
- The association was verified in two independent cohorts in the paper, but a new analysis should not assume generalizability beyond the EAD immune-niche populations defined here

## Evidence
