---
name: tumor-microenvironment-cluster-annotation
description: Use when scRNA-seq cells form one transcriptional cluster (e.g., tumor
  cells in a TNBC UMAP) but you need to reveal heterogeneous biological programs —
  such as mitotic cell cycle, DNA replication, DNA repair, or immune response — active
  in different regions of the cluster.
when_to_use_negative:
- Your input is already a feature/footprint table of pathway scores per cell — activity()
  expects a normalized gene-by-cell expression matrix.
- You need cluster discovery/de novo annotation of cells rather than characterizing
  heterogeneity within known, already-defined clusters (the article uses it after
  UMAP grouping exists).
- The gene identifiers of your expression matrix are not compatible with the Human
  PPIN edge lists (full_human_ppin) or the process edge sets bundled with ORIGINS2.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: ORIGINS2
  role: 'Core R package: provides activity() to compute PPIN activity per cell, pre-built
    process edge sets, build_ppin() for custom PPINs, and full_human_ppin adjacency
    list'
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Used to install the ORIGINS2 package from GitHub via remotes::install_github()
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: R
  role: Runtime environment in which ORIGINS2 activity scores are computed and UMAP/ggplot2
    visualization is assembled
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: ORIGINS2 import dependency for data manipulation of expression matrices and
    score tables
- name: magrittr
  role: ORIGINS2 import dependency for pipeline-style operations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:1ff04ac18483363d845b139df504da8aea639e9aaf9ea6c9dccd0ad3e0519682
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
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tumor-microenvironment-cluster-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-cell activity of protein-protein interaction networks (PPINs) for biological processes on scRNA-seq data using the ORIGINS2 R package, then visualize scores on a UMAP embedding to resolve heterogeneous programs within a single tumor cell cluster, as done for the TNBC sample TN-0135.

## When to use

Use when scRNA-seq cells form one transcriptional cluster (e.g., tumor cells in a TNBC UMAP) but you need to reveal heterogeneous biological programs — such as mitotic cell cycle, DNA replication, DNA repair, or immune response — active in different regions of the cluster.

## When NOT to use

- Your input is already a feature/footprint table of pathway scores per cell — activity() expects a normalized gene-by-cell expression matrix.
- You need cluster discovery/de novo annotation of cells rather than characterizing heterogeneity within known, already-defined clusters (the article uses it after UMAP grouping exists).
- The gene identifiers of your expression matrix are not compatible with the Human PPIN edge lists (full_human_ppin) or the process edge sets bundled with ORIGINS2.

## Inputs

- Normalized scRNA-seq expression matrix (matrix or data frame; rows = genes, columns = cells, e.g., TNBC sample TN-0135)
- PPIN adjacency edge lists for biological processes (pre-built ORIGINS2 objects like dna_repair_edges, or custom edges built with build_ppin() against full_human_ppin)
- Cell cluster labels (e.g., TC/CAF/TAM/T-cell annotations) keyed to cell barcodes

## Outputs

- Per-cell PPIN activity vector per biological process (sum of all PPIN edges when normalization = FALSE)
- Cells x processes activity score table keyed to cell barcodes
- 2D UMAP embedding of the activity score table with cluster labels overlaid (e.g., ppin_activity_umap.png)

## How to apply

Load the normalized expression matrix (rows = genes, columns = cells) into R and call library(ORIGINS2). For each biological process of interest, invoke activity(expression_matrix, <process>_edges, normalization = FALSE) — setting normalization = FALSE so each score is the raw sum of all PPIN edge contributions; pre-built edge sets include differentiation_edges, proliferation_edges, migration_edges, mitotic_cell_cycle_edges, dna_replication_edges, dna_repair_edges, stem_differentiation_edges, innate_immune_response_edges, and adaptive_immune_response_edges. Assemble the resulting per-cell activity vectors into a cells x processes score table keyed to the TNBC cell barcodes (TC, CAF, TAM, T-cell labels). Run UMAP (e.g., via uwot/umap-learn) on this activity score table to embed cells in 2D, then overlay cluster labels with ggplot2. Interpretation: in the TN-0135 analysis, mitotic cell cycle, DNA replication and DNA repair scores were correlated and highly activated in the lower-left region of the tumor cell cluster while innate immune response affected TC and macrophages, demonstrating that tumor cells keep diverse biological programs active in a differentiated manner within the cluster.

## Related tools

- **ORIGINS2** (Core R package: provides activity() to compute PPIN activity per cell, pre-built process edge sets, build_ppin() for custom PPINs, and full_human_ppin adjacency list) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Used to install the ORIGINS2 package from GitHub via remotes::install_github()) — https://github.com/danielasenraoka/ORIGINS2
- **R** (Runtime environment in which ORIGINS2 activity scores are computed and UMAP/ggplot2 visualization is assembled) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (ORIGINS2 import dependency for data manipulation of expression matrices and score tables)
- **magrittr** (ORIGINS2 import dependency for pipeline-style operations)

## Examples

```
library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- Each activity() output is a finite per-cell vector; with normalization = FALSE its value equals the sum of all PPIN edge contributions for that cell (verify no NaNs and spot-check the sum against the edge list).
- The assembled score table dimensions are (number of cells) x (number of processes) with rownames matching the TNBC cell barcodes exactly.
- The UMAP renders all four cluster types (TC, CAF, TAM, T-cell) with their assigned labels on the activity-derived embedding.
- Within-cluster check: biological programs (e.g., mitotic cell cycle, DNA replication, DNA repair) show spatially differentiated, correlated high activity in specific regions of the tumor cell cluster rather than uniform scores.
- The figure is saved reproducibly (e.g., ppin_activity_umap.png) and matches the article's finding that a single UMAP group keeps diverse biological programs active in a differentiated manner.

## Limitations

- PPIN-based activity depends on the bundled Human PPIN (full_human_ppin) and pre-built process edge sets; non-human or poorly covered processes require building a custom PPIN via build_ppin(genes_bp, ppi_edges_complete = full_human_ppin).
- normalization = FALSE yields scores that are raw sums of PPIN edges and thus not directly comparable across processes of different edge counts; normalization = TRUE rescales by PPIN edge number, so the choice affects cross-process comparisons.
- Interpretation of heterogeneity (e.g., lower-left tumor cell region) is tied to the UMAP layout, which is sensitive to embedding parameters.

## Evidence
