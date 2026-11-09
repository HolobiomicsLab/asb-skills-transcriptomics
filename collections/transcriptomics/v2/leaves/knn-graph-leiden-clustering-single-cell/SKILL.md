---
name: knn-graph-leiden-clustering-single-cell
description: Use when you have extracted per-cell latent factors from a multi-omics
  integration model (e.g. scMoMaT extract_cell_factors) and need discrete cluster
  labels plus a 2D UMAP visualization to interpret populations.
when_to_use_negative:
- Cells do not come from a shared integrated embedding — clustering raw per-batch
  count matrices will not remove batch effects
- The analysis goal is sub-clustering within already-annotated cell types at finer
  resolution than 0.9 provides, without re-tuning resolution
- Input is already a cluster label table or annotation (no embedding to graph-ify)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMoMaT
  role: Provides post_process() to build the KNN graph from cell factors, leiden_cluster()
    for graph clustering, and UMAP/plot_latent for visualization
  repo: https://github.com/PeterZZQ/scMoMaT
- name: UMAP
  role: 2D embedding of the precomputed-distance graph (metric='precomputed') for
    joint visualization of clusters
- name: MultiMAP
  role: Alternative integration method producing embeddings that could be clustered
    the same way; used as a baseline comparator
  repo: https://github.com/Teichlab/MultiMAP
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:3499355aabc6ae1622cd1b1b1dfd465515d4b802cab782bb8c09abada87a3462
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - Leiden
  - MultiMAP
  - NumPy
  - PyTorch
  - UMAP
  - scMoMaT
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# knn-graph-leiden-clustering-single-cell

## Summary

Builds a shared k-nearest-neighbor graph from integrated single-cell embeddings (e.g. scMoMaT cell factors) and clusters it with the Leiden algorithm to obtain cell populations for downstream annotation and marker-based interpretation.

## When to use

Use when you have extracted per-cell latent factors from a multi-omics integration model (e.g. scMoMaT extract_cell_factors) and need discrete cluster labels plus a 2D UMAP visualization to interpret populations.

## When NOT to use

- Cells do not come from a shared integrated embedding — clustering raw per-batch count matrices will not remove batch effects
- The analysis goal is sub-clustering within already-annotated cell types at finer resolution than 0.9 provides, without re-tuning resolution
- Input is already a cluster label table or annotation (no embedding to graph-ify)

## Inputs

- Cell factor embedding zs (list of per-batch matrices from model.extract_cell_factors())
- Neighborhood size (n_neighbors=30, range 30–50)
- Optional neighborhood radius r (between 0 and 1, default None)
- Leiden resolution (0.9 in the scMoMaT demos)

## Outputs

- s_pair_dist (post-processed pairwise distance matrix)
- knn_indices and knn_dists (KNN graph structure)
- labels_leiden (Leiden cluster assignment per cell)
- x_umap (2-dimensional UMAP coordinates for visualization)

## How to apply

After extracting cell factors (zs) from the first-stage scMoMaT model (K=30 latent dimensions, T=4000 iterations), post-process them into a KNN graph with scmomat.post_process using n_neighbors=30 (recommended range 30–50), njobs=8, and neighborhood radius r=None. Cluster the resulting graph with scmomat.leiden_cluster at resolution=0.9 to obtain labels_leiden. For visualization, fit a scmomat.UMAP object (n_components=2, n_neighbors=30, min_dist=0.20, metric='precomputed', supplying knn_indices and knn_dists) on the s_pair_dist matrix. The resulting cluster labels feed the second-stage retraining (scmomat.scmomat_retrain with labels=labels_leiden) that learns cluster-specific biomarker scores, so cluster quality directly determines annotation quality.

## Related tools

- **scMoMaT** (Provides post_process() to build the KNN graph from cell factors, leiden_cluster() for graph clustering, and UMAP/plot_latent for visualization) — https://github.com/PeterZZQ/scMoMaT
- **UMAP** (2D embedding of the precomputed-distance graph (metric='precomputed') for joint visualization of clusters)
- **MultiMAP** (Alternative integration method producing embeddings that could be clustered the same way; used as a baseline comparator) — https://github.com/Teichlab/MultiMAP

## Examples

```
s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, n_neighbors=30, njobs=8, r=None); labels_leiden = scmomat.leiden_cluster(X=None, knn_indices=knn_indices, knn_dists=knn_dists, resolution=0.9)
```

## Evaluation signals

- Leiden clustering returns a non-empty labels_leiden array whose length equals the total number of cells across all batches
- UMAP plot (mode='joint') shows clusters separating into transcriptionally coherent populations rather than batch-mixed blobs
- For MOp_5batches, top markers per cluster reproduce expected cortical patterns (e.g. Sulf1 -> L6, Rorb -> L4/5, Calb1 -> L2/3) as in Fig. 4d-e
- Training losses from the subsequent retraining stage (model2.train(T=2000)) decrease over iterations
- Graph connectivity (GC) computed against ground-truth labels (e.g. PBMC Mimitou et al. annotations) remains high, with strong ARI/NMI

## Limitations

- The post-processing step historically caused memory overflow issues, addressed in scMoMaT v0.2.2 — large datasets may still be memory-bound
- Leiden resolution (0.9 in demos) and neighborhood size (30–50) are hand-tuned hyperparameters; different values change cluster granularity
- Cluster quality gates the second-stage biomarker detection: poorly separated clusters yield uninterpretable marker scores

## Evidence

- [methods] Post-process cell factors into KNN graph: "s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, n_neighbors, njobs = 8, r = r)"
- [methods] Leiden clustering on post-processed graph: "labels_leiden = scmomat.leiden_cluster(X = None, knn_indices = knn_indices, knn_dists = knn_dists, resolution = resolution)"
- [readme] README post-processing and visualization parameters: "umap_op = scmomat.UMAP(n_components = 2, n_neighbors = n_neighbors, min_dist = 0.20, random_state = 0, metric='precomputed', knn_dists=knn_dists, knn_indices=knn_indices)"
- [readme] README memory-overflow fix: "scMoMaT v0.2.2 **Updated the post-processing step to solve the memory overflow issue**"
