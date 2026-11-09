---
name: umap-2d-embedding-visualization
description: Use when a scRNA-seq dataset has been reduced to a high-dimensional embedding
  (e.g. 40 PCA dimensions) and a 2D visualization is needed to inspect or display
  Leiden cluster structure, as done for the HuBMAP HBMP3-spleen-CC2 dataset.
when_to_use_negative:
- When the goal is the clustering itself — Cellar constructs the neighbors graph for
  Leiden clustering on the PCA embedding, not on the 2D UMAP projection
- When analyzing non-expression modalities that Cellar handles with other reductions,
  e.g. lymph node CODEX data which was 'reduced via UMAP with 10 dimensions'
- When the input data has not been filtered, CPM-normalized (total count 1e5), log1p-transformed,
  and scaled — the embedding is defined downstream of those preprocessing steps
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cellar
  role: Interactive pipeline that orchestrates PCA, UMAP dimensionality reduction,
    and Leiden clustering on HuBMAP scRNA-seq datasets
  repo: https://github.com/euxhenh/cellar
- name: scikit-learn
  role: Provides the PCA implementation with randomized SVD solver that produces the
    embedding reduced before 2D UMAP visualization
- name: faiss
  role: Builds the approximate nearest-neighbors graph (15 neighbors) used for Leiden
    clustering alongside the UMAP visualization
  repo: https://github.com/facebookresearch/faiss
- name: AnnData
  role: In-memory annotated data object (used in memory-mapping mode) that stores
    the embedding, UMAP coordinates, and cluster labels
- name: scanpy
  role: Used for preprocessing (filtering, normalization, log1p, scaling) prior to
    dimensionality reduction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:2ee583aab9793189bc39ba85632ff32634e8943c7c86cff78eb233c637cb5b2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/euxhenh/cellar
  tools:
  - AnnData
  - CUDA
  - Cellar
  - Julia
  - UMAP.jl
  - faiss
  - scLENS
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# umap-2d-embedding-visualization

## Summary

Reduce high-dimensional single-cell embeddings (e.g. 40-dimension PCA of scRNA-seq data) to 2D with UMAP within the Cellar pipeline, producing a low-dimensional visualization that accompanies Leiden clustering of the cells.

## When to use

Use when a scRNA-seq dataset has been reduced to a high-dimensional embedding (e.g. 40 PCA dimensions) and a 2D visualization is needed to inspect or display Leiden cluster structure, as done for the HuBMAP HBMP3-spleen-CC2 dataset.

## When NOT to use

- When the goal is the clustering itself — Cellar constructs the neighbors graph for Leiden clustering on the PCA embedding, not on the 2D UMAP projection
- When analyzing non-expression modalities that Cellar handles with other reductions, e.g. lymph node CODEX data which was 'reduced via UMAP with 10 dimensions'
- When the input data has not been filtered, CPM-normalized (total count 1e5), log1p-transformed, and scaled — the embedding is defined downstream of those preprocessing steps

## Inputs

- Preprocessed scRNA-seq count matrix in AnnData (.h5ad) format (e.g. 5273-cell HBMP3-spleen-CC2 HuBMAP spleen dataset)
- 40-dimension PCA embedding produced with scikit-learn randomized SVD
- Leiden cluster assignments stored in adata.obs['labels']

## Outputs

- 2D UMAP embedding of the cells for visualization
- UMAP/cluster overlay compatible with an AnnData session file and per-cluster cell-count table

## How to apply

After preprocessing (cell/gene filtering at 50–3000 thresholds, CPM normalization with total count 1e5, log1p transform, zero-mean/unit-variance scaling), reduce the data to 40 dimensions using scikit-learn PCA with a randomized SVD solver, as done in Cellar. From that embedding, run UMAP to project cells to 2D for visualization while separately constructing an approximate nearest-neighbors graph with faiss using 15 neighbors for Leiden clustering at resolution 1. The UMAP coordinates are a visualization layer, not the clustering input — clustering operates on the PCA embedding. Persist results in an AnnData session object so the 2D embedding and cluster labels can be jointly inspected against reference figures such as Supplementary Fig. 1a.

## Related tools

- **Cellar** (Interactive pipeline that orchestrates PCA, UMAP dimensionality reduction, and Leiden clustering on HuBMAP scRNA-seq datasets) — https://github.com/euxhenh/cellar
- **scikit-learn** (Provides the PCA implementation with randomized SVD solver that produces the embedding reduced before 2D UMAP visualization)
- **faiss** (Builds the approximate nearest-neighbors graph (15 neighbors) used for Leiden clustering alongside the UMAP visualization) — https://github.com/facebookresearch/faiss
- **AnnData** (In-memory annotated data object (used in memory-mapping mode) that stores the embedding, UMAP coordinates, and cluster labels)
- **scanpy** (Used for preprocessing (filtering, normalization, log1p, scaling) prior to dimensionality reduction)

## Examples

```
import scanpy as sc; sc.tl.umap(adata)  # after sc.pp.pca(adata, n_comps=40, svd_solver='randomized') and faiss-based neighbors for Leiden
```

## Evaluation signals

- Output AnnData session contains assignments for all 5273 cells of the HBMP3-spleen-CC2 dataset
- The accompanying Leiden clustering (resolution 1, 15 faiss neighbors) reproduces 16 clusters as reported in Supplementary Fig. 1a
- 2D UMAP coordinates are available for every cell and consistent with the cluster labels in adata.obs['labels']
- PCA embedding used as UMAP input has 40 dimensions, matching the scikit-learn randomized-SVD setting
- Visual cluster separation on the UMAP is consistent with the per-cluster cell-count table exported by Cellar

## Limitations

- No software versions are stated for Cellar, Python, or the underlying libraries, so exact reproducibility of the embedding may vary
- Reproducibility depends on the external Cellar web server (https://cellar.cmu.hubmapconsortium.org/app/cellar) rather than a locally installable archived artifact
- No accession IDs are given for the HuBMAP annotated datasets, complicating retrieval of the exact input matrix
- UMAP here serves visualization only; cluster structure is defined by Leiden on the PCA embedding, so 2D plots may visually distort cluster relationships

## Evidence

- [results] PCA + UMAP dimensionality reduction and Leiden clustering on the HBMP3-spleen-CC2 dataset: [evidence span withheld — anchor status: not_found]
- [methods] UMAP used for visualization alongside faiss-based neighbors graph: [evidence span withheld — anchor status: not_found]
- [methods] PCA implementation details: "We used the PCA implementation of the scikit-learn package with a randomized SVD solver."
- [discussion] faiss accelerates the neighbors graph used for Leiden clustering: "approximate nearest neighbors based on faiss25 to speed up neighbors graph construction for Leiden clustering"
- [discussion] AnnData memory-mapped storage for large datasets: "using the Annotated Data object17 in memory-mapping mode which allows the analysis of large datasets"
