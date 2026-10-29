---
name: phate-diffusion-embedding
description: Use when a processed scRNA-seq dataset (e.g., a Seurat object carrying
  Louvain cluster assignments and PCA/UMAP reductions) needs a diffusion-based, trajectory-friendly
  embedding to support Slingshot trajectory inference and pseudotime visualization.
when_to_use_negative:
- Input is scATAC-seq data analyzed through SCALA's ArchR-based workflow, where dimensionality
  reduction is performed with LSI rather than a PHATE expression embedding.
- The dataset exceeds 50,000 cells on the online SCALA web server, where some steps
  cannot be completed — prefer the desktop version.
- The goal is cluster annotation or marker identification (e.g., CIPR, presto) rather
  than trajectory or pseudotime analysis.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: phateR
  role: Computes the PHATE diffusion embedding on the processed Seurat object; the
    GitHub version is required for phate runs with Seurat
  repo: https://github.com/KrishnaswamyLab/phateR
- name: Seurat
  role: Hosts the processed scRNA-seq data, cluster assignments and reductions; must
    be patched for PHATE compatibility
  repo: https://github.com/scottgigante/seurat
- name: Slingshot
  role: Consumes the PHATE (or PCA) reduction plus cluster labels to build the cluster-level
    MST and compute pseudotime per lineage
- name: destiny
  role: Provides diffusion map embeddings, an alternative trajectory-friendly reduced-dimension
    option supported by SCALA
  repo: https://github.com/theislab/destiny
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:a1b3ec4d3520e887304d46e1e71b381e93567d4abdf913edb1f21cc15128b82e
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
  - SCALA
  - Seurat
  - Slingshot
  - destiny
  - phateR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# phate-diffusion-embedding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes a PHATE diffusion embedding of scRNA-seq cells with the GitHub version of phateR on a patched Seurat object, producing a diffusion-based, trajectory-friendly reduced-dimension space that SCALA uses for Slingshot trajectory inference and pseudotime visualization.

## When to use

Use when a processed scRNA-seq dataset (e.g., a Seurat object carrying Louvain cluster assignments and PCA/UMAP reductions) needs a diffusion-based, trajectory-friendly embedding to support Slingshot trajectory inference and pseudotime visualization.

## When NOT to use

- Input is scATAC-seq data analyzed through SCALA's ArchR-based workflow, where dimensionality reduction is performed with LSI rather than a PHATE expression embedding.
- The dataset exceeds 50,000 cells on the online SCALA web server, where some steps cannot be completed — prefer the desktop version.
- The goal is cluster annotation or marker identification (e.g., CIPR, presto) rather than trajectory or pseudotime analysis.

## Inputs

- Processed scRNA-seq Seurat object with Louvain cluster assignments and reduced-dimension coordinates (PCA, UMAP), e.g., the PBMC3k dataset
- Gene-by-cell expression matrix from the quality-controlled, normalized dataset
- Conda environment with Python phate and required system libraries (HDF5, GSL, libgdal, etc.)

## Outputs

- PHATE embedding coordinates per cell, stored in the Seurat object and exportable as a coordinate table
- Trajectory-friendly diffusion space usable as reduced-dimension input for Slingshot MST construction and pseudotime computation
- Plots showing the MST and pseudotime values overlaid on UMAP projections

## How to apply

Load the processed Seurat object (e.g., the PBMC3k scRNA-seq dataset with Louvain cluster assignments and reduced-dimension PCA/UMAP coordinates) and compute a PHATE embedding of the cells with phateR, using the GitHub version of phateR because the CRAN release WILL NOT WORK for phate runs with Seurat. Ensure the Seurat patch for PHATE support (github.com/scottgigante/seurat) is applied after Seurat, phate and phateR are installed, and install Slingshot (version 2.1.0 or newer) only after phateR, following the strict install order in install_libraries.R; the Python phate package must also be present in the conda environment. Use the resulting PHATE embedding as one of the reduced-dimension inputs SCALA supports for trajectory inference (PCA, UMAP, tSNE, diffusion map, or PHATE), together with cluster labels defining initial and final states; in the paper's use case, Slingshot ran on the first 25 PCs determined by SVD k-fold cross-validation, with PHATE providing the alternative trajectory-friendly space. Export the PHATE coordinates and verify the cluster-level Minimal Spanning Tree (MST) and per-lineage pseudotime values are rendered on UMAP projections.

## Related tools

- **phateR** (Computes the PHATE diffusion embedding on the processed Seurat object; the GitHub version is required for phate runs with Seurat) — https://github.com/KrishnaswamyLab/phateR
- **Seurat** (Hosts the processed scRNA-seq data, cluster assignments and reductions; must be patched for PHATE compatibility) — https://github.com/scottgigante/seurat
- **Slingshot** (Consumes the PHATE (or PCA) reduction plus cluster labels to build the cluster-level MST and compute pseudotime per lineage)
- **destiny** (Provides diffusion map embeddings, an alternative trajectory-friendly reduced-dimension option supported by SCALA) — https://github.com/theislab/destiny

## Examples

```
devtools::install_github("KrishnaswamyLab/phateR")
```

## Evaluation signals

- phateR is installed from GitHub (the CRAN release will not perform phate runs with Seurat), and the Seurat PHATE patch was applied after Seurat, phate and phateR.
- PHATE coordinates are computed for all cells in the dataset and can be exported to a coordinate table.
- The downstream Slingshot run succeeds, producing a cluster-level Minimal Spanning Tree (MST) and pseudotime values for each defined lineage.
- The UMAP plot displays the overlaid MST and a separate pseudotime scatter plot is rendered in UMAP space, mirroring SCALA's trajectory outputs.
- The reduced-dimension input to Slingshot matches the documented choice (e.g., the first 25 PCs determined by SVD k-fold cross-validation in the PBMC3k use case, with PHATE among the supported options).

## Limitations

- The CRAN version of phateR will not work with Seurat; the GitHub version of phateR is required.
- SCALA requires modifications (patching) to the Seurat package for phate/phateR compatibility, which may lead to inconsistencies with other packages in the R installation.
- Installation order is critical: Slingshot (2.1.0 or newer) must be installed after phateR and after Seurat has been patched.
- The online SCALA version restricts dataset size to < 2 GB and cannot complete some steps for datasets larger than 50,000 cells; prefer the desktop version.

## Evidence
