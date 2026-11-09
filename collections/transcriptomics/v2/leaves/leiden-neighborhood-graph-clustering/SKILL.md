---
name: leiden-neighborhood-graph-clustering
description: Use when clustering cells in a learned latent feature matrix (e.g. Miscell
  encoder output loaded into an AnnData object) and a graph-based Leiden partition
  is preferred over DBSCAN clustering of the t-SNE embedding.
when_to_use_negative:
- The analysis goal is DBSCAN clustering directly on the 2D t-SNE embedding (eps=1,
  min_samples=5) rather than graph-based Leiden clustering on the full feature space.
- 'The dataset is small: the Miscell paper notes that deep-learning-based approaches
  demand a large sample size to obtain good performance.'
- The input is raw or batch-affected expression data that has not yet been batch-corrected
  and encoded into latent features, since Miscell leaves batch correction as a preprocessing
  step outside the model.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Builds the neighborhood graph (sc.pp.neighbors with n_neighbors=10, use_rep='X'),
    runs Leiden clustering (sc.tl.leiden), and renders the t-SNE plot colored by Leiden
    clusters (sc.pl.tsne with save='_scanpy.pdf').
  repo: https://github.com/lixiangchun/Miscell
- name: anndata
  role: Wraps the Miscell latent feature matrix into an AnnData object and stores
    the FIt-SNE coordinates in adata.obsm['X_tsne'].
  repo: https://github.com/lixiangchun/Miscell
- name: FIt-SNE
  role: Produces the 2D t-SNE embedding (seed=123, nthreads=12, perplexity_list=[30,
    36, 42, 48]) on which the Leiden clusters are visualized.
  repo: https://github.com/KlugerLab/FIt-SNE
- name: Miscell
  role: Self-supervised deep neural network encoder that generates the latent feature
    representation used as the neighborhood-graph input ('X').
  repo: https://github.com/lixiangchun/Miscell
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:27861cb5570f8e68a4312c4ea0a30c16523bab2596d4ca463a45dfd45a06ed0c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  repo_url: https://github.com/lixiangchun/Miscell
  tools:
  - FIt-SNE
  - Miscell
  - Scanpy
  - Seurat
  - anndata
  - scVI
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# leiden-neighborhood-graph-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cluster single-cell latent features by building a k-nearest-neighbor graph with Scanpy (n_neighbors=10, use_rep='X') and running Leiden community detection, then visualizing the resulting clusters on a FIt-SNE t-SNE embedding. This is the graph-based clustering pipeline the Miscell paper and README use as an alternative to DBSCAN-on-t-SNE for delineating single-cell clusters.

## When to use

Use when clustering cells in a learned latent feature matrix (e.g. Miscell encoder output loaded into an AnnData object) and a graph-based Leiden partition is preferred over DBSCAN clustering of the t-SNE embedding.

## When NOT to use

- The analysis goal is DBSCAN clustering directly on the 2D t-SNE embedding (eps=1, min_samples=5) rather than graph-based Leiden clustering on the full feature space.
- The dataset is small: the Miscell paper notes that deep-learning-based approaches demand a large sample size to obtain good performance.
- The input is raw or batch-affected expression data that has not yet been batch-corrected and encoded into latent features, since Miscell leaves batch correction as a preprocessing step outside the model.

## Inputs

- Latent feature matrix from the Miscell self-supervised encoder (features for all cells, e.g. PBMC 43,095 cells, Smart-seq 71,494 cells, or Tabula Muris 55,656 cells)
- anndata.AnnData object wrapping the feature matrix
- 2D FIt-SNE embedding (fast_tsne output with seed=123, nthreads=12, perplexity_list=[30, 36, 42, 48])
- Ground-truth cell-type labels for external validation (e.g. FACS-sorted CD4+/CD8+ T cell labels or Tabula Muris Senis cell-type annotations)

## Outputs

