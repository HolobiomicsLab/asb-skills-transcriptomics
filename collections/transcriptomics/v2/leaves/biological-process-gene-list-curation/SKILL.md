---
name: biological-process-gene-list-curation
description: Use when you have a gene ID list associated with a biological process
  of interest and need to derive the corresponding PPIN edge set (adjacency list)
  for downstream PPIN activity scoring in ORIGINS2.
when_to_use_negative:
- The biological process of interest is one of the 18 BP already shipped with the
  package (e.g. differentiation_edges, acute_inflammatory_response_edges) — use the
  pre-built adjacency list directly instead.
- You need to compute network activity scores from an expression matrix — that is
  the separate activity(expression_matrix, adjacency_edges, normalization) step, not
  PPIN construction.
- Your gene identifiers cannot be matched to the node IDs used in the full_human_ppin
  edge list.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: ORIGINS2
  role: R package providing build_ppin() and the bundled full_human_ppin adjacency
    list used to construct the custom PPIN
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: Imported dependency used to filter the full human PPIN edge list to the gene-list
    subset
- name: magrittr
  role: Imported dependency providing the %>% pipe used in the edge-subsetting operations
- name: remotes
  role: Used to install ORIGINS2 from GitHub via remotes::install_github()
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:0832460e9509a71fa73515c8e81285876d606247aafef2d321ace3635758fa24
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
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# biological-process-gene-list-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curates a custom protein-protein interaction network (PPIN) for an arbitrary biological process or user-supplied gene list by overlaying the gene IDs onto the full Human PPIN adjacency list (full_human_ppin) and keeping only edges whose both endpoints belong to the list, as implemented in the ORIGINS2 build_ppin function.

## When to use

Use when you have a gene ID list associated with a biological process of interest and need to derive the corresponding PPIN edge set (adjacency list) for downstream PPIN activity scoring in ORIGINS2.

## When NOT to use

- The biological process of interest is one of the 18 BP already shipped with the package (e.g. differentiation_edges, acute_inflammatory_response_edges) — use the pre-built adjacency list directly instead.
- You need to compute network activity scores from an expression matrix — that is the separate activity(expression_matrix, adjacency_edges, normalization) step, not PPIN construction.
- Your gene identifiers cannot be matched to the node IDs used in the full_human_ppin edge list.

## Inputs

- genes_bp: gene ID list associated with the biological process of interest
- ppi_edges_complete: full Human PPIN adjacency list (full_human_ppin, bundled in ORIGINS2 as a data frame of edges)
- optional reference BP edges data frame (e.g. differentiation_edges, cell_cycle_edges) for validation

## Outputs

- A subset PPIN adjacency edge data frame containing only interactions whose both nodes are in the supplied gene list
- Exported edge list file (e.g. reconstructed_bp_ppin_edges.csv) for use as adjacency_edges input to activity()

## How to apply

Load ORIGINS2 (library(ORIGINS2)), which bundles the full Human PPIN adjacency list as full_human_ppin. Call build_ppin(genes_bp, ppi_edges_complete = full_human_ppin), where genes_bp is the gene ID list for the biological process of interest and ppi_edges_complete is the complete PPIN adjacency edge data frame. The function subsets the full Human PPIN edge list (using dplyr/magrittr %>% operations) so that both endpoint genes of each retained edge are members of the supplied gene list. Validate the result by comparing the reconstructed edge set against a pre-built reference such as differentiation_edges or acute_inflammatory_response_edges shipped with the package, and export the subset adjacency list (e.g. reconstructed_bp_ppin_edges.csv).

## Related tools

- **ORIGINS2** (R package providing build_ppin() and the bundled full_human_ppin adjacency list used to construct the custom PPIN) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (Imported dependency used to filter the full human PPIN edge list to the gene-list subset)
- **magrittr** (Imported dependency providing the %>% pipe used in the edge-subsetting operations)
- **remotes** (Used to install ORIGINS2 from GitHub via remotes::install_github())

## Examples

```
library(ORIGINS2); custom_edges <- build_ppin(genes_bp, ppi_edges_complete = full_human_ppin)
```

## Evaluation signals

- Row-by-row comparison of the reconstructed edge set against the corresponding pre-built BP edges dataset shipped with ORIGINS2 must match exactly (output_matches_reference).
- Every edge in the output has both endpoint genes present in the supplied gene list (both-endpoint membership invariant).
- The output CSV exists and its row count equals the expected edge count of the subset PPIN.
- The resulting edge data frame is schema-compatible as the adjacency_edges argument of activity().

## Limitations

- Custom PPINs can only contain edges already present in the bundled full_human_ppin; interactions absent from the full Human PPIN cannot be recovered.
- Genes in the user's list that lack interactions in full_human_ppin yield no edges, so the resulting network may be smaller than the gene list suggests.

## Evidence
