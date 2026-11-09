---
name: minimal-spanning-tree-visualization-in-umap
description: Use when a processed scRNA-seq Seurat object already carries Louvain
  cluster assignments and reduced-dimension coordinates (PCA, UMAP, PHATE) and you
  need to infer and visually present cellular trajectories as a cluster-level MST.
when_to_use_negative:
- The dataset is not from human or mouse — SCALA's analysis currently supports only
  human and mouse datasets
- The object lacks prior clustering; Slingshot's MST is built from cluster labels
  defining initial and final states, so an unclustered expression matrix alone is
  insufficient
- The online SCALA version is used for datasets larger than 50,000 cells, where some
  steps cannot be completed (prefer the desktop version)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Slingshot
  role: Runs on cluster labels and selected reduced dimensions (e.g., first 25 PCs)
    with user-specified initial/final states to construct the cluster-level MST and
    pseudotime
- name: Seurat
  role: Provides the processed Seurat object with Louvain cluster assignments and
    PCA/UMAP reductions that feed Slingshot; requires a PHATE-support patch
  repo: https://github.com/scottgigante/seurat
- name: phateR
  role: GitHub version computes a PHATE embedding as a trajectory-friendly reduced
    dimension option for Slingshot
  repo: https://github.com/KrishnaswamyLab/phateR
- name: Monocle
  role: Computes an alternative pseudotime ordering for comparison with Slingshot
    lineages
- name: SCALA
  role: Interactive R/Shiny pipeline whose trajectory module implements this Slingshot
    + MST-on-UMAP workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:2d8ad5a2df809d0425a985e7b5fc7298a4f045b7b1f9df44602717f01fbf63e3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/scottgigante/seurat
  tools:
  - Monocle
  - SCALA
  - Seurat
  - Slingshot
  - phateR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Minimal Spanning Tree Visualization in UMAP

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a cluster-level Minimal Spanning Tree (MST) with Slingshot from single-cell cluster labels and reduced-dimension coordinates, then overlays it on a UMAP projection alongside pseudotime scatter plots to communicate lineage structure, mirroring SCALA's trajectory analysis module.

## When to use

Use when a processed scRNA-seq Seurat object already carries Louvain cluster assignments and reduced-dimension coordinates (PCA, UMAP, PHATE) and you need to infer and visually present cellular trajectories as a cluster-level MST.

## When NOT to use

- The dataset is not from human or mouse — SCALA's analysis currently supports only human and mouse datasets
- The object lacks prior clustering; Slingshot's MST is built from cluster labels defining initial and final states, so an unclustered expression matrix alone is insufficient
- The online SCALA version is used for datasets larger than 50,000 cells, where some steps cannot be completed (prefer the desktop version)

## Inputs

- Processed Seurat object with Louvain cluster assignments and reduced-dimension coordinates (PCA, UMAP)
- Cell cluster labels used to define the trajectory's initial and final states
- Optional PHATE embedding computed with phateR
- User-defined initial and final cluster states

## Outputs

- Cluster-level Minimal Spanning Tree (MST)
- Per-cell pseudotime values for each defined lineage (tabular)
- UMAP plot with the Slingshot MST overlaid
- Pseudotime scatter plot in UMAP space
- Exported PHATE coordinates

## How to apply

Load the processed scRNA-seq dataset (e.g., PBMC3k Seurat object) with Louvain cluster assignments and PCA/UMAP reductions, and optionally compute a PHATE embedding with the GitHub version of phateR for a diffusion-based visualization space. Run Slingshot (version 2.1.0 or newer, installed after phateR and the Seurat PHATE patch) using the cluster labels and a chosen reduced dimension (PCA, UMAP, tSNE, diffusion map, or PHATE); SCALA's use case used the first 25 PCs determined by SVD k-fold cross-validation. Specify user-defined initial and final states from the clusters to orient the trajectory, which Slingshot uses to construct the cluster-level MST and per-lineage pseudotime. Optionally compute a Monocle pseudotime ordering for comparison with the Slingshot lineages. Export pseudotime values per lineage to a table, export PHATE coordinates, then plot the MST overlaid on the UMAP projection and render pseudotime as a separate scatter plot in UMAP space.

## Related tools

- **Slingshot** (Runs on cluster labels and selected reduced dimensions (e.g., first 25 PCs) with user-specified initial/final states to construct the cluster-level MST and pseudotime)
- **Seurat** (Provides the processed Seurat object with Louvain cluster assignments and PCA/UMAP reductions that feed Slingshot; requires a PHATE-support patch) — https://github.com/scottgigante/seurat
- **phateR** (GitHub version computes a PHATE embedding as a trajectory-friendly reduced dimension option for Slingshot) — https://github.com/KrishnaswamyLab/phateR
- **Monocle** (Computes an alternative pseudotime ordering for comparison with Slingshot lineages)
- **SCALA** (Interactive R/Shiny pipeline whose trajectory module implements this Slingshot + MST-on-UMAP workflow)

## Examples

```
lineages <- slingshot(seur, clusterLabels = "louvain", reducedDim = "PCA", start.clust = "S1", end.clust = "S5")
```

## Evaluation signals

- The MST is visibly drawn on the UMAP projection connecting the specified cluster states
- Pseudotime values are produced for each defined lineage and written to the output table
- The number of lineages matches the user-specified initial and final cluster states
- The selected reduced dimension (e.g., first 25 PCs) matches the dimensionality-reduction step's output
- Monocle's alternative pseudotime ordering is broadly consistent with the Slingshot lineages

## Limitations

- Slingshot must be version 2.1.0 or newer and installed AFTER phateR and the Seurat PHATE patch, due to strict installation ordering
- SCALA requires a patched Seurat for PHATE compatibility, which may cause inconsistencies with other packages in the R installation
- The CRAN version of phateR will not work with Seurat; the GitHub version is required
- Online SCALA sessions cannot complete some steps for datasets larger than 50,000 cells

## Evidence

- [methods] Run Slingshot (version 2.1.0 or newer) using the cluster labels and the selected reduced dimensions: [evidence span withheld — anchor status: not_found]
- [methods] Plot the MST overlaid on a UMAP projection: [evidence span withheld — anchor status: not_found]
