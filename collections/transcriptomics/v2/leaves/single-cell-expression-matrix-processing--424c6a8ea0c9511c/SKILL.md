---
name: single-cell-expression-matrix-processing
description: Use when you have a normalized scRNA-seq expression matrix and want per-cell
  PPIN activity per biological process, then Pearson-correlate those BP activity profiles
  across cells to see whether BPs cluster into distinct or anti-correlated groups.
when_to_use_negative:
- The input is not a per-cell normalized expression matrix (e.g. already aggregated
  pseudobulk or a feature/summary table) — activity() expects genes as rows and cells
  as columns.
- You need per-cell BP activity normalized by PPIN size for cross-BP magnitude comparison
  — the Fig. 3C workflow uses normalization=FALSE (raw edge sums); set normalization=TRUE
  instead for size-normalized activity.
- The biological process of interest is not among the 18 preloaded BP edge sets and
  you have no gene ID list to build a custom PPIN with build_ppin().
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ORIGINS2
  role: R package providing the activity() function to compute per-cell PPIN activity
    for preloaded BP edge sets and build_ppin() for custom processes
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: R
  role: Runtime language; base stats cor() computes the Pearson correlation matrix
    across BP activity profiles
- name: dplyr
  role: Data handling for assembling the per-cell BP activity table prior to correlation
- name: remotes
  role: Used to install ORIGINS2 from GitHub via remotes::install_github("danielasenraoka/ORIGINS2")
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:7cd320c5b724b02f86288f2e59fdb18530f6b7a6d0b9d069131b0445f1e8d4dc
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
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-expression-matrix-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-cell ORIGINS2 PPIN activity vectors from a normalized scRNA-seq expression matrix (genes as rows, cells as columns) and correlate the resulting biological-process activities across cells to reveal BP groups, e.g. the two anti-correlated BP clusters (inflammatory/immune vs replication/differentiation) reported in the healthy breast sample (Fig. 3C).

## When to use

Use when you have a normalized scRNA-seq expression matrix and want per-cell PPIN activity per biological process, then Pearson-correlate those BP activity profiles across cells to see whether BPs cluster into distinct or anti-correlated groups.

## When NOT to use

- The input is not a per-cell normalized expression matrix (e.g. already aggregated pseudobulk or a feature/summary table) — activity() expects genes as rows and cells as columns.
- You need per-cell BP activity normalized by PPIN size for cross-BP magnitude comparison — the Fig. 3C workflow uses normalization=FALSE (raw edge sums); set normalization=TRUE instead for size-normalized activity.
- The biological process of interest is not among the 18 preloaded BP edge sets and you have no gene ID list to build a custom PPIN with build_ppin().

## Inputs

- Normalized scRNA-seq expression matrix (matrix or data frame; rows = genes, columns = cells)
- ORIGINS2 BP adjacency edge sets (e.g. differentiation_edges, acute_inflammatory_response_edges, cell_cycle_edges, dna_replication_edges, full_human_ppin for custom build_ppin)

## Outputs

- Per-cell activity vector per BP (numeric vector, one value per cell, sum of all PPIN edges when normalization=FALSE)
- Pearson correlation matrix across BP activity profiles
- bp_activity_correlation_matrix.csv

## How to apply

Load ORIGINS2 (library(ORIGINS2)) and read the normalized expression matrix of the sample with genes as rows and cells as columns. For each BP edge set included in the package (e.g. differentiation_edges, acute_inflammatory_response_edges, dna_replication_edges), call activity(expression_matrix, adjacency_edges, FALSE); setting normalization=FALSE means the output vector is the raw sum of all PPIN edges for each cell. Assemble the per-cell activity vectors — here the inflammatory/immune edge sets (acute/chronic inflammatory response, inflammatory, immune, innate, adaptive, humoral immune response) and the replication/differentiation sets (DNA replication, DNA repair, cell cycle, mitotic cell cycle, differentiation, stem differentiation, proliferation, stem proliferation) — into one activity table with dplyr. Compute the Pearson correlation matrix across BP activity profiles with base R's cor(), inspect it for block structure, and export it as bp_activity_correlation_matrix.csv. Correct application yields the expected two-group structure: inflammatory/immune BPs correlate with each other, replication/differentiation BPs with each other, and the two groups anti-correlated (healthy breast sample, Fig. 3C).

## Related tools

- **ORIGINS2** (R package providing the activity() function to compute per-cell PPIN activity for preloaded BP edge sets and build_ppin() for custom processes) — https://github.com/danielasenraoka/ORIGINS2
- **R** (Runtime language; base stats cor() computes the Pearson correlation matrix across BP activity profiles)
- **dplyr** (Data handling for assembling the per-cell BP activity table prior to correlation)
- **remotes** (Used to install ORIGINS2 from GitHub via remotes::install_github("danielasenraoka/ORIGINS2"))

## Examples

```
library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- The correlation matrix shows two main BP blocks: inflammatory/immune BPs (acute, chronic, inflammatory, immune, innate, adaptive, humoral) mutually correlated, and replication/differentiation BPs (DNA replication, DNA repair, cell cycle, mitotic cell cycle, differentiation, stem differentiation, proliferation, stem proliferation) mutually correlated.
- The two blocks appear anti-correlated with each other in the healthy breast sample, reproducing Fig. 3C.
- Each activity output has one entry per cell (length equals number of columns of the expression matrix) when computed with normalization=FALSE.
- All 15 BP edge sets used in the workflow resolve without error from the ORIGINS2 package data.
- The exported bp_activity_correlation_matrix.csv is square (15x15 BP labels) with diagonal = 1.

## Limitations

- Only 18 BP adjacency edge sets are preloaded in ORIGINS2; other processes require a gene ID list plus full_human_ppin via build_ppin().
- activity() with normalization=FALSE returns raw sums of PPIN edges, so activity magnitudes are not directly comparable across BPs with different edge counts unless normalized (normalization=TRUE).
- ORIGINS2 requires the full Human PPIN (full_human_ppin); custom PPINs built from incomplete gene ID lists may miss edges.

## Evidence
