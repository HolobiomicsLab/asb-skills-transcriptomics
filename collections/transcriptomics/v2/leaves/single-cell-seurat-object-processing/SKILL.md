---
name: single-cell-seurat-object-processing
description: Use when starting from raw or unprocessed scRNA-seq data — Seurat objects,
  .rds count matrices, or paired count/metadata files — that must be loaded, normalized,
  and clustered before cell-type annotation with the scMiko/Miko scoring pipeline.
when_to_use_negative:
- Input is already a processed, scored Seurat object that only needs annotation steps
  (e.g., sigScore/post-hoc coherence filtering) — skip to the scoring modules instead
- Data lives in a non-R/Scanpy-native format (e.g., .h5ad) that cannot be directly
  read with readRDS() without conversion to Seurat structure
- The analysis goal is gene program discovery on an already-annotated atlas where
  re-clustering would destroy the author-curated labels used for benchmarking
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: scMiko
  role: R package providing the Seurat-based single-cell analytic workflows (loading,
    preprocessing, clustering) and generating scPipeline reports
  repo: https://github.com/NMikolajewicz/scMiko
- name: scPipeline
  role: Modular Rmarkdown scripts that execute the preprocessing/clustering modules
    and emit reproducible dashboard reports
  repo: https://github.com/NMikolajewicz/scPipeline
- name: Seurat
  role: Underlying framework in which scMiko is designed; supplies object structure,
    normalization, scaling, PCA, clustering (FindNeighbors/FindClusters), and UMAP
  repo: https://satijalab.org/seurat/
- name: sctransform
  role: Alternative normalization/variance-stabilization method (regularized negative
    binomial regression) whose core functionality is integrated into Seurat for the
    preprocessing step
  repo: https://github.com/satijalab/sctransform
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:5e66e19b56858b40cbc80ef76b7925b71fab2884853bcbafe4059c3f7d0c56c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  - scPipeline
  - sctransform
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# single-cell-seurat-object-processing

## Summary

Load and preprocess single-cell RNA-seq data as Seurat objects (e.g., .rds files like so_tyser2021_220621.rds or paired count/matrix files like ps_count_matrix.rds + ps_meta_data.rds) using scMiko and the modular scPipeline Rmarkdown workflows, so downstream steps (clustering, Miko scoring, SSN analysis) operate on a consistent Seurat scaffold.

## When to use

Use when starting from raw or unprocessed scRNA-seq data — Seurat objects, .rds count matrices, or paired count/metadata files — that must be loaded, normalized, and clustered before cell-type annotation with the scMiko/Miko scoring pipeline.

## When NOT to use

- Input is already a processed, scored Seurat object that only needs annotation steps (e.g., sigScore/post-hoc coherence filtering) — skip to the scoring modules instead
- Data lives in a non-R/Scanpy-native format (e.g., .h5ad) that cannot be directly read with readRDS() without conversion to Seurat structure
- The analysis goal is gene program discovery on an already-annotated atlas where re-clustering would destroy the author-curated labels used for benchmarking

## Inputs

- Seurat object stored as .rds (e.g., so_tyser2021_220621.rds)
- Raw count matrix as .rds (e.g., ps_count_matrix.rds)
- Cell-level metadata table as .rds (e.g., ps_meta_data.rds)
- Cell Ranger (10X Genomics) output for raw scRNA-seq data

## Outputs

- Normalized, scaled, and clustered Seurat object with cluster identities in metadata
- scPipeline flexdashboard report logging preprocessing and clustering parameters
- Clustered object ready for Miko scoring (nullScore/mikoScore/sigScore) and annotation

## How to apply

Install the scMiko package with devtools::install_github(repo = "NMikolajewicz/scMiko") and run the corresponding scPipeline Rmarkdown module, which generates a self-contained flexdashboard report logging analytic parameters. Load inputs as Seurat objects: either a prebuilt .rds object (so.query <- readRDS("../data/demo/so_tyser2021_220621.rds")) or a count matrix plus metadata pair (ps.data <- readRDS("../data/demo/ps_count_matrix.rds"); ps.meta <- readRDS("../data/demo/ps_meta_data.rds")) assembled into a Seurat object. Normalize (LogNormalize or sctransform via satijalab/sctransform, whose core functionality is integrated into Seurat), scale, run PCA, cluster, and visualize with UMAP. Choose the clustering configuration using the marker-specificity resolution criterion before proceeding to annotation. The rationale is that reliable annotation 'begins with identifying the optimal clustering configuration', and the Seurat framework ensures compatibility with downstream scMiko functions such as mikoScore() and coherentFraction().

## Related tools

- **scMiko** (R package providing the Seurat-based single-cell analytic workflows (loading, preprocessing, clustering) and generating scPipeline reports) — https://github.com/NMikolajewicz/scMiko
- **scPipeline** (Modular Rmarkdown scripts that execute the preprocessing/clustering modules and emit reproducible dashboard reports) — https://github.com/NMikolajewicz/scPipeline
- **Seurat** (Underlying framework in which scMiko is designed; supplies object structure, normalization, scaling, PCA, clustering (FindNeighbors/FindClusters), and UMAP) — https://satijalab.org/seurat/
- **sctransform** (Alternative normalization/variance-stabilization method (regularized negative binomial regression) whose core functionality is integrated into Seurat for the preprocessing step) — https://github.com/satijalab/sctransform

## Examples

```
devtools::install_github(repo = "NMikolajewicz/scMiko"); so.query <- readRDS("../data/demo/so_tyser2021_220621.rds")
```

## Evaluation signals

- readRDS() call returns an object of Seurat class with populated RNA assay and count slots
- Post-processing metadata contains a cluster identity column (e.g., seurat_clusters) used as input to per-cluster annotation
- Generated scPipeline flexdashboard report records the analytic parameters and scripts used, matching the executed module
- Downstream scMiko functions (e.g., coherentFraction(object = so.query_scored, ...)) accept the object without slot errors
- Number of clusters reflects the specificity-based optimal resolution rather than an arbitrary default

## Limitations

- scMiko 'is developed and tested in Windows, however it is expected to work in MacOS X and Linux' — platform quirks may arise outside Windows
- Specificity-based resolution selection is limited by the stability and reproducibility of clusters, especially in single-replicate datasets where technical artifacts can appear as distinct clusters
- Preprocessing choices constrain downstream annotation: 'marker-based annotations are only as good as the cell-type prescribed to the original dataset'

## Evidence

- [readme] scMiko is an R package developed for single-cell RNA-seq analysis. It was designed using the Seurat framework: "scMiko is an **R package** developed for single-cell RNA-seq analysis. It was designed using the [Seurat](https://satijalab.org/seurat/) framework"
- [discussion] offer modular workflows that enable data preprocessing, normalization, integration, clustering, annotation, gene program discovery and gene association analyses.: "offer modular workflows that enable data preprocessing, normalization, integration, clustering, annotation, gene program discovery and gene association analyses."
- [discussion] Reliable annotation begins with identifying the optimal clustering configuration.: "Reliable annotation begins with identifying the optimal clustering configuration."
- [methods] so.query <- readRDS("../data/demo/so_tyser2021_220621.rds"): "so.query <- readRDS("../data/demo/so_tyser2021_220621.rds")"
- [readme] Core functionality of this package has been integrated into Seurat: [evidence span withheld — anchor status: external_doc]
