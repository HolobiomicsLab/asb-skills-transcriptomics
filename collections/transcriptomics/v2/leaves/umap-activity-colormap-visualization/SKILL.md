---
name: umap-activity-colormap-visualization
description: Use when scRNA-seq cells already have a UMAP embedding and per-cell activity
  scores from a PPIN (with or without negative-regulator edges) need to be visualized
  as a color scale and compared across cell-type populations.
when_to_use_negative:
- The input is raw (not normalized) single-cell expression data — the activity() call
  expects a normalized expression matrix
- No precomputed UMAP embedding exists for the cells and the goal is dimensionality
  reduction rather than activity visualization
- The biological process of interest has no corresponding PPIN edge list (neither
  bundled in ORIGINS2 nor buildable from a gene ID list via build_ppin with full_human_ppin)
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ORIGINS2
  role: R package supplying the activity() function to compute per-cell PPIN edge-activity
    sums, bundled process edge lists (differentiation_edges), and full_human_ppin
    for custom network construction
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Used to install ORIGINS2 from GitHub via remotes::install_github()
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: dplyr
  role: Used (with magrittr pipe) to filter/prune edges exclusive to negative regulators
    from the differentiation edge list before recomputing activity
- name: magrittr
  role: Provides the %>% pipe used with dplyr filtering of PPIN edges
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:4338b53a197e0d4ea43c5319e079be0f7f8194294a754651d2865c34a64342ae
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
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# umap-activity-colormap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Overlay per-cell PPIN activity scores (e.g., cell differentiation GO:0030154 computed with the ORIGINS2 activity() function) as color scales on a precomputed UMAP embedding of scRNA-seq cells, and compare probability densities of activity per cell type to assess how network composition changes the visualization.

## When to use

Use when scRNA-seq cells already have a UMAP embedding and per-cell activity scores from a PPIN (with or without negative-regulator edges) need to be visualized as a color scale and compared across cell-type populations.

## When NOT to use

- The input is raw (not normalized) single-cell expression data — the activity() call expects a normalized expression matrix
- No precomputed UMAP embedding exists for the cells and the goal is dimensionality reduction rather than activity visualization
- The biological process of interest has no corresponding PPIN edge list (neither bundled in ORIGINS2 nor buildable from a gene ID list via build_ppin with full_human_ppin)

## Inputs

- Normalized expression matrix (genes as rows, cells as columns), e.g. for healthy breast sample ind4
- Per-cell type labels (L1, L2, basal)
- PPIN edge list for the biological process (e.g., differentiation_edges from ORIGINS2)
- Precomputed UMAP embedding coordinates for the same cells

## Outputs

- Per-cell activity score vector(s) (unnormalized sums of PPIN edge activities)
- UMAP scatter plots colored by activity, with and without negative-regulator edges
- Probability density plots of activity distributions per cell type (exported as e.g. differentiation_activity_umap_density_ind4.png)
- Activity score tables (one per edge-set variant)

## How to apply

Load the normalized expression matrix (genes as rows, cells as columns) with per-cell type labels (e.g., L1, L2, basal for the healthy breast sample ind4) and a bundled edge list such as differentiation_edges from ORIGINS2. Compute unnormalized per-cell activity via activity(expression_matrix, differentiation_edges, normalization = FALSE), which returns the sum of all PPIN edge activities per cell; optionally prune edges exclusive to negative regulators using dplyr/magrittr (%>%) filtering and recompute to compare absolute vs. relative values. Overlay both activity score vectors as continuous color scales on the precomputed UMAP coordinates of the same cells, then estimate and plot probability densities of the two activity distributions per cell type. Correct application is judged by the two color maps preserving similar relative (per cell type) patterns while showing lower absolute values when negative regulators are excluded.

## Related tools

- **ORIGINS2** (R package supplying the activity() function to compute per-cell PPIN edge-activity sums, bundled process edge lists (differentiation_edges), and full_human_ppin for custom network construction) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Used to install ORIGINS2 from GitHub via remotes::install_github()) — https://github.com/danielasenraoka/ORIGINS2
- **dplyr** (Used (with magrittr pipe) to filter/prune edges exclusive to negative regulators from the differentiation edge list before recomputing activity)
- **magrittr** (Provides the %>% pipe used with dplyr filtering of PPIN edges)

## Examples

```
library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- The unnormalized activity output equals the sum of all PPIN edge activities per cell (normalization = FALSE), so values should drop in absolute magnitude when negative-regulator edges are removed
- Relative ordering of activity across cell types (L1 vs. L2 vs. basal) should be preserved between the full and negative-regulator-pruned color maps
- Each cell in the expression matrix must map one-to-one to a UMAP embedding point and a type label before coloring
- The activity score vector length must equal the number of cells (columns) in the expression matrix
- Density plots should show distinguishable per-cell-type activity distributions matching the color-scale patterns

## Limitations

- Excluding edges exclusive to negative regulators lowers absolute activity values, so absolute scores are not comparable across edge-set variants — only relative per-cell-type patterns are
- The activity score depends on the normalization flag: normalization = TRUE normalizes by the number of PPIN edges, normalization = FALSE does not; choosing inconsistently changes the color scale
- The method requires the PPIN edges and gene IDs to be human (full_human_ppin); non-human scRNA-seq data needs gene ID mapping before use

## Evidence
