---
name: scrna-seq-cell-annotation
description: Use when you have a normalized scRNA-seq expression matrix (rows = genes,
  columns = cells) and need per-cell activity scores of biological-process PPINs for
  cell annotation, UMAP embedding, or pseudotime inference.
when_to_use_negative:
- Input is not a normalized single-cell expression matrix (e.g. bulk RNA-seq or an
  already-computed activity/feature table) — ORIGINS2 expects gene-by-cell expression
  data
- Organism data outside the scope of the included full Human PPIN adjacency list,
  since build_ppin relies on full_human_ppin for custom processes
- You require cell-state labels from a model with a predefined number of cell types
  — this method quantifies per-cell BP activity without that requirement, so it will
  not output predefined type assignments directly
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ORIGINS2
  role: 'Core R package: computes per-cell PPIN activity for biological processes
    via activity() and builds custom PPINs via build_ppin()'
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Required to install ORIGINS2 from its GitHub repository
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: Used (with magrittr) to assemble the cells x biological-process activity matrix
- name: magrittr
  role: Piping support when assembling the per-cell BP activity matrix
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:e1d892eadc27b1c3fb4141ff2f481309672c37612b2e96781d48c76205f55c49
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# scrna-seq-cell-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate scRNA-seq cells by computing per-cell activity of protein-protein interaction networks (PPINs) associated with biological processes using the ORIGINS2 R package. Per-cell BP activities yield an activity space in which standard scRNA-seq techniques (e.g. UMAP, pseudotime) can be applied without a predefined number of cell types.

## When to use

Use when you have a normalized scRNA-seq expression matrix (rows = genes, columns = cells) and need per-cell activity scores of biological-process PPINs for cell annotation, UMAP embedding, or pseudotime inference.

## When NOT to use

- Input is not a normalized single-cell expression matrix (e.g. bulk RNA-seq or an already-computed activity/feature table) — ORIGINS2 expects gene-by-cell expression data
- Organism data outside the scope of the included full Human PPIN adjacency list, since build_ppin relies on full_human_ppin for custom processes
- You require cell-state labels from a model with a predefined number of cell types — this method quantifies per-cell BP activity without that requirement, so it will not output predefined type assignments directly

## Inputs

- Normalized scRNA-seq expression matrix or data frame (rows = genes, columns = cells)
- BP-specific PPIN adjacency edge data frame (18 preloaded, e.g. differentiation_edges, cell_cycle_edges, dna_repair_edges)
- Gene ID list (genes_bp) plus full_human_ppin adjacency list for building custom PPINs

## Outputs

- Per-cell activity vector for each biological process (one value per cell)
- Cells x biological-process activity matrix
- Low-dimensional UMAP embedding of cells in activity space
- Pseudotime scores per cell (e.g. embedding_pseudotime.csv)

## How to apply

Install the remotes package and then install ORIGINS2 from GitHub via remotes::install_github("danielasenraoka/ORIGINS2"). Load ORIGINS2 and, for each of the 18 preloaded BP edge datasets (e.g. differentiation_edges, cell_cycle_edges, acute_inflammatory_response_edges), call activity(expression_matrix, adjacency_edges, FALSE) on the normalized expression matrix; normalization = FALSE means the output vector is the sum of all PPIN edges, while TRUE normalizes by the number of PPIN edges. Assemble the per-cell activity vectors into a cells x biological-process activity matrix using dplyr/magrittr, then run UMAP on this matrix to embed cells in activity space and perform trajectory inference to assign pseudotime. For biological processes not preloaded, build a custom PPIN with build_ppin(genes_bp, ppi_edges_complete = full_human_ppin), where full_human_ppin is the complete Human PPIN adjacency list included in the package. Use the resulting activity scores/embedding to annotate cells, exploiting the reported behavior that tumor breast cells forming a single UMAP group keep diverse biological programs active in a differentiated manner.

## Related tools

- **ORIGINS2** (Core R package: computes per-cell PPIN activity for biological processes via activity() and builds custom PPINs via build_ppin()) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Required to install ORIGINS2 from its GitHub repository) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (Used (with magrittr) to assemble the cells x biological-process activity matrix)
- **magrittr** (Piping support when assembling the per-cell BP activity matrix)

## Examples

```
diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- activity() returns exactly one activity value per cell for each biological process queried
- embedding_pseudotime.csv exists, contains one row per cell with UMAP coordinates and a pseudotime column, and pseudotime values lie within the valid [0, 1] range
- Cells occupy a coherent grouped structure in the UMAP embedding, consistent with the finding that tumor breast cells form a single group in UMAP space
- build_ppin output uses the full_human_ppin adjacency list restricted to the supplied genes_bp gene ID list

## Limitations

- Only 18 BP PPINs are preloaded; other processes require build_ppin, which is limited to genes present in the full Human PPIN adjacency list
- With normalization = FALSE the activity output is the raw sum of all PPIN edge weights, so values are not directly comparable across biological processes with different edge counts
- The package must be installed from GitHub via remotes; the license is unspecified (NOASSERTION), which may affect redistribution in some workflows

## Evidence
