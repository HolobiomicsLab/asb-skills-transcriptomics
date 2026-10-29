---
name: biological-process-edge-set-selection
description: Use when computing per-cell biological-process activities from a normalized
  scRNA-seq expression matrix and you must decide which of ORIGINS2's 18 BP adjacency
  edge sets (or a custom PPIN) to use for a given research question.
when_to_use_negative:
- The analysis target is already a per-cell activity or feature table — edge-set selection
  is only the precursor to running activity().
- The process of interest is not a biological process representable as a PPIN gene
  list and no gene ID list (genes_bp) is available for build_ppin.
- The input expression matrix is unnormalized or not gene-by-cell, since activity()
  expects a normalized expression matrix with genes as rows and cells as columns.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: ORIGINS2
  role: R package providing the 18 pre-loaded BP adjacency edge sets, the build_ppin
    function, and the full_human_ppin adjacency list used to select or construct PPINs
    for activity computation
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: R
  role: Runtime environment in which ORIGINS2 edge sets are loaded and passed to activity()
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Used to install ORIGINS2 from GitHub before its edge sets are available
- name: dplyr
  role: Imported by ORIGINS2 and used downstream to handle/dplyr-assemble the per-BP
    activity vectors
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:5cdc7d417dc725564cebe9dc442a5667cf5d69c523023626c762c67358324c27
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
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# biological-process-edge-set-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects the appropriate protein-protein interaction network (PPIN) edge sets — the 18 biological-process adjacency edge sets shipped with the ORIGINS2 R package, or custom PPINs built with build_ppin from full_human_ppin — to serve as input to the activity() function for per-cell BP activity computation in scRNA-seq analysis.

## When to use

Use when computing per-cell biological-process activities from a normalized scRNA-seq expression matrix and you must decide which of ORIGINS2's 18 BP adjacency edge sets (or a custom PPIN) to use for a given research question.

## When NOT to use

- The analysis target is already a per-cell activity or feature table — edge-set selection is only the precursor to running activity().
- The process of interest is not a biological process representable as a PPIN gene list and no gene ID list (genes_bp) is available for build_ppin.
- The input expression matrix is unnormalized or not gene-by-cell, since activity() expects a normalized expression matrix with genes as rows and cells as columns.

## Inputs

- ORIGINS2 pre-loaded BP adjacency edge sets (data frames of PPIN edges for 18 biological processes)
- Gene ID list (genes_bp) for a custom biological process of interest
- full_human_ppin — complete Human PPIN adjacency list included in the ORIGINS2 package
- Normalized scRNA-seq expression matrix (rows = genes, columns = cells) that the chosen edge sets will be applied to

## Outputs

- A set of selected adjacency_edges data frames (one per biological process) ready to pass to activity()
- Optionally, a custom PPIN adjacency data frame produced by build_ppin

## How to apply

Load ORIGINS2 (library(ORIGINS2)) and inspect its 18 pre-loaded BP adjacency edge sets, e.g. differentiation_edges, proliferation_edges, immune_response_edges, dna_replication_edges, cell_cycle_edges, acute/chronic_inflammatory_response_edges, hormone_secretion_edges. Choose the BP edge sets matching the biological programs under study — for example, the article's cross-BP correlation analyses combined the inflammatory/immune edge sets (acute_inflammatory_response_edges, chronic_inflammatory_response_edges, inflammatory_response_edges, immune_response_edges, innate_immune_response_edges, adaptive_immune_response_edges, humoral_immune_response_edges) with the replication/differentiation sets (dna_replication_edges, dna_repair_edges, cell_cycle_edges, mitotic_cell_cycle_edges, differentiation_edges, stem_differentiation_edges, proliferation_edges, stem_proliferation_edges). If the BP of interest is not among the pre-loaded sets, build a custom PPIN with build_ppin(genes_bp, ppi_edges_complete = full_human_ppin), where genes_bp is the gene ID list for the process and full_human_ppin is the complete Human PPIN adjacency list included in the package. Each selected adjacency_edges data frame is then passed to activity(expression_matrix, adjacency_edges, normalization); set normalization=FALSE to get raw sums of PPIN edges or TRUE to normalize by the number of edges (used as a sensitivity setting).

## Related tools

- **ORIGINS2** (R package providing the 18 pre-loaded BP adjacency edge sets, the build_ppin function, and the full_human_ppin adjacency list used to select or construct PPINs for activity computation) — https://github.com/danielasenraoka/ORIGINS2
- **R** (Runtime environment in which ORIGINS2 edge sets are loaded and passed to activity()) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Used to install ORIGINS2 from GitHub before its edge sets are available)
- **dplyr** (Imported by ORIGINS2 and used downstream to handle/dplyr-assemble the per-BP activity vectors)

## Examples

```
library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE); custom_ppin <- build_ppin(genes_bp, ppi_edges_complete = full_human_ppin)
```

## Evaluation signals

- Each selected adjacency_edges object is a data frame of PPIN edges present in the ORIGINS2 namespace (e.g. differentiation_edges, immune_response_edges) after library(ORIGINS2).
- A custom PPIN built with build_ppin uses full_human_ppin as ppi_edges_complete and contains only edges whose genes appear in the supplied genes_bp list.
- The number of distinct BP edge sets selected matches the BP panel of the analysis (e.g. 18 sets for the cross-BP correlation heatmap of the TNBC sample, yielding an 18 x 18 correlation matrix).
- Running activity() with each selected edge set on the same expression matrix returns one activity value per cell (vector length equals number of cells).
- Sensitivity check: activity computed with normalization=FALSE (raw edge sums) and normalization=TRUE (divided by the number of PPIN edges) on the same edge set produces concordant per-cell activity profiles.

## Limitations

- Only 18 BP adjacency edge sets are pre-loaded in ORIGINS2; other processes require a gene ID list and build_ppin against full_human_ppin, and activity() is restricted to the full Human PPIN context.
- normalization=FALSE returns raw sums of PPIN edges, so activity magnitudes are not directly comparable across edge sets of different sizes; use normalization=TRUE to normalize by the number of edges.
- Interpretation of BP activity patterns is sample-context dependent: inflammatory/immune and replication/differentiation BPs appear anti-correlated in the healthy breast sample but not in the TNBC sample.

## Evidence

- [readme] Adjacency edges associated with 18 BP already loaded within the package.: "Adjacency edges associated with 18 BP already loaded within the package."
- [readme] build_ppin with full_human_ppin: "Build any PPIN associated with any gene ID list using the function build_ppin(genes_bp, ppi_edges_complete = full_human_ppin)."
