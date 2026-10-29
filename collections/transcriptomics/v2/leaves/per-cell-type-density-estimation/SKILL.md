---
name: per-cell-type-density-estimation
description: Use when scRNA-seq cells carry per-cell activity scores (from ORIGINS2
  activity()) plus known cell-type labels, and you need density distributions per
  type to compare biological programs across clusters.
when_to_use_negative:
- Cells lack per-cell type annotations, so no per-type stratification of the activity
  distribution can be formed
- The goal is annotating unlabeled cells from scratch via PPIN activity rather than
  comparing densities across already-assigned types
- The expression matrix is raw counts rather than the normalized expression matrix
  activity() expects
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: ORIGINS2
  role: Provides activity() to compute per-cell PPIN activity scores that are then
    stratified into per-cell-type density distributions
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: R
  role: Language in which activity computation, UMAP overlay, and per-type density
    estimation and plotting are performed
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: Used to filter PPIN edge lists (e.g., removing edges exclusive to negative
    regulators) before recomputing activities for comparison
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: magrittr
  role: Supplies the %>% pipe used with dplyr to prune the differentiation edge list
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Needed to install ORIGINS2 from GitHub before any activity computation or
    plotting
  repo: https://github.com/danielasenraoka/ORIGINS2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:071c827d00a1dedb07ec9c1b6b3bc4fe1f93c4c9e37b7918264d7aa56faf56ea
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
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# per-cell-type-density-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate and plot per-cell-type probability densities of computed PPIN activity scores (e.g., GO:0030154 cell differentiation) so that distributions can be compared across annotated cell populations (L1, L2, basal) in a scRNA-seq sample, optionally overlaid on a precomputed UMAP embedding.

## When to use

Use when scRNA-seq cells carry per-cell activity scores (from ORIGINS2 activity()) plus known cell-type labels, and you need density distributions per type to compare biological programs across clusters.

## When NOT to use

- Cells lack per-cell type annotations, so no per-type stratification of the activity distribution can be formed
- The goal is annotating unlabeled cells from scratch via PPIN activity rather than comparing densities across already-assigned types
- The expression matrix is raw counts rather than the normalized expression matrix activity() expects

## Inputs

- Per-cell activity score vector returned by activity() (one value per cell)
- Normalized expression matrix (genes as rows, cells as columns)
- PPIN edge data frame (e.g., differentiation_edges, or a filtered version with negative-regulator edges removed)
- Per-cell type labels (e.g., L1, L2, basal)
- Precomputed UMAP embedding of the cells

## Outputs

- Per-cell-type probability density plots of activity scores
- UMAP overlay of activity scores as a color scale
- Activity score tables exported alongside the figure (e.g., differentiation_activity_umap_density_ind4.png)

## How to apply

Compute per-cell activity with ORIGINS2's activity(expression_matrix, adjacency_edges, normalization), choosing normalization = FALSE to obtain the raw sum of all PPIN edge activities per cell, or normalization = TRUE to normalize by the number of PPIN edges when comparing across PPINs of different sizes. Join the returned activity vector to the per-cell type labels (e.g., L1, L2, basal from the healthy breast sample ind4). Overlay the scores as a color scale on the precomputed UMAP embedding, then estimate and plot probability densities of the activity distribution separately per cell type. Rerun the same density estimation after filtering out edges exclusive to negative regulators (using dplyr/magrittr %>% on the edge list) to judge whether the relative ordering between cell types is preserved — relative results should be similar while absolute values drop when negative regulators are excluded.

## Related tools

- **ORIGINS2** (Provides activity() to compute per-cell PPIN activity scores that are then stratified into per-cell-type density distributions) — https://github.com/danielasenraoka/ORIGINS2
- **R** (Language in which activity computation, UMAP overlay, and per-type density estimation and plotting are performed) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (Used to filter PPIN edge lists (e.g., removing edges exclusive to negative regulators) before recomputing activities for comparison) — https://github.com/danielasenraoka/ORIGINS2
- **magrittr** (Supplies the %>% pipe used with dplyr to prune the differentiation edge list) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Needed to install ORIGINS2 from GitHub before any activity computation or plotting) — https://github.com/danielasenraoka/ORIGINS2

## Examples

```
diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- Density curves per cell type (L1, L2, basal) are produced for both the full and the negative-regulator-filtered edge sets
- Relative ordering of activity distributions across cell types is similar with and without negative regulators, while absolute values are lower when negative regulators are excluded (expected comparison from the source card)
- With normalization = FALSE the output equals the sum of all PPIN edge activities per cell; with normalization = TRUE it is normalized by the number of edges of the PPIN
- Activity overlay on the UMAP embedding is consistent with the per-type density estimates
- The figure (differentiation_activity_umap_density_ind4.png) and the two activity score tables are exported together

## Limitations

- With normalization = FALSE, absolute activity values depend on PPIN size, so comparisons across different PPINs require normalization = TRUE (normalized by the number of edges)
- Removing edges exclusive to negative regulators changes absolute activity values even though the relative pattern across cell types is preserved, so only relative comparisons remain meaningful across edge-set variants

## Evidence
