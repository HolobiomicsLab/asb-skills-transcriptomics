---
name: ppin-activity-quantification-per-cell
description: Use when you have a normalized scRNA-seq expression matrix (genes x cells)
  and need per-cell activity scores for biological processes via PPIN edge sums, e.g.
  to build BP activity matrices for correlation or cell-type annotation.
when_to_use_negative:
- Input is not a gene-by-cell normalized expression matrix (e.g. it is already a feature/activity
  table or bulk expression without single-cell resolution)
- The biological process of interest has no curated gene list and no bundled ORIGINS2
  edge set — build_ppin requires a gene ID list plus the full Human PPIN
- The goal is standard cell-type marker-gene annotation (e.g. gene-set scoring with
  Seurat/AddModuleScore) rather than PPIN-based activity quantification
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: ORIGINS2
  role: R package providing activity() to compute per-cell PPIN activity, 18 bundled
    BP edge sets, build_ppin() and full_human_ppin for custom networks
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Used to install ORIGINS2 from GitHub via remotes::install_github("danielasenraoka/ORIGINS2")
- name: dplyr
  role: Imported dependency of ORIGINS2 used in the package's data handling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:5a4c74d591d599e5bc5a4e9072a89b5f3b578239207c5a01b3133ffbaa63deaf
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# ppin-activity-quantification-per-cell

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify per-cell activity of protein-protein interaction networks (PPINs) associated with 18 biological processes from a normalized scRNA-seq expression matrix using the ORIGINS2 R package's activity() function, enabling downstream BP-vs-BP correlation analyses for cell annotation.

## When to use

Use when you have a normalized scRNA-seq expression matrix (genes x cells) and need per-cell activity scores for biological processes via PPIN edge sums, e.g. to build BP activity matrices for correlation or cell-type annotation.

## When NOT to use

- Input is not a gene-by-cell normalized expression matrix (e.g. it is already a feature/activity table or bulk expression without single-cell resolution)
- The biological process of interest has no curated gene list and no bundled ORIGINS2 edge set — build_ppin requires a gene ID list plus the full Human PPIN
- The goal is standard cell-type marker-gene annotation (e.g. gene-set scoring with Seurat/AddModuleScore) rather than PPIN-based activity quantification

## Inputs

- Normalized scRNA-seq expression matrix (matrix or data frame, genes as rows, cells as columns)
- PPIN adjacency edge data frame for a biological process (one of 18 bundled sets such as differentiation_edges, cell_cycle_edges, immune response edges)
- Logical normalization flag (FALSE = raw edge sums, TRUE = activity divided by number of PPIN edges)
- Optional: gene ID list for a custom BP, plus full_human_ppin adjacency list (bundled with ORIGINS2)

## Outputs

- Per-cell activity vector for each BP (one value per cell)
- BP x cell activity matrix assembled across the 18 biological processes
- Optional downstream: BP x BP Pearson correlation matrix (e.g. written to bp_activity_correlation_tnbc.csv)

## How to apply

Install ORIGINS2 from GitHub with remotes::install_github("danielasenraoka/ORIGINS2") and load it with library(ORIGINS2). Provide expression_matrix (rows = genes, columns = cells, normalized expression) and choose one of the 18 BP adjacency edge sets shipped with the package (e.g. differentiation_edges, cell_cycle_edges, acute_inflammatory_response_edges). Call activity(expression_matrix, adjacency_edges, normalization): with normalization=FALSE the output is the raw sum of all PPIN edges; with normalization=TRUE activity is divided by the number of edges of the PPIN — running both settings serves as a sensitivity check. Stack the resulting per-BP, per-cell vectors into a BP x cell activity matrix; downstream, pairwise Pearson correlation (cor(method='pearson')) among the 18 BP activities can reveal programs that are co-active (tumoral TNBC breast sample) versus anti-correlated (healthy sample). For processes not shipped, build a custom PPIN with build_ppin(genes_bp, ppi_edges_complete = full_human_ppin), using the full Human PPIN adjacency list included in the package.

## Related tools

- **ORIGINS2** (R package providing activity() to compute per-cell PPIN activity, 18 bundled BP edge sets, build_ppin() and full_human_ppin for custom networks) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Used to install ORIGINS2 from GitHub via remotes::install_github("danielasenraoka/ORIGINS2"))
- **dplyr** (Imported dependency of ORIGINS2 used in the package's data handling)

## Examples

```
library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- The activity() call returns one activity value per cell (vector length equals number of columns of expression_matrix) for the chosen BP edge set
- Running activity() with normalization=FALSE vs normalization=TRUE yields edge-sum vs edge-count-normalized scores — both settings should preserve relative per-cell ordering (sensitivity check)
- The assembled BP x cell matrix has 18 rows (one per bundled BP) and unit-length per-cell activity series suitable for cor(method='pearson')
- Downstream 18 x 18 Pearson correlation matrix is symmetric with unit diagonal, and in the TNBC breast sample most off-diagonal coefficients are positive or near zero (no strong anti-correlations), contrasting with the healthy sample
- Custom PPINs built via build_ppin(genes_bp, ppi_edges_complete = full_human_ppin) yield edges that are a subset of the full Human PPIN adjacency list

## Limitations

- Activity scores depend on the normalization flag: with normalization=FALSE the output is the raw sum of all PPIN edges, so BPs with larger networks get larger raw activities; run normalization=TRUE as a sensitivity setting
- Only 18 BP adjacency edge sets are pre-loaded; other processes require a gene ID list and the bundled full_human_ppin
- The finding that tumoral breast cells keep diverse programs active in a differentiated manner was established for the TNBC breast sample versus healthy sample; the correlation structure is sample-specific

## Evidence