- Leiden cluster assignments per cell (adata.obs['leiden'])
- t-SNE plot colored by Leiden clusters saved as PDF (save='_scanpy.pdf', e.g. tsne_scanpy.pdf)
- Clustering quality metrics (ARI, NMI, V-measure) written to clustering_metrics.csv or smartseq_clustering_metrics.csv

## How to apply

Load the feature matrix (e.g. features extracted from a pretrained Miscell checkpoint) into an AnnData object and store the 2D FIt-SNE coordinates (computed with fast_tsne, seed=123, nthreads=12, perplexity_list=[30, 36, 42, 48]) in adata.obsm['X_tsne']. Build the neighborhood graph with sc.pp.neighbors(adata, n_neighbors=10, use_rep='X'), i.e. using the full feature matrix rather than the t-SNE coordinates, then run Leiden community detection with sc.tl.leiden(adata). Visualize the partition with sc.pl.tsne(adata, color=['leiden'], save='_scanpy.pdf', show=False) to produce the t-SNE plot colored by Leiden clusters (e.g. tsne_scanpy.pdf). Finally, score the assignments against reference labels (e.g. FACS-sorted CD4+/CD8+ T cell labels) with adjusted Rand index, normalized mutual information, and V-measure, saving per-method results to clustering_metrics.csv.

## Related tools

- **Scanpy** (Builds the neighborhood graph (sc.pp.neighbors with n_neighbors=10, use_rep='X'), runs Leiden clustering (sc.tl.leiden), and renders the t-SNE plot colored by Leiden clusters (sc.pl.tsne with save='_scanpy.pdf').) — https://github.com/lixiangchun/Miscell
- **anndata** (Wraps the Miscell latent feature matrix into an AnnData object and stores the FIt-SNE coordinates in adata.obsm['X_tsne'].) — https://github.com/lixiangchun/Miscell
- **FIt-SNE** (Produces the 2D t-SNE embedding (seed=123, nthreads=12, perplexity_list=[30, 36, 42, 48]) on which the Leiden clusters are visualized.) — https://github.com/KlugerLab/FIt-SNE
- **Miscell** (Self-supervised deep neural network encoder that generates the latent feature representation used as the neighborhood-graph input ('X').) — https://github.com/lixiangchun/Miscell

## Examples

```
import scanpy as sc, anndata; adata=anndata.AnnData(features); adata.obsm['X_tsne']=tsne; sc.pp.neighbors(adata, n_neighbors=10, use_rep='X'); sc.tl.leiden(adata); sc.pl.tsne(adata, color=['leiden'], save='_scanpy.pdf', show=False)
```

## Evaluation signals

- sc.pp.neighbors is called with n_neighbors=10 and use_rep='X' (the latent feature matrix, not the t-SNE coordinates).
- The Leiden clustering runs on the neighborhood graph built from the encoder features, and clusters appear as spatially coherent groups when plotted on the FIt-SNE embedding (adata.obsm['X_tsne']).
- The t-SNE plot colored by Leiden clusters is saved as a PDF (e.g. tsne_scanpy.pdf) via sc.pl.tsne(..., save='_scanpy.pdf').
- Against reference labels (e.g. FACS-sorted CD4+/CD8+ T cell labels for the Smart-seq dataset), Miscell + Leiden achieves ARI/NMI/V-measure gains over Scanpy, Seurat, and scVI baselines (e.g. +120% ARI on Smart-seq, +33% ARI on PBMC, +10% ARI on Tabula Muris over the second-best method).
- Per-method ARI, NMI, and V-measure values are written to a comparison table (clustering_metrics.csv / smartseq_clustering_metrics.csv) covering all four methods.

## Limitations

- Miscell requires a large sample size for good performance, limiting use on small datasets.
- No batch-correction method is internally included; batch effects must be corrected as a preprocessing step before running Miscell and clustering.
- Cluster-specific marker gene identification by Miscell is limited to highly variable genes, so marker genes outside HVGs may be missed (relevant when the Leiden clusters are used downstream for marker discovery).

## Evidence
