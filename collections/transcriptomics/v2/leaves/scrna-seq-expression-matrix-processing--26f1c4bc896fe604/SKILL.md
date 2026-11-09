---
name: scrna-seq-expression-matrix-processing
description: Use when you have a normalized scRNA-seq expression matrix and want per-cell
  activity of BP-associated PPINs or a BP x cell activity matrix for downstream correlation/annotation
  analyses.
when_to_use_negative:
- Input is raw (not normalized) counts — activity() expects the normalized expression
  matrix of the data set
- Genes are not human gene IDs compatible with the full Human PPIN adjacency list
  (full_human_ppin)
- The goal is cell-type annotation of an already-annotated, clustered dataset rather
  than quantifying BP-PPIN activity or BP-BP correlation structure across cells
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ORIGINS2
  role: R package providing the activity() function for per-cell PPIN activity, the
    18 BP adjacency edge sets, full_human_ppin, and build_ppin() for custom BP networks
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Used to install ORIGINS2 from GitHub via remotes::install_github('danielasenraoka/ORIGINS2')
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: Imported dependency of ORIGINS2 used in the R workflow for data manipulation
- name: magrittr
  role: Imported dependency of ORIGINS2 providing the pipe operator used in the R
    workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:894b95992a4def6520a1f82fca364c9725cded792eb064034d880ad8f57c5d2d
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
  - magrittr
  - remotes
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# scrna-seq-expression-matrix-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Processes a normalized scRNA-seq expression matrix (genes x cells) with the ORIGINS2 R package to compute per-cell activities of PPINs associated with 18 biological processes, and derives pairwise Pearson correlations among BP activities (e.g. for a TNBC breast sample vs. healthy sample).

## When to use

Use when you have a normalized scRNA-seq expression matrix and want per-cell activity of BP-associated PPINs or a BP x cell activity matrix for downstream correlation/annotation analyses.

## When NOT to use

- Input is raw (not normalized) counts — activity() expects the normalized expression matrix of the data set
- Genes are not human gene IDs compatible with the full Human PPIN adjacency list (full_human_ppin)
- The goal is cell-type annotation of an already-annotated, clustered dataset rather than quantifying BP-PPIN activity or BP-BP correlation structure across cells

## Inputs

- Normalized scRNA-seq expression matrix or data frame (rows = genes, columns = cells)
- BP PPIN adjacency edge data frame (one of the 18 edge sets preloaded in ORIGINS2, e.g. differentiation_edges, acute_inflammatory_response_edges, cell_cycle_edges, dna_replication_edges)
- Optional: gene ID list for a custom BP, with full_human_ppin as the complete PPIN adjacency list
- normalization logical flag (FALSE = raw edge sums, TRUE = activity divided by number of PPIN edges)

## Outputs

- Per-cell activity vector for each BP (output of activity())
- BP x cell activity matrix (18 BP rows x cells)
- 18 x 18 symmetric Pearson correlation matrix with unit diagonal, e.g. written to bp_activity_correlation_tnbc.csv
- Custom PPIN edge data frame from build_ppin()

## How to apply

Load the ORIGINS2 package (library(ORIGINS2)) after installing via remotes::install_github('danielasenraoka/ORIGINS2'). Pass the normalized expression matrix (rows = genes, columns = cells) together with one of the 18 BP adjacency edge sets shipped with the package (e.g. differentiation_edges, cell_cycle_edges, immune_response_edges) to activity(expression_matrix, adjacency_edges, normalization); run once with normalization=FALSE (raw PPIN edge sums) and once with TRUE (activity divided by the number of PPIN edges) as a sensitivity check. Assemble per-BP activity vectors into a BP x cell matrix, compute the pairwise Pearson correlation matrix with cor(method='pearson'), and inspect off-diagonal coefficients. In the TNBC breast sample most BP pairs are positively or weakly correlated rather than anti-correlated, contrasting with the healthy sample where inflammatory/immune and replication/differentiation groups are anti-correlated. Custom BP PPINs can be built with build_ppin(genes_bp, ppi_edges_complete = full_human_ppin) using the full Human PPIN adjacency list included in the package.

## Related tools

- **ORIGINS2** (R package providing the activity() function for per-cell PPIN activity, the 18 BP adjacency edge sets, full_human_ppin, and build_ppin() for custom BP networks) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Used to install ORIGINS2 from GitHub via remotes::install_github('danielasenraoka/ORIGINS2')) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (Imported dependency of ORIGINS2 used in the R workflow for data manipulation)
- **magrittr** (Imported dependency of ORIGINS2 providing the pipe operator used in the R workflow)

## Examples

```
diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- Output correlation file exists and is an 18 x 18 symmetric matrix with unit diagonal
- Majority of off-diagonal Pearson coefficients for the TNBC sample are positive or near zero, with no strongly negative anti-correlations
- Contrast preserved: healthy sample shows anti-correlation between inflammatory/immune and replication/differentiation BP groups, while the TNBC sample does not
- Running activity() with normalization=FALSE and TRUE yields consistent correlation structure (sensitivity agreement)
- Every gene in the expression matrix that belongs to a BP PPIN is matched against the adjacency edges before summing edge activities

## Limitations

- Only 18 BP PPINs are preloaded; other processes require building a custom PPIN via build_ppin() against full_human_ppin, which is limited to the full Human PPIN adjacency list
- activity() requires a normalized expression matrix; raw counts as input are not handled by the documented workflow
- Findings (BP correlation patterns) are demonstrated for breast (TNBC) vs. healthy samples and may not transfer to other tissues or datasets

## Evidence
