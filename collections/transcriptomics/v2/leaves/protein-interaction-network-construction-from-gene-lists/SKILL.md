---
name: protein-interaction-network-construction-from-gene-lists
description: Use when you have a gene ID list associated with a biological process
  (e.g. GO:0008283 cell proliferation) and need an adjacency-edge PPIN to compute
  per-cell network activity on scRNA-seq expression data with ORIGINS2.
when_to_use_negative:
- The biological process of interest is already covered by one of the 18 preloaded
  BP adjacency edge sets in ORIGINS2 (e.g. cell_cycle_edges, differentiation_edges)
  — use those directly instead of building a custom network.
- You need normalized per-cell activity values rather than a network — the construction
  step only produces edges; normalization is controlled later by the logical normalization
  parameter of activity().
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3170
tools:
- name: ORIGINS2
  role: Provides build_ppin() and the packaged full_human_ppin adjacency list used
    to construct the custom process-specific PPIN
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Used to install ORIGINS2 from GitHub before building networks
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: R
  role: Language in which build_ppin and the PPIN adjacency objects are used
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:c62e07cbbb2bc2fdf17005ea22280dd290a8c55a65da6cd0c3f4e4fd2e57b7c5
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Protein-protein interaction network construction from gene lists

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a custom PPIN for any biological process of interest by intersecting a supplied gene ID list (genes_bp) with the complete human PPIN adjacency list (full_human_ppin) using ORIGINS2's build_ppin function in R.

## When to use

Use when you have a gene ID list associated with a biological process (e.g. GO:0008283 cell proliferation) and need an adjacency-edge PPIN to compute per-cell network activity on scRNA-seq expression data with ORIGINS2.

## When NOT to use

- The biological process of interest is already covered by one of the 18 preloaded BP adjacency edge sets in ORIGINS2 (e.g. cell_cycle_edges, differentiation_edges) — use those directly instead of building a custom network.
- You need normalized per-cell activity values rather than a network — the construction step only produces edges; normalization is controlled later by the logical normalization parameter of activity().

## Inputs

- genes_bp: gene ID list associated with the biological process of interest (e.g. GO:0008283 cell proliferation genes)
- full_human_ppin: complete human PPIN adjacency list included within the ORIGINS2 package
- R environment with ORIGINS2 installed via remotes

## Outputs

- A PPIN adjacency-edge data frame (e.g. proliferation_edges) restricted to the genes in the input gene list, suitable as the adjacency_edges argument of activity()

## How to apply

Install ORIGINS2 from GitHub via remotes::install_github("danielasenraoka/ORIGINS2") and load it with library(ORIGINS2). Prepare the gene ID list (genes_bp) associated with the biological process of interest — for example the GO:0008283 cell proliferation genes used in the basal-vs-L2 proliferation study. Call build_ppin(genes_bp, ppi_edges_complete = full_human_ppin), where full_human_ppin is the complete human PPIN adjacency list packaged within ORIGINS2; the function retains only PPIN edges whose genes appear in the supplied list, yielding an edge adjacency object usable directly as the adjacency_edges argument of activity(). Decide whether to keep negative-regulator edges: the source task shows retaining them ('overestimated' activity) or filtering them out changes downstream Mann-Whitney significance (0.01 level) for basal vs L2 proliferation activity. Alternatively, skip construction and use one of the 18 preloaded process-specific edge sets (e.g. cell_cycle_edges, differentiation_edges) already included in the package.

## Related tools

- **ORIGINS2** (Provides build_ppin() and the packaged full_human_ppin adjacency list used to construct the custom process-specific PPIN) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Used to install ORIGINS2 from GitHub before building networks) — https://github.com/danielasenraoka/ORIGINS2
- **R** (Language in which build_ppin and the PPIN adjacency objects are used)

## Examples

```
remotes::install_github("danielasenraoka/ORIGINS2"); library(ORIGINS2); proliferation_edges <- build_ppin(genes_bp, ppi_edges_complete = full_human_ppin)
```

## Evaluation signals

- All edges in the returned adjacency object have both endpoint genes present in full_human_ppin, confirming the intersection against the complete human PPIN succeeded.
- Every gene in genes_bp that participates in an edge of full_human_ppin is represented in the constructed adjacency edges (no silent gene loss beyond PPIN coverage).
- The constructed edges object is accepted directly by activity(expression_matrix, adjacency_edges, FALSE) and returns a per-cell activity vector of length equal to the number of cells (columns of the expression matrix).
- Sensitivity check: removing negative-regulator edges from the constructed network changes downstream statistical results (per the source task, the basal-vs-L2 Mann-Whitney significance at the 0.01 level depends on negative-regulator handling), so the edge set's regulator composition should be documented.

## Limitations

- Custom PPINs are limited to edges present in the packaged full_human_ppin adjacency list; process genes without human PPI edges in that list contribute nothing.
- Unnormalized activity (normalization = FALSE) is the sum of all PPIN edge contributions and is overestimated when negative regulators are retained, which can flip the significance of cell-type activity comparisons.

## Evidence
