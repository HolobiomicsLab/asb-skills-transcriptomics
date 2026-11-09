---
name: protein-protein-interaction-network-subsetting
description: Use when you need a PPIN edge list restricted to a specific biological
  process or custom gene list, to feed into PPIN activity scoring of scRNA-seq expression
  matrices.
when_to_use_negative:
- The biological process of interest already has a pre-built PPIN loaded in ORIGINS2
  (18 BP are included, e.g. differentiation_edges, cell_cycle_edges, dna_repair_edges)
  — use those directly instead of rebuilding.
- You need the complete human interactome rather than a process-specific subnetwork
  — use full_human_ppin directly.
- The input is a normalized expression matrix needing activity scoring rather than
  a gene list needing an edge set — use the activity() function instead.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3170
tools:
- name: ORIGINS2
  role: Provides build_ppin, the bundled full_human_ppin adjacency list, and pre-built
    *_edges datasets for validation
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: Used (Imports) for filtering the full PPIN edge list when reconstructing the
    subsetting logic
- name: magrittr
  role: Provides the %>% pipe used in dplyr-based reconstruction of the build_ppin
    subsetting
- name: remotes
  role: Needed to install ORIGINS2 from GitHub via remotes::install_github
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:2efbf76366c9568a09a92f65402b8ac0170ed96e266e6292c162bf55d3c443b4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# protein-protein-interaction-network-subsetting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a custom protein-protein interaction network (PPIN) for an arbitrary biological process or user-supplied gene list by subsetting edges of the bundled full human PPIN (full_human_ppin) in the ORIGINS2 R package, using the build_ppin function.

## When to use

Use when you need a PPIN edge list restricted to a specific biological process or custom gene list, to feed into PPIN activity scoring of scRNA-seq expression matrices.

## When NOT to use

- The biological process of interest already has a pre-built PPIN loaded in ORIGINS2 (18 BP are included, e.g. differentiation_edges, cell_cycle_edges, dna_repair_edges) — use those directly instead of rebuilding.
- You need the complete human interactome rather than a process-specific subnetwork — use full_human_ppin directly.
- The input is a normalized expression matrix needing activity scoring rather than a gene list needing an edge set — use the activity() function instead.

## Inputs

- genes_bp: a gene ID list associated with the biological process of interest
- ppi_edges_complete: the full Human PPIN adjacency list (full_human_ppin data frame included in ORIGINS2)

## Outputs

- A subset PPIN adjacency list (edge data frame) containing only protein-protein interactions whose nodes are included in the supplied gene list, usable as adjacency_edges in activity()

## How to apply

Load the ORIGINS2 package (library(ORIGINS2)) and supply two inputs to build_ppin: genes_bp, the gene ID list associated with the biological process of interest, and ppi_edges_complete = full_human_ppin, the complete Human PPIN adjacency list bundled with the package. The function overlays the gene list onto the full human PPIN and keeps only those edges whose two endpoint genes are both members of the supplied gene list, which can be reproduced with dplyr/magrittr (%>%) filtering of the edge data frame. The result is a subset adjacency list (data frame of edges) specific to the biological process, suitable as the adjacency_edges argument of the activity() function. Validate by reconstructing the edge set for a pre-built BP (e.g. differentiation_edges or cell_cycle_edges) and comparing it row-by-row against the shipped dataset.

## Related tools

- **ORIGINS2** (Provides build_ppin, the bundled full_human_ppin adjacency list, and pre-built *_edges datasets for validation) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (Used (Imports) for filtering the full PPIN edge list when reconstructing the subsetting logic)
- **magrittr** (Provides the %>% pipe used in dplyr-based reconstruction of the build_ppin subsetting)
- **remotes** (Needed to install ORIGINS2 from GitHub via remotes::install_github)

## Examples

```
library(ORIGINS2); bp_edges <- build_ppin(genes_bp, ppi_edges_complete = full_human_ppin)
```

## Evaluation signals

- Row-by-row comparison of the reconstructed edge set against the corresponding pre-built BP edges dataset shipped with ORIGINS2 (e.g. differentiation_edges) shows exact agreement (output_matches_reference).
- Every edge in the output has both endpoint genes present in the supplied gene list (subset invariant).
- Edge count of the output is less than or equal to the edge count of full_human_ppin and equals the number of full-PPIN edges with both endpoints in genes_bp.
- Output is a data frame of edges in the same schema as the *_edges datasets, so it is accepted directly by activity(expression_matrix, adjacency_edges, FALSE).
- Written CSV (e.g. reconstructed_bp_ppin_edges.csv) exists with the expected edge-count rows.

## Limitations

- The subsetting is purely a filter of full_human_ppin: it cannot add interactions absent from the bundled full Human PPIN, so a custom PPIN is only as complete as full_human_ppin.
- Quality of the resulting PPIN depends entirely on the user-supplied gene ID list (genes_bp); the function does not curate or validate the gene list.
- Identifiers in genes_bp must match the gene ID convention of full_human_ppin, otherwise edges will not match.

## Evidence
