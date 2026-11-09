---
name: sample-level-niche-score-computation
description: Use when a scRNA-seq study identifies a multi-cell-type disease niche
  and you need one scalar per sample (sum of cell-type percentages) to test group
  differences, e.g. EAD vs non-EAD across cold preservation and portal reperfusion
  stages.
when_to_use_negative:
- Input is already an aggregated abundance/feature table per sample rather than cell-level
  annotations — recompute percentages would be meaningless.
- The disease-associated niche of interest is defined by ligand-receptor communication
  scores rather than cell-type composition — use scCrossTalk cell-cell communication
  inference (find_lrpairs, plot_cci_*) instead.
- You need per-cell differential expression (e.g. SAA1 in hepatocytes, FPR1 in S100A12+
  neutrophils) rather than a per-sample compositional score.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: R
  role: Environment in which per-sample percentages of the three niche populations
    are computed, summed into the niche score, and compared with a one-sided Welch's
    test
- name: ggplot2
  role: Used to plot the per-stage non-EAD vs EAD niche score comparisons
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:e996cca4bebdcc3364aa6511575513425b380202eebb6fd20cfa7940e9357d1b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/multitalk/scCrossTalk
  tools:
  - R
  - ggplot2
  - scCrossTalk
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# sample-level-niche-score-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes a per-sample composite immune niche score by summing the percentages of defined disease-associated cell populations (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) within each scRNA-seq sample, enabling group-level statistical comparison of the niche between conditions such as EAD vs non-EAD liver transplant patients.

## When to use

Use when a scRNA-seq study identifies a multi-cell-type disease niche and you need one scalar per sample (sum of cell-type percentages) to test group differences, e.g. EAD vs non-EAD across cold preservation and portal reperfusion stages.

## When NOT to use

- Input is already an aggregated abundance/feature table per sample rather than cell-level annotations — recompute percentages would be meaningless.
- The disease-associated niche of interest is defined by ligand-receptor communication scores rather than cell-type composition — use scCrossTalk cell-cell communication inference (find_lrpairs, plot_cci_*) instead.
- You need per-cell differential expression (e.g. SAA1 in hepatocytes, FPR1 in S100A12+ neutrophils) rather than a per-sample compositional score.

## Inputs

- scRNA-seq count matrices for the 12 liver transplant samples (7 donors)
- Per-cell cell type annotations identifying MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils
- Sample metadata mapping each sample to donor, stage (cold preservation vs portal reperfusion), and EAD status

## Outputs

- 12-row table of per-sample immune niche scores with donor, stage, and EAD group labels (ead_niche_scores.csv)
- Per-stage one-sided Welch's test statistics comparing non-EAD vs EAD niche scores

## How to apply

Load the scRNA-seq count matrices and cell type annotations for the 12 liver samples (7 donors; cold preservation and 2-hour portal reperfusion stages) in R. For each sample, compute the percentage of cells belonging to each niche population — mucosal-associated invariant T (MAIT) cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils — relative to all annotated cells in that sample. Sum the three percentages per sample to obtain the immune niche score (bounded between 0% and 300%), and label each sample with donor, stage, and EAD status. Within each stage, compare non-EAD vs EAD scores using a one-sided Welch's test, matching the paper's one-sided non-EAD vs EAD comparison convention. Write the 12-row per-sample table with group labels to ead_niche_scores.csv for validation and downstream plotting (e.g. ggplot2).

## Related tools

- **R** (Environment in which per-sample percentages of the three niche populations are computed, summed into the niche score, and compared with a one-sided Welch's test)
- **ggplot2** (Used to plot the per-stage non-EAD vs EAD niche score comparisons)

## Evaluation signals

- ead_niche_scores.csv exists with exactly 12 rows, one per sample
- Each niche score is a sum of three cell-type percentages and therefore bounded between 0% and 300%
- EAD samples show higher niche scores than non-EAD samples within both the cold preservation and portal reperfusion stages, consistent with the reported pathogenic niche association
- Each score is reproducible as the arithmetic sum of the three per-population percentages computed relative to all annotated cells in that sample
- Group comparison uses a one-sided Welch's test per stage, matching the paper's non-EAD vs EAD testing convention

## Limitations

- The score is defined only for the three EAD-associated populations identified in this paper (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils); applying it to other tissues or niches requires re-deriving the constituent populations.
- Only 12 samples from 7 donors were available, so per-stage group comparisons have very limited statistical power.
- Percentages depend on the quality and completeness of cell type annotation; misannotation of MAIT, NK, or neutrophil subsets directly biases the score.

## Evidence
