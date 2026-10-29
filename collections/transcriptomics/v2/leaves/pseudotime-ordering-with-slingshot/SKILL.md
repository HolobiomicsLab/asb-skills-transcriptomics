---
name: pseudotime-ordering-with-slingshot
description: Use when a processed scRNA-seq dataset (Seurat object with Louvain cluster
  assignments and PCA/UMAP/PHATE reductions) needs trajectory orientation and per-cell
  pseudotime values along defined lineages.
when_to_use_negative:
- Input is a raw count matrix without clustering or dimensionality reduction — trajectory
  states need precomputed cluster labels and PCA/PHATE coordinates
- Analysis of organisms other than human or mouse — SCALA currently supports only
  human and mouse datasets
- Datasets larger than 50,000 cells in the online SCALA version, where some steps
  cannot be completed (use the desktop version)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Slingshot
  role: Constructs the cluster-level MST and computes pseudotime along defined lineages
    from cluster labels and reduced dimensions
- name: Monocle
  role: Computes an alternative pseudotime ordering for comparison with Slingshot
    lineages
- name: phateR
  role: Provides the GitHub-version PHATE diffusion embedding used as a trajectory-friendly
    reduced-dimension input
  repo: https://github.com/KrishnaswamyLab/phateR
- name: Seurat
  role: Hosts the processed dataset, cluster assignments, and reductions; requires
    a patch for PHATE compatibility
  repo: https://github.com/scottgigante/seurat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:6a0eb9a495820ce79abc564e697038c441a560fc577b2e0b62cb173b4511f86d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/KrishnaswamyLab/phateR
  tools:
  - Monocle
  - SCALA
  - Seurat
  - Slingshot
  - phateR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# pseudotime-ordering-with-slingshot

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cellular trajectory and pseudotime inference as performed in SCALA's scRNA-seq module: Slingshot builds a cluster-level Minimal Spanning Tree over reduced-dimension coordinates (e.g., the first 25 PCs) using cell clusters to define initial and final states, with Monocle used as an alternative pseudotime ordering for comparison.

## When to use

Use when a processed scRNA-seq dataset (Seurat object with Louvain cluster assignments and PCA/UMAP/PHATE reductions) needs trajectory orientation and per-cell pseudotime values along defined lineages.

## When NOT to use

- Input is a raw count matrix without clustering or dimensionality reduction — trajectory states need precomputed cluster labels and PCA/PHATE coordinates
- Analysis of organisms other than human or mouse — SCALA currently supports only human and mouse datasets
- Datasets larger than 50,000 cells in the online SCALA version, where some steps cannot be completed (use the desktop version)

## Inputs

- Processed Seurat scRNA-seq object with Louvain cluster assignments and reduced-dimension coordinates (PCA, UMAP)
- PHATE embedding computed with phateR (GitHub version)
- User-defined initial and final cell-state cluster labels

## Outputs

- Cluster-level Minimal Spanning Tree (MST) overlaid on a UMAP projection
- Per-lineage pseudotime table for each cell
- Scatter plot of pseudotime values rendered in UMAP space
- Exported PHATE coordinates
- Alternative Monocle pseudotime ordering for comparison

## How to apply

Load the processed dataset (e.g., PBMC3k Seurat object) carrying Louvain cluster assignments and reduced-dimension coordinates. Compute a PHATE embedding with the GitHub version of phateR to obtain a diffusion-based trajectory-friendly space; note Seurat must be patched for PHATE support. Run Slingshot (version 2.1.0 or newer) using cluster labels and the selected reduced dimensions (PCA, UMAP, tSNE, diffusion map, or PHATE) — in the SCALA use case the first 25 PCs, as determined by SVD k-fold cross-validation — specifying user-defined initial and final states so the cluster-level MST is oriented. Compute an alternative pseudotime ordering with Monocle to compare against the Slingshot lineages. Extract per-lineage pseudotime values to a table, export PHATE coordinates, and plot the MST overlaid on UMAP plus pseudotime as a UMAP scatter plot.

## Related tools

- **Slingshot** (Constructs the cluster-level MST and computes pseudotime along defined lineages from cluster labels and reduced dimensions)
- **Monocle** (Computes an alternative pseudotime ordering for comparison with Slingshot lineages)
- **phateR** (Provides the GitHub-version PHATE diffusion embedding used as a trajectory-friendly reduced-dimension input) — https://github.com/KrishnaswamyLab/phateR
- **Seurat** (Hosts the processed dataset, cluster assignments, and reductions; requires a patch for PHATE compatibility) — https://github.com/scottgigante/seurat

## Examples

```
BiocManager::install("slingshot")
```

## Evaluation signals

- The MST is visibly drawn in the UMAP projection plot
- Pseudotime values are produced for every defined lineage (one value per cell per lineage)
- The number of informative PCs supplied to Slingshot matches the SVD k-fold cross-validation result (25 PCs in the SCALA use case)
- The Monocle ordering yields a comparable pseudotime arrangement to the Slingshot lineages on the same cells
- PHATE coordinates were exported from the GitHub version of phateR, not the CRAN version

## Limitations

- Slingshot must be version 2.1.0 or newer and installed after phateR and after Seurat has been patched; the CRAN phateR version will not work
- The Seurat patch required for PHATE compatibility may cause inconsistencies with other packages in the R installation
- Only human and mouse datasets are currently supported by SCALA for this analysis

## Evidence
