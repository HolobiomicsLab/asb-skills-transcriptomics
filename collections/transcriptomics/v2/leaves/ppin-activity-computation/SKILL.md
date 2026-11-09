---
name: ppin-activity-computation
description: Use when you have a normalized scRNA-seq expression matrix (genes as
  rows, cells as columns) and need per-cell activity scores for biological-process
  PPINs, e.g. for cell annotation, UMAP maps, or correlation of process activities.
when_to_use_negative:
- The input is raw (unnormalized) counts or already a processed feature/summary table
  — activity() expects a normalized expression matrix with genes as rows and cells
  as columns.
- The goal is pathway enrichment on bulk RNA-seq or gene-set statistics rather than
  per-cell PPIN activity scoring in single-cell data.
- A predefined number of cell types or states must be specified up front — ORIGINS2
  explicitly quantifies BP activity per cell without requiring predefined cell types
  or states.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: ORIGINS2
  role: Core R package providing the activity() scoring function, 18 preloaded BP
    adjacency edge sets, full_human_ppin, and build_ppin() for custom PPINs
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Used to install ORIGINS2 from its GitHub repository (remotes::install_github("danielasenraoka/ORIGINS2"))
- name: dplyr
  role: Data manipulation for assembling per-cell activity vectors into cells x process
    activity tables (Imports dependency of ORIGINS2)
- name: magrittr
  role: Piping (%>%) for the dplyr-based assembly of activity tables (Imports dependency
    of ORIGINS2)
- name: R
  role: Runtime environment; ORIGINS2 is freely available as an open source R package
    for scRNA-seq cell annotation
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_008
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:25e1213bcc05d87cd1413e059e4754a566e612d7ae7df39abf43afe946a9921f
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# ppin-activity-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-cell activity of protein-protein interaction networks (PPIN) associated with biological processes from a normalized scRNA-seq expression matrix using the ORIGINS2 R package's activity() function. This produces per-cell, per-process activity scores usable for clustering, UMAP embedding, and statistical comparison.

## When to use

Use when you have a normalized scRNA-seq expression matrix (genes as rows, cells as columns) and need per-cell activity scores for biological-process PPINs, e.g. for cell annotation, UMAP maps, or correlation of process activities.

## When NOT to use

- The input is raw (unnormalized) counts or already a processed feature/summary table — activity() expects a normalized expression matrix with genes as rows and cells as columns.
- The goal is pathway enrichment on bulk RNA-seq or gene-set statistics rather than per-cell PPIN activity scoring in single-cell data.
- A predefined number of cell types or states must be specified up front — ORIGINS2 explicitly quantifies BP activity per cell without requiring predefined cell types or states.

## Inputs

- Normalized scRNA-seq expression matrix (rows = genes, columns = cells), matrix or data frame
- PPIN adjacency edge data frame for a biological process, either preloaded in ORIGINS2 (e.g. differentiation_edges, cell_cycle_edges, full_human_ppin) or user-built
- Gene ID list for the biological process of interest (when building a custom PPIN)
- Boolean normalization parameter (FALSE = sum of all PPIN edges; TRUE = normalized by PPIN edge count)

## Outputs

- Per-cell activity vector for each biological-process PPIN (length = number of cells)
- Cells x biological-process activity score table (for multi-process analyses)
- CSV files of activity values and derived statistics (e.g. bp_activity_correlation_matrix.csv, proliferation_activity_mannwhitney.csv, embedding_pseudotime.csv)

## How to apply

Install ORIGINS2 from GitHub via remotes::install_github("danielasenraoka/ORIGINS2") and load it with library(ORIGINS2). Call activity(expression_matrix, adjacency_edges, normalization) where expression_matrix is the normalized matrix (rows = genes, columns = cells) and adjacency_edges is one of the 18 preloaded BP edge data frames (e.g. differentiation_edges, dna_replication_edges, innate_immune_response_edges) or a custom network built with build_ppin(genes_bp, ppi_edges_complete = full_human_ppin). Choose the normalization flag deliberately: with normalization=FALSE the output vector is the sum of all PPIN edge contributions (used for cross-BP correlation and UMAP work in tasks such as the TNBC Fig. 4 activity maps and healthy-breast Fig. 3C clustering), while normalization=TRUE scales activity by the number of edges in the PPIN. For custom biological processes, supply a gene ID list and the packaged full Human PPIN adjacency list (full_human_ppin) to build_ppin. Assemble the resulting per-cell activity vectors into a cells x process table (dplyr/magrittr) for downstream analyses such as Mann-Whitney tests between cell types, Pearson correlation matrices, or UMAP embedding.

## Related tools

- **ORIGINS2** (Core R package providing the activity() scoring function, 18 preloaded BP adjacency edge sets, full_human_ppin, and build_ppin() for custom PPINs) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Used to install ORIGINS2 from its GitHub repository (remotes::install_github("danielasenraoka/ORIGINS2")))
- **dplyr** (Data manipulation for assembling per-cell activity vectors into cells x process activity tables (Imports dependency of ORIGINS2))
- **magrittr** (Piping (%>%) for the dplyr-based assembly of activity tables (Imports dependency of ORIGINS2))
- **R** (Runtime environment; ORIGINS2 is freely available as an open source R package for scRNA-seq cell annotation)

## Examples

```
library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- With normalization=FALSE, each output value must equal the sum of all PPIN edge contributions for that cell (finite, non-negative per-cell scores); with normalization=TRUE, activity is normalized by the number of edges of the PPIN.
- The output vector length must equal the number of cells (columns) in the input expression matrix.
- Downstream sanity checks from source tasks: UMAP pseudotime values lie within a valid [0, 1] range, and Pearson correlation matrices of BP activities (e.g. inflammatory/immune vs replication/differentiation groups) reproduce the expected anti-correlated block structure.
- Statistical replication check: Mann-Whitney (wilcox.test) p-values for basal vs L2 proliferation activity must differ depending on whether negative-regulator edges are retained in the PPIN (significant at 0.05/0.01 with regulators included, non-significant when excluded).
- Custom build_ppin outputs should contain only edges present in full_human_ppin and restricted to the supplied genes_bp gene ID list.

## Limitations

- Results depend on the normalization flag: using normalization=FALSE (edge-sum) versus TRUE (edge-count-normalized) changes the activity scale and can flip statistical conclusions (e.g. the basal vs L2 proliferation difference in the healthy breast sample was significant only when negative regulators were included).
- Only 18 BP edge sets are preloaded; processes outside these require a custom gene ID list and the packaged full Human PPIN, so coverage is bounded by the underlying human PPI network.
- The output is a single per-cell scalar per PPIN — it does not resolve within-process heterogeneity beyond what the edge sums capture.

## Evidence
