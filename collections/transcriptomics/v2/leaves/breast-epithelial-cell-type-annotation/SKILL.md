---
name: breast-epithelial-cell-type-annotation
description: Use when a normalized scRNA-seq expression matrix (genes as rows, cells
  as columns) from breast tissue has per-cell type labels and the goal is to characterize
  or annotate cell populations by PPIN activity of biological processes rather than
  marker genes alone.
when_to_use_negative:
- Input is raw (not normalized) counts — activity() expects a normalized expression
  matrix
- The goal is cluster discovery from scratch rather than characterizing already-annotated
  cell populations (cell labels and UMAP embedding are assumed precomputed)
- The biological process of interest has no corresponding gene list and no edges exist
  in full_human_ppin to build a custom PPIN
edam_operation: http://edamontology.org/operation_3934
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: ORIGINS2
  role: R package providing activity() to compute PPIN activity per cell, build_ppin()
    to create custom process PPINs, and bundled adjacency edges (e.g., differentiation_edges)
    and full_human_ppin
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Used to install ORIGINS2 from GitHub via remotes::install_github('danielasenraoka/ORIGINS2')
- name: dplyr
  role: Used (with magrittr %>%) to filter/prune PPIN edges exclusive to negative
    regulators before recomputing activity
- name: magrittr
  role: Pipe operator (%>%) for edge-filtering steps on the differentiation edge list
- name: R
  role: Execution environment for ORIGINS2 activity computation, UMAP overlays, density
    plots, and stats::wilcox.test comparisons
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:39b4f6edf670455da07c2dfce300f9bd8fcca4d644837445065fb003090aeab2
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
  edam_operation: http://edamontology.org/operation_3934
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# breast-epithelial-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates breast epithelial cells (L1, L2, basal) in scRNA-seq data by computing protein-protein interaction network (PPIN) activity scores for biological processes (e.g., GO:0030154 cell differentiation, GO:0008283 cell proliferation) using the ORIGINS2 R package, then overlaying activity distributions on UMAP embeddings and testing them across cell types.

## When to use

Use when a normalized scRNA-seq expression matrix (genes as rows, cells as columns) from breast tissue has per-cell type labels and the goal is to characterize or annotate cell populations by PPIN activity of biological processes rather than marker genes alone.

## When NOT to use

- Input is raw (not normalized) counts — activity() expects a normalized expression matrix
- The goal is cluster discovery from scratch rather than characterizing already-annotated cell populations (cell labels and UMAP embedding are assumed precomputed)
- The biological process of interest has no corresponding gene list and no edges exist in full_human_ppin to build a custom PPIN

## Inputs

- Normalized scRNA-seq expression matrix (rows = genes, columns = cells), e.g. the healthy breast sample ind4
- Per-cell type labels (L1, L2, basal)
- Precomputed UMAP embedding of the cells
- PPIN adjacency edges for the biological process of interest (e.g., differentiation_edges, proliferation/cell_cycle_edges) or a gene ID list plus full_human_ppin for build_ppin()

## Outputs

- Per-cell PPIN activity score vectors (unnormalized sums of PPIN edge activities)
- UMAP activity maps colored by process activity, with and without negative-regulator edges
- Probability density plots of activity per cell type (L1, L2, basal)
- Mann-Whitney U test p-values comparing activity distributions across cell types (e.g., basal vs L2)
- Exported activity score tables (e.g., CSV) and figures such as differentiation_activity_umap_density_ind4.png

## How to apply

Install ORIGINS2 from GitHub with remotes::install_github('danielasenraoka/ORIGINS2') and load it with library(ORIGINS2). Provide a normalized expression matrix (rows = genes, columns = cells), per-cell type labels (L1, L2, basal for the healthy breast sample ind4), and either a bundled process PPIN (e.g., differentiation_edges) or a custom one built with build_ppin(genes_bp, ppi_edges_complete = full_human_ppin). Compute per-cell activity with activity(expression_matrix, adjacency_edges, normalization = FALSE), which returns the unnormalized sum of all PPIN edge activities per cell; set normalization = TRUE to normalize by the number of PPIN edges. Overlay activity scores as a color scale on a precomputed UMAP embedding and compare probability densities of activity per cell type (L1, L2, basal). Because negative-regulator edges can overestimate activity, optionally prune edges exclusive to negative regulators (dplyr/magrittr %>%) and rerun to check whether relative results and group differences (e.g., two-sided Mann-Whitney via stats::wilcox.test at the 0.01 significance level) are preserved.

## Related tools

- **ORIGINS2** (R package providing activity() to compute PPIN activity per cell, build_ppin() to create custom process PPINs, and bundled adjacency edges (e.g., differentiation_edges) and full_human_ppin) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Used to install ORIGINS2 from GitHub via remotes::install_github('danielasenraoka/ORIGINS2'))
- **dplyr** (Used (with magrittr %>%) to filter/prune PPIN edges exclusive to negative regulators before recomputing activity)
- **magrittr** (Pipe operator (%>%) for edge-filtering steps on the differentiation edge list)
- **R** (Execution environment for ORIGINS2 activity computation, UMAP overlays, density plots, and stats::wilcox.test comparisons)

## Examples

```
diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- activity() with normalization = FALSE returns, per cell, the sum of all PPIN edge activities; verify output length equals the number of cells in the expression matrix
- With vs. without negative-regulator edges, relative (color-scale) results on the UMAP should be similar while absolute activity values drop when negative regulators are excluded
- For proliferation (GO:0008283) in the healthy breast sample, the basal-vs-L2 Mann-Whitney p-value should differ between network variants: significant with negative regulators included, not significant at the 0.01 level when excluded
- Density plots per cell type (L1, L2, basal) should separate in a manner consistent with the UMAP color scale
- Custom PPINs built with build_ppin(genes_bp, ppi_edges_complete = full_human_ppin) should contain only edges whose endpoints are in the supplied gene list

## Limitations

- Activity magnitudes depend on negative-regulator handling: including negative-regulator edges can overestimate activity, and excluding them lowers absolute values even though relative comparisons remain similar
- Group-level significance of activity differences can change with network composition (basal vs L2 proliferation difference is significant only when negative regulators are included)
- The method assumes the expression matrix is already normalized and that cells have known type labels and a precomputed UMAP embedding

## Evidence
