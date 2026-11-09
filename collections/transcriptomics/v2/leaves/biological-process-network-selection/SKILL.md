---
name: biological-process-network-selection
description: Use when you have a normalized scRNA-seq expression matrix (rows = genes,
  columns = cells) and need per-cell activity scores for a biological process such
  as cell differentiation, cell cycle, DNA repair, or immune response, or when you
  must build a custom PPIN from a gene ID list.
when_to_use_negative:
- Input is raw (unnormalized) count data — the activity() function expects a normalized
  expression matrix
- The biological process of interest is gene-set-level bulk enrichment rather than
  per-cell PPIN activity scoring
- You need cell-type annotation from marker labels rather than continuous pathway
  activity quantification
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: ORIGINS2
  role: R package providing the activity() scoring function, 18 preloaded BP edge
    datasets (adjacency_edges), build_ppin(), and the full_human_ppin adjacency list
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Used to install ORIGINS2 from its GitHub repository via remotes::install_github
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: Imported dependency used to assemble per-cell activity vectors into a cells
    x biological-process activity matrix
- name: magrittr
  role: Imported dependency for piped assembly of the activity score table
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:c2e67bc360a3374549991a3ea14ee69f5fe49b7485e5c2095a9b0297a6fd6634
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
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# biological-process-network-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Select and invoke the appropriate preloaded protein-protein interaction network (PPIN) edge dataset in the ORIGINS2 R package for the biological process of interest, then compute per-cell activity scores on a normalized scRNA-seq expression matrix. This is the core step that turns gene expression into per-cell biological-process activity vectors for downstream UMAP, clustering, or trajectory analyses.

## When to use

Use when you have a normalized scRNA-seq expression matrix (rows = genes, columns = cells) and need per-cell activity scores for a biological process such as cell differentiation, cell cycle, DNA repair, or immune response, or when you must build a custom PPIN from a gene ID list.

## When NOT to use

- Input is raw (unnormalized) count data — the activity() function expects a normalized expression matrix
- The biological process of interest is gene-set-level bulk enrichment rather than per-cell PPIN activity scoring
- You need cell-type annotation from marker labels rather than continuous pathway activity quantification

## Inputs

- Normalized scRNA-seq expression matrix (rows = genes, columns = cells), as matrix or data frame
- Preloaded ORIGINS2 adjacency edge data frame for the chosen biological process (one of 18 BP sets, e.g. differentiation_edges, cell_cycle_edges)
- Optional: gene ID list (genes_bp) and full_human_ppin adjacency list for building a custom PPIN
- Logical normalization flag (FALSE = sum of all PPIN edges; TRUE = normalized by number of edges)

## Outputs

- Per-cell activity vector (one finite score per cell) for the selected biological process
- Cells x biological-processes activity score table keyed to cell barcodes (after combining multiple activity() calls)

## How to apply

Install the ORIGINS2 package from GitHub with remotes::install_github("danielasenraoka/ORIGINS2") and load it with library(ORIGINS2). Choose the preloaded adjacency edge data frame matching your biological process — the package ships 18 BP edge sets (e.g. differentiation_edges, cell_cycle_edges, dna_repair_edges, dna_replication_edges, adaptive_immune_response_edges). Call activity(expression_matrix, adjacency_edges, normalization) on the normalized expression matrix; set normalization = FALSE to obtain the raw sum of all PPIN edge contributions, or TRUE to normalize by PPIN edge count — pick FALSE (as in the TNBC Fig. 4 reproduction and the activity-space extension) when comparing correlated module activations across cells. If no preloaded PPIN matches your process, build one with build_ppin(genes_bp, ppi_edges_complete = full_human_ppin), supplying your gene ID list and the full Human PPIN adjacency list included in the package. Assemble the resulting per-cell activity vectors into a cells x processes matrix for embedding (UMAP) or trajectory inference.

## Related tools

- **ORIGINS2** (R package providing the activity() scoring function, 18 preloaded BP edge datasets (adjacency_edges), build_ppin(), and the full_human_ppin adjacency list) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Used to install ORIGINS2 from its GitHub repository via remotes::install_github) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (Imported dependency used to assemble per-cell activity vectors into a cells x biological-process activity matrix)
- **magrittr** (Imported dependency for piped assembly of the activity score table)

## Examples

```
remotes::install_github("danielasenraoka/ORIGINS2"); library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- With normalization = FALSE, each per-cell activity value equals the sum of all PPIN edge contributions for that cell and is finite (no NA/Inf)
- Activity vector length matches the number of cells (columns) in the input expression matrix, keyed to the same cell barcodes
- The selected adjacency_edges data frame is one of the 18 BP edge sets preloaded in ORIGINS2, or a custom PPIN successfully built by build_ppin() from full_human_ppin
- Downstream UMAP of the assembled cells x processes activity table renders coherent grouped structure (e.g. tumor breast cells form a single group while keeping diverse programs active)
- Repeated activity() calls with the same inputs reproduce identical score vectors (deterministic edge-sum computation)

## Limitations

- Per-cell activity depends on the quality of the PPIN for the chosen BP; coverage is limited to the edges present in the preloaded 18 BP sets or the Human PPIN used by build_ppin
- The choice of normalization flag changes score scale: normalization = FALSE yields edge-count-dependent raw sums, so scores across processes with different PPIN sizes are not directly comparable without normalization
- The methodology was developed and demonstrated for scRNA-seq data (e.g. TNBC sample TN-0135); applicability to other data modalities is not established in the article

## Evidence
