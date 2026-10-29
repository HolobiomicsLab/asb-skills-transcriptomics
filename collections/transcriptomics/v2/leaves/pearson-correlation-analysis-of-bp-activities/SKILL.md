---
name: pearson-correlation-analysis-of-bp-activities
description: Use when you have per-cell activity vectors for multiple ORIGINS2 biological-process
  PPINs (e.g. 18 BP edge sets) from a normalized scRNA-seq matrix and need to characterize
  cross-BP relationships such as anti-correlation between immune and replication programs.
when_to_use_negative:
- Input is a raw (unnormalized) count matrix — the activity() function expects a normalized
  expression matrix.
- You need cell-type annotation of individual cells rather than relationships among
  biological processes; the ORIGINS2 activity/UMAP annotation workflow is the appropriate
  skill instead.
- Only one BP edge set is available — Pearson correlation across BP activities requires
  at least two per-BP activity profiles.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ORIGINS2
  role: R package that supplies the activity() function to compute per-cell PPIN activity
    for each biological-process edge set and the 18 preloaded BP adjacency edge sets
    used as correlation inputs
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: R
  role: environment in which activity() is run and base-stats cor(method='pearson')
    computes the BP-to-BP correlation matrix
- name: dplyr
  role: data handling for assembling the per-BP activity vectors into a BP x cell
    activity table before correlation
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:9ad2d7a9c146ed7ce60e6a657fcaace80748f13cc4084468bbfaf84d8461d31d
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
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pearson-correlation-analysis-of-bp-activities

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pairwise Pearson correlation matrices among per-cell ORIGINS2 PPIN activity profiles of biological processes (BPs) in a scRNA-seq sample, to reveal whether BPs cluster into correlated or anti-correlated groups (e.g. inflammatory/immune vs replication/differentiation).

## When to use

Use when you have per-cell activity vectors for multiple ORIGINS2 biological-process PPINs (e.g. 18 BP edge sets) from a normalized scRNA-seq matrix and need to characterize cross-BP relationships such as anti-correlation between immune and replication programs.

## When NOT to use

- Input is a raw (unnormalized) count matrix — the activity() function expects a normalized expression matrix.
- You need cell-type annotation of individual cells rather than relationships among biological processes; the ORIGINS2 activity/UMAP annotation workflow is the appropriate skill instead.
- Only one BP edge set is available — Pearson correlation across BP activities requires at least two per-BP activity profiles.

## Inputs

- normalized scRNA-seq expression matrix (genes as rows, cells as columns) for a single sample (e.g. healthy breast or TNBC sample)
- ORIGINS2 BP adjacency edge sets (18 BP data frames such as differentiation_edges, acute_inflammatory_response_edges, humoral_immune_response_edges, dna_repair_edges, mitotic_cell_cycle_edges)
- normalization flag setting (FALSE = raw edge sums; TRUE = activity divided by number of PPIN edges)

## Outputs

- BP x BP Pearson correlation matrix (e.g. 18 x 18, symmetric, unit diagonal), e.g. bp_activity_correlation_matrix.csv (healthy) or bp_activity_correlation_tnbc.csv
- identified BP grouping structure (correlated vs anti-correlated groups)

## How to apply

Load ORIGINS2 (library(ORIGINS2)) and, for each of the 18 BP adjacency edge sets shipped with the package (e.g. differentiation_edges, immune_response_edges, dna_replication_edges, cell_cycle_edges), compute the per-cell activity with activity(expression_matrix, adjacency_edges, normalization), running once with normalization=FALSE (raw sum of all PPIN edges) and optionally normalization=TRUE (activity divided by the number of PPIN edges) as a sensitivity check. Assemble the per-BP activity vectors into a BP x cell activity table (handle with dplyr). Compute the pairwise Pearson correlation matrix across BP activity profiles with R's cor(method='pearson'). In the healthy breast sample the BPs split into two main anti-correlated groups — inflammatory/immune responses vs cell replication and differentiation — whereas in the TNBC sample most BPs are positively or weakly correlated, reflecting that tumor cells keep diverse programs active in a differentiated manner; inspect off-diagonal coefficients to confirm which regime your sample matches.

## Related tools

- **ORIGINS2** (R package that supplies the activity() function to compute per-cell PPIN activity for each biological-process edge set and the 18 preloaded BP adjacency edge sets used as correlation inputs) — https://github.com/danielasenraoka/ORIGINS2
- **R** (environment in which activity() is run and base-stats cor(method='pearson') computes the BP-to-BP correlation matrix)
- **dplyr** (data handling for assembling the per-BP activity vectors into a BP x cell activity table before correlation)

## Examples

```
library(ORIGINS2); act <- sapply(list(immune_response_edges, dna_replication_edges, differentiation_edges, cell_cycle_edges), function(e) activity(expression_matrix, e, FALSE)); C <- cor(t(act), method='pearson')
```

## Evaluation signals

- Output CSV is an N x N (e.g. 18 x 18) symmetric correlation matrix with unit diagonal and Pearson coefficients in [-1, 1].
- Healthy breast sample: BPs separate into two main groups (inflammatory/immune responses vs cell replication and differentiation) that appear anti-correlated — strongly negative off-diagonal coefficients between the two groups.
- TNBC sample: the majority of off-diagonal Pearson coefficients are positive or near zero, with no strongly negative anti-correlations, contrasting with the healthy sample.
- Sensitivity check: the grouping pattern is consistent whether activity is computed with normalization=FALSE or normalization=TRUE.
- Each per-BP activity vector has length equal to the number of cells in the expression matrix.

## Limitations

- Results are sample-dependent: the anti-correlated inflammatory/immune vs replication/differentiation pattern is characteristic of the healthy breast sample and is not reproduced in the TNBC sample, so group structure must be inspected per dataset.
- Activity with normalization=FALSE is the raw sum of all PPIN edges and is therefore sensitive to PPIN size; the normalization=TRUE variant (divided by number of edges) is provided as a sensitivity setting.
- Pearson correlation of activity profiles captures only linear relationships across cells and presumes the expression matrix is normalized.

## Evidence
