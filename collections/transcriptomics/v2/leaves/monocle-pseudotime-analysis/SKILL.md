---
name: monocle-pseudotime-analysis
description: Use when a processed scRNA-seq dataset (clustered cells with reduced-dimension
  coordinates, e.g. PCA/UMAP) needs a second, independent pseudotime ordering to validate
  or compare against Slingshot lineage inference.
when_to_use_negative:
- The dataset has not yet been through QC, normalization, dimensionality reduction,
  and clustering — trajectory inference requires cluster labels and reduced-dimension
  coordinates as input.
- Only a single, already-established ordering is needed and no independent comparison
  of trajectory inference results is required.
- The dataset is not from human or mouse, since SCALA currently supports only human
  and mouse datasets.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCALA
  role: Interactive R/Shiny pipeline hosting the trajectory inference module that
    runs Slingshot and Monocle on clustered scRNA-seq data
  repo: http://scala.pavlopouloslab.info
- name: Slingshot
  role: 'Primary trajectory algorithm: builds the cluster-level MST on the selected
    reduced dimensions (first 25 PCs in the use case) with user-defined initial/final
    states'
- name: Monocle
  role: Computes the alternative pseudo-time ordering of cells for comparison with
    Slingshot lineages
- name: phateR
  role: Computes a PHATE diffusion-based embedding used as a trajectory-friendly reduced-dimension
    space for Slingshot
  repo: https://github.com/KrishnaswamyLab/phateR
- name: Seurat
  role: Provides the processed Seurat object with cluster assignments and PCA/UMAP
    coordinates used as trajectory input
  repo: https://github.com/scottgigante/seurat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:223ecb231f3455c9981769ffd77f86a6fb347a135c044b80a838348d98e5e251
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  tools:
  - Monocle
  - SCALA
  - Seurat
  - Slingshot
  - phateR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# monocle-pseudotime-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes an alternative cellular pseudo-time ordering of single cells with Monocle, used within SCALA's trajectory inference module to cross-compare its lineages against Slingshot-derived trajectories.

## When to use

Use when a processed scRNA-seq dataset (clustered cells with reduced-dimension coordinates, e.g. PCA/UMAP) needs a second, independent pseudotime ordering to validate or compare against Slingshot lineage inference.

## When NOT to use

- The dataset has not yet been through QC, normalization, dimensionality reduction, and clustering — trajectory inference requires cluster labels and reduced-dimension coordinates as input.
- Only a single, already-established ordering is needed and no independent comparison of trajectory inference results is required.
- The dataset is not from human or mouse, since SCALA currently supports only human and mouse datasets.

## Inputs

- Processed scRNA-seq dataset (e.g. Seurat object such as PBMC3k) with Louvain cluster assignments
- Reduced-dimension coordinates (PCA, UMAP, tSNE, diffusion map, or PHATE embeddings)
- User-defined initial and final cell states/clusters for trajectory orientation

## Outputs

- Per-cell pseudotime values per lineage written to a table
- Cluster-level Minimal Spanning Tree (MST) overlaid on a UMAP projection
- Pseudotime scatter plot rendered in UMAP space
- Exported PHATE coordinates

## How to apply

After clustering and dimensionality reduction (e.g. Louvain clusters on a PBMC3k Seurat object), run Monocle to compute an alternative pseudo-time ordering of the cells alongside the Slingshot lineages. Slingshot itself is run (version 2.1.0 or newer) on the user-selected reduced dimensions — SCALA supports PCA, UMAP, tSNE, diffusion map, or PHATE — using the first 25 most informative PCs in the paper's use case, with cell clusters defining the initial and final states. The Monocle pseudotime serves as a comparison point against the Slingshot cluster-level Minimal Spanning Tree (MST). Extract the per-lineage pseudotime values into a table and visualize them as a scatter plot in UMAP space, with the MST overlaid on the UMAP projection. Correct application is judged by agreement/consistency between Monocle pseudotime and Slingshot lineages and by the presence of pseudotime values for every defined lineage.

## Related tools

- **SCALA** (Interactive R/Shiny pipeline hosting the trajectory inference module that runs Slingshot and Monocle on clustered scRNA-seq data) — http://scala.pavlopouloslab.info
- **Slingshot** (Primary trajectory algorithm: builds the cluster-level MST on the selected reduced dimensions (first 25 PCs in the use case) with user-defined initial/final states)
- **Monocle** (Computes the alternative pseudo-time ordering of cells for comparison with Slingshot lineages)
- **phateR** (Computes a PHATE diffusion-based embedding used as a trajectory-friendly reduced-dimension space for Slingshot) — https://github.com/KrishnaswamyLab/phateR
- **Seurat** (Provides the processed Seurat object with cluster assignments and PCA/UMAP coordinates used as trajectory input) — https://github.com/scottgigante/seurat

## Examples

```
Rscript install_libraries.R  # installs Slingshot (>=2.1.0), Monocle, phateR (GitHub version) and patched Seurat for SCALA trajectory analysis
```

## Evaluation signals

- The cluster-level Minimal Spanning Tree (MST) is drawn in the UMAP plot.
- Pseudotime values are produced for each defined lineage.
- Monocle pseudotime ordering is compared against Slingshot lineages for consistency, mirroring SCALA's trajectory outputs (Fig. 1 trajectory analysis module).
- Extracted pseudotime table contains values for all cells in the pooled dataset.

## Limitations

- Slingshot must be version 2.1.0 or newer and installed AFTER phateR and after Seurat has been patched for PHATE support.
- SCALA requires the GitHub version of phateR — the CRAN version WILL NOT WORK for phate runs with Seurat.
- SCALA's Seurat patch for PHATE compatibility may cause inconsistencies with other packages in the R installation.
- Online SCALA version restricts dataset size to < 2 GB and cannot complete some steps for datasets larger than 50,000 cells; the desktop version is suggested for larger datasets.

## Evidence

- [full_text] SCALA's scRNA-seq trajectory step computes cellular trajectories for the pooled dataset by running the Slingshot algorithm on the most informative principal components: [evidence span withheld — anchor status: not_found]
- [full_text] Monocle comparison step: [evidence span withheld — anchor status: not_found]
- [full_text] Reduced dimension options: [evidence span withheld — anchor status: not_found]
