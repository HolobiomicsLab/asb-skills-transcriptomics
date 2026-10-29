---
name: tumor-vs-healthy-bp-coactivation-comparison
description: Use when per-cell scRNA-seq activity profiles for the 18 ORIGINS2 biological
  processes are available for tumor and healthy samples and the goal is to detect
  which BP pairs are correlated vs anti-correlated per condition.
when_to_use_negative:
- The input is not a per-cell scRNA-seq normalized expression matrix (genes as rows,
  cells as columns) — bulk or already-aggregated profiles cannot yield per-cell BP
  activity vectors
- Only one condition/sample is available with no healthy counterpart — the skill's
  validation hinges on the tumor-vs-healthy contrast in correlation signs
- The goal is cell-type annotation/labeling rather than BP co-activation structure
  — use the ORIGINS2 activity/UMAP annotation workflow instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ORIGINS2
  role: Provides the 18 BP PPIN adjacency edge sets and the activity() function that
    computes per-cell BP activity from the normalized expression matrix
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: R
  role: Runtime environment in which the activity computation and the Pearson correlation
    matrix (cor(method='pearson')) are performed
- name: remotes
  role: Used to install ORIGINS2 from GitHub via remotes::install_github()
- name: dplyr
  role: Imported dependency of ORIGINS2 used in the data-manipulation steps of the
    workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:4b47a0bc1c4e4a01e1eeb64630d44c43245ec01e2853e861a6cfc377e0227ef2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielasenraoka/ORIGINS2
  repo_url: https://github.com/danielasenraoka/ORIGINS2
  tools:
  - ORIGINS2
  - R
  - dplyr
  - remotes
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Tumor vs healthy BP co-activation comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pairwise Pearson correlations among the 18 package-shipped biological-process (BP) PPIN activity vectors per cell, to contrast co-activation patterns in a tumoral (TNBC) breast sample against the healthy sample where inflammatory/immune and replication/differentiation programs are anti-correlated.

## When to use

Use when per-cell scRNA-seq activity profiles for the 18 ORIGINS2 biological processes are available for tumor and healthy samples and the goal is to detect which BP pairs are correlated vs anti-correlated per condition.

## When NOT to use

- The input is not a per-cell scRNA-seq normalized expression matrix (genes as rows, cells as columns) — bulk or already-aggregated profiles cannot yield per-cell BP activity vectors
- Only one condition/sample is available with no healthy counterpart — the skill's validation hinges on the tumor-vs-healthy contrast in correlation signs
- The goal is cell-type annotation/labeling rather than BP co-activation structure — use the ORIGINS2 activity/UMAP annotation workflow instead

## Inputs

- Normalized scRNA-seq expression matrix (genes x cells) for the tumoral (TNBC) breast sample and the healthy breast sample
- The 18 BP adjacency edge data frames shipped with ORIGINS2 (e.g. differentiation_edges, cell_cycle_edges, acute_inflammatory_response_edges, adaptive_immune_response_edges, dna_repair_edges, dna_replication_edges)
- Optional: full_human_ppin adjacency list via build_ppin() for custom BP gene sets

## Outputs

- BP x cell activity matrix per sample (18 BP rows)
- 18 x 18 symmetric Pearson correlation matrix with unit diagonal per sample (e.g. bp_activity_correlation_tnbc.csv)
- Comparison of off-diagonal correlation signs between tumor and healthy samples

## How to apply

Load ORIGINS2 and select its 18 BP adjacency edge sets (differentiation_edges, cell_cycle_edges, immune_response_edges, etc.). For each BP, compute the per-cell activity vector with activity(expression_matrix, adjacency_edges, normalization), running once with normalization=FALSE (raw PPIN edge sums) and once with normalization=TRUE (activity divided by the number of PPIN edges) as a sensitivity check. Assemble the 18 activity vectors into a BP x cell matrix per sample, then compute the pairwise Pearson correlation matrix with cor(method='pearson') in R. Interpret the off-diagonal coefficients: in the tumoral breast sample most BPs should be positively or weakly correlated rather than anti-correlated, reflecting that tumor breast cells keep diverse biological programs active in a differentiated manner within the cluster, whereas the healthy sample shows anti-correlation between inflammatory/immune and replication/differentiation groups. Write the matrix (e.g. bp_activity_correlation_tnbc.csv) for validation and downstream comparison across conditions.

## Related tools

- **ORIGINS2** (Provides the 18 BP PPIN adjacency edge sets and the activity() function that computes per-cell BP activity from the normalized expression matrix) — https://github.com/danielasenraoka/ORIGINS2
- **R** (Runtime environment in which the activity computation and the Pearson correlation matrix (cor(method='pearson')) are performed)
- **remotes** (Used to install ORIGINS2 from GitHub via remotes::install_github())
- **dplyr** (Imported dependency of ORIGINS2 used in the data-manipulation steps of the workflow)

## Examples

```
library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE); bp_cor <- cor(t(bp_activity_matrix), method='pearson'); write.csv(bp_cor, 'bp_activity_correlation_tnbc.csv')
```

## Evaluation signals

- Output CSV is an 18 x 18 symmetric matrix with unit diagonal (all self-correlations = 1)
- Majority of off-diagonal Pearson coefficients in the TNBC sample are positive or near zero, with no strongly negative anti-correlations
- Healthy-sample matrix reproduces the anti-correlation between inflammatory/immune BPs and replication/differentiation BPs, contrasting with the tumor sample
- Raw (normalization=FALSE) and normalized (normalization=TRUE) sensitivity runs yield concordant correlation-sign patterns
- Activity vectors are reproducible from the README example call pattern: activity(expression_matrix, <bp>_edges, FALSE/TRUE)

## Limitations

- The finding is demonstrated on breast (TNBC vs healthy) scRNA-seq data; correlation-sign patterns in other tissues/conditions may differ
- The normalization parameter changes the scale of activity (raw edge sums vs edge-count-normalized); results should be checked for sensitivity to this choice
- Custom BPs require a gene ID list and build_ppin() against full_human_ppin; BP coverage is limited to genes present in the expression matrix and the human PPIN

## Evidence
