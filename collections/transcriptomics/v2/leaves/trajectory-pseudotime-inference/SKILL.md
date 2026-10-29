---
name: trajectory-pseudotime-inference
description: Use when you need per-cell PPIN activity scores for 18 preloaded biological
  processes and want to embed cells in activity space (UMAP) and assign pseudotime
  along differentiation.
when_to_use_negative:
- You already have a cells x features embedding and only need annotation — use cell-annotation
  approaches instead of rebuilding the activity space
- You need cell-type counts or a predefined number of states — ORIGINS2 explicitly
  does not require a predefined number of cell types or states
- Expression matrix is raw/unaligned (ORIGINS2 expects a normalized expression matrix
  with genes as rows and cells as columns)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ORIGINS2
  role: Computes per-cell PPIN activity scores for 18 biological processes via activity();
    provides build_ppin() for custom BP networks
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Installs the ORIGINS2 package from GitHub
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: Assembles per-cell activity vectors into the cells x biological-process activity
    matrix
- name: magrittr
  role: Pipes used when assembling the activity matrix
- name: R
  role: Execution environment for the ORIGINS2 workflow and downstream UMAP/trajectory
    inference
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:4c2fef71ddf8abe7c0bf3cafdbaf5dcbb82993d8710f2e171644ad995959f160
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- pseudotime-trajectory-inference
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# trajectory-pseudotime-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps scRNA-seq cells onto an ORIGINS2 per-cell biological-process activity space and performs trajectory inference (UMAP embedding + pseudotime scoring) to order cells along a differentiation path.

## When to use

Use when you need per-cell PPIN activity scores for 18 preloaded biological processes and want to embed cells in activity space (UMAP) and assign pseudotime along differentiation.

## When NOT to use

- You already have a cells x features embedding and only need annotation — use cell-annotation approaches instead of rebuilding the activity space
- You need cell-type counts or a predefined number of states — ORIGINS2 explicitly does not require a predefined number of cell types or states
- Expression matrix is raw/unaligned (ORIGINS2 expects a normalized expression matrix with genes as rows and cells as columns)

## Inputs

- Normalized scRNA-seq expression matrix (rows = genes, columns = cells)
- ORIGINS2 preloaded PPIN adjacency edge data frames (e.g. differentiation_edges, 18 BP datasets)
- Optional custom PPIN built via build_ppin(genes_bp, ppi_edges_complete = full_human_ppin)

## Outputs

- Cells x biological-process activity matrix
- UMAP embedding coordinates in activity space
- Per-cell pseudotime scores along the differentiation path
- embedding_pseudotime.csv (single table with UMAP coordinates and pseudotime)

## How to apply

Install remotes and the ORIGINS2 R package via remotes::install_github("danielasenraoka/ORIGINS2"), then load ORIGINS2. For each of the 18 preloaded BP edge datasets (e.g. differentiation_edges, proliferation_edges), compute per-cell BP activity with activity() on the normalized expression matrix (rows = genes, columns = cells), setting normalization = FALSE so the output is the sum of all PPIN edge weights rather than edge-normalized. Assemble the resulting per-cell activity vectors into a cells x biological-process activity matrix using dplyr/magrittr, then run UMAP on this activity matrix to embed cells in a low-dimensional activity space. Perform trajectory inference on the UMAP embedding to assign each cell a pseudotime score along the differentiation path. Rationale: ORIGINS2 quantifies PPIN activity for every individual cell without requiring a predefined number of cell types or states, making the activity matrix a valid basis for standard scRNA-seq analyses such as embedding and pseudotime.

## Related tools

- **ORIGINS2** (Computes per-cell PPIN activity scores for 18 biological processes via activity(); provides build_ppin() for custom BP networks) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Installs the ORIGINS2 package from GitHub) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (Assembles per-cell activity vectors into the cells x biological-process activity matrix)
- **magrittr** (Pipes used when assembling the activity matrix)
- **R** (Execution environment for the ORIGINS2 workflow and downstream UMAP/trajectory inference)

## Examples

```
library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- embedding_pseudotime.csv exists and contains one row per cell with embedding coordinates and a pseudotime column
- Pseudotime values fall within a valid [0, 1] range
- Cells occupy a coherent grouped structure in the UMAP embedding
- Tumor breast cells form a single group in the UMAP space, consistent with the reported finding
- activity() was called with normalization = FALSE so outputs are PPIN edge sums, not edge-normalized scores

## Limitations

- activity() depends on the choice of normalization parameter: FALSE gives the raw sum of all PPIN edges, TRUE normalizes by PPIN edge count — the choice changes downstream embedding
- Only 18 BP PPIN adjacency datasets are preloaded; other processes require building a custom PPIN with build_ppin() against full_human_ppin
- Interpretation caveat from the article: tumor breast cells can form a single UMAP group while still keeping diverse biological programs active in a differentiated manner within the cluster

## Evidence
