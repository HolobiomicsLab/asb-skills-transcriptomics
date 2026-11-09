---
name: anti-correlated-bp-group-identification
description: Use when scRNA-seq cells have ORIGINS2 PPIN activities for several biological
  processes and you need to test whether BPs cluster into distinct, potentially anti-correlated
  groups within a sample.
when_to_use_negative:
- The expression matrix is not normalized — activity() expects a normalized expression
  matrix, so raw counts would bias the per-cell edge sums
- You only have one BP of interest and no set of BP activity profiles to correlate
  — build_ppin/activity alone suffices without the correlation step
- The goal is cluster-level annotation of cell groups rather than relationships among
  biological processes within one sample
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: ORIGINS2
  role: R package supplying activity() to compute per-cell PPIN activity for BP edge
    sets, preloaded BP edge data frames, and build_ppin() for custom BP networks
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: R
  role: Runtime environment; cor() from base stats computes the Pearson correlation
    matrix across BP activity profiles
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: Data handling for assembling per-cell BP activity vectors into a single activity
    table before correlation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:f0fbae56acce849397ce28e1a4c85f72cc63789c0d112c60a388f3a213ea4534
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# anti-correlated-bp-group-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-cell activity of multiple biological-process PPINs with the ORIGINS2 R package and builds a Pearson correlation matrix across BP activity profiles to identify BP groups (e.g. inflammatory/immune vs replication/differentiation) and their anti-correlation within a scRNA-seq sample.

## When to use

Use when scRNA-seq cells have ORIGINS2 PPIN activities for several biological processes and you need to test whether BPs cluster into distinct, potentially anti-correlated groups within a sample.

## When NOT to use

- The expression matrix is not normalized — activity() expects a normalized expression matrix, so raw counts would bias the per-cell edge sums
- You only have one BP of interest and no set of BP activity profiles to correlate — build_ppin/activity alone suffices without the correlation step
- The goal is cluster-level annotation of cell groups rather than relationships among biological processes within one sample

## Inputs

- Normalized scRNA-seq expression matrix (genes as rows, cells as columns)
- ORIGINS2 BP adjacency edge sets (e.g. acute_inflammatory_response_edges, differentiation_edges, dna_replication_edges, cell_cycle_edges, humoral_immune_response_edges)
- Optional custom PPINs built with build_ppin(genes_bp, ppi_edges_complete = full_human_ppin)

## Outputs

- Per-cell activity vector for each BP PPIN (unnormalized sum of PPIN edges when normalization=FALSE)
- Pearson correlation matrix across BP activity profiles
- CSV file bp_activity_correlation_matrix.csv encoding the BP group structure

## How to apply

Load ORIGINS2 (library(ORIGINS2)) and a normalized expression matrix with genes as rows and cells as columns. For each BP edge set (e.g. acute_inflammatory_response_edges, differentiation_edges, cell_cycle_edges), call activity(expression_matrix, adjacency_edges, normalization=FALSE) so the output vector is the raw sum of all PPIN edges per cell; assemble the inflammatory/immune BP vectors (acute, chronic, inflammatory, immune, innate, adaptive, humoral immune response edges) and the replication/differentiation vectors (dna_replication, dna_repair, cell_cycle, mitotic_cell_cycle, differentiation, stem_differentiation, proliferation, stem_proliferation edges) into one activity table. Compute the Pearson correlation matrix across BP activity profiles with R's cor() (dplyr for data handling), inspect it to confirm the two anti-correlated BP groups, and write the result to bp_activity_correlation_matrix.csv. Correct application is judged by a coherent block-diagonal correlation structure: within-group positive correlations and negative cross-group correlations between the immune and replication/differentiation blocks.

## Related tools

- **ORIGINS2** (R package supplying activity() to compute per-cell PPIN activity for BP edge sets, preloaded BP edge data frames, and build_ppin() for custom BP networks) — https://github.com/danielasenraoka/ORIGINS2
- **R** (Runtime environment; cor() from base stats computes the Pearson correlation matrix across BP activity profiles) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (Data handling for assembling per-cell BP activity vectors into a single activity table before correlation)

## Examples

```
library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE); cor_mat <- cor(activity_table); write.csv(cor_mat, "bp_activity_correlation_matrix.csv")
```

## Evaluation signals

- The correlation matrix shows two blocks: within-group positive Pearson correlations among the seven inflammatory/immune BPs and among the eight replication/differentiation BPs
- Cross-block entries between inflammatory/immune and replication/differentiation BPs are negative, confirming anti-correlation of the two groups
- Setting normalization=FALSE yields activity vectors equal to the sum of all PPIN edges per cell (sanity-check against manual row-sum of the edge-weighted expression)
- The output file bp_activity_correlation_matrix.csv is square and symmetric with unit diagonal (correlation of each BP with itself = 1)
- The number of activity columns equals the number of BP edge sets loaded (15 BP edge sets enumerated in the workflow)

## Limitations

- Only 18 BP adjacency edge sets are preloaded in the package; other BPs require build_ppin() with a gene ID list and full_human_ppin
- The normalization=FALSE output is the raw sum of PPIN edges, so BPs with larger networks may have systematically higher activity magnitudes that affect correlation interpretation
- Findings are sample-specific: the anti-correlated grouping was demonstrated for the healthy breast sample and may differ in tumor samples where diverse programs stay active within a cluster

## Evidence

- [methods] activity function signature and normalization parameter: "activity(expression_matrix, adjacency_edges, normalization)"
- [methods] activity call example with differentiation_edges: "diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)"
- [readme] README description of normalization=FALSE semantics: "if FALSE no normalization is performed and the output vector is the sum of all the PPIN edges"
- [readme] README description of expected expression matrix layout: "expression_matrix is a matrix or data frame containing the normalized expression matrix of the data set, where each row is a gene and each column is a cell."
- [readme] README statement of package purpose: "R package that calculates the activity of protein-protein interaction networks (PPIN) associated with biological processes for scRNA-seq data."
- [abstract] Abstract finding on differentiated programs within tumor clusters: "tumor breast cells, even when they form a single group in the UMAP space, keep diverse biological programs active in a differentiated manner within the cluster."
