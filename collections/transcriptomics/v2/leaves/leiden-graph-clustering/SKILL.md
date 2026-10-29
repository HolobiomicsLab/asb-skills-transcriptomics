---
name: leiden-graph-clustering
description: Use when a learned joint cell embedding (e.g. scMoMaT cell factors zs)
  needs to be converted into a neighborhood graph, Leiden clusters, and a 2D UMAP
  visualization in a mosaic single-cell multi-omics pipeline.
when_to_use_negative:
- When clustering raw count matrices or per-modality features instead of the learned
  cell factors zs from the trained scMoMaT model.
- When the task is biomarker extraction rather than cell graph construction/clustering
  — feature factors come from the second-stage model2.extract_feature_factors(), not
  from post_process/leiden_cluster.
- When a method other than scMoMaT produced the embedding and you cannot supply the
  per-batch cell factor list it expects.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMoMaT
  role: Provides post_process() to build the cross-batch-normalized neighborhood graph
    from cell factors and leiden_cluster() to obtain cluster labels
  repo: https://github.com/PeterZZQ/scMoMaT
- name: UMAP
  role: Embeds the post-processed graph in 2D using precomputed knn_indices/knn_dists
    fitted on s_pair_dist
- name: Leiden algorithm
  role: Graph clustering of the post-processed KNN graph to produce cell cluster labels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:5ede214368653739865aba9b3fc894af35b85e1ad25b56d1cad99086ae29c9e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - Leiden algorithm
  - Leiden clustering
  - UMAP
  - scMoMaT
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# leiden-graph-clustering

## Summary

Cluster the post-processed neighborhood graph built from scMoMaT cell factors using Leiden clustering, then embed the graph in 2D with UMAP for visualization of joint cell clusters.

## When to use

Use when a learned joint cell embedding (e.g. scMoMaT cell factors zs) needs to be converted into a neighborhood graph, Leiden clusters, and a 2D UMAP visualization in a mosaic single-cell multi-omics pipeline.

## When NOT to use

- When clustering raw count matrices or per-modality features instead of the learned cell factors zs from the trained scMoMaT model.
- When the task is biomarker extraction rather than cell graph construction/clustering — feature factors come from the second-stage model2.extract_feature_factors(), not from post_process/leiden_cluster.
- When a method other than scMoMaT produced the embedding and you cannot supply the per-batch cell factor list it expects.

## Inputs

- Cell factors zs from model.extract_cell_factors() (list of per-batch arrays)
- Batch assignment of each cell
- n_neighbors (neighborhood size, 30-50, e.g. 30)
- Optional radius r in (0,1) for neighbor pruning (default None)
- Leiden resolution (e.g. 0.9)

## Outputs

- s_pair_dist (pairwise distance matrix over all cells)
- knn_indices and knn_dists (post-processed neighborhood graph)
- labels_leiden (exactly one Leiden label per cell)
- x_umap (2D UMAP embedding, shape (ncells, 2))
- Joint latent UMAP plot of Leiden clusters

## How to apply

After the first training stage, extract cell factors zs with model.extract_cell_factors() and build the post-processed KNN graph via scmomat.post_process(zs, n_neighbors, njobs=8, r=r), which allocates per-batch neighbor counts proportional to batch size, normalizes cross-batch distances by the within-/cross-batch mean distance ratio, and optionally prunes neighbors by a radius r in (0,1) (r=None default; use 30-50 for neighborhood size, e.g. n_neighbors=30). Run Leiden clustering with scmomat.leiden_cluster(X=None, knn_indices=knn_indices, knn_dists=knn_dists, resolution=0.9) to obtain labels_leiden, which are then used for second-stage retraining via scmomat.scmomat_retrain. Embed cells in 2D with scmomat.UMAP(n_components=2, n_neighbors=n_neighbors, min_dist=0.20, random_state=0, metric='precomputed', knn_dists=knn_dists, knn_indices=knn_indices) fitted on s_pair_dist, and render with scmomat.plot_latent(x_umap, annos=labels_leiden, mode='joint'). Note scMoMaT v0.2.2 updated the post-processing step to solve a memory overflow issue.

## Related tools

- **scMoMaT** (Provides post_process() to build the cross-batch-normalized neighborhood graph from cell factors and leiden_cluster() to obtain cluster labels) — https://github.com/PeterZZQ/scMoMaT
- **UMAP** (Embeds the post-processed graph in 2D using precomputed knn_indices/knn_dists fitted on s_pair_dist)
- **Leiden algorithm** (Graph clustering of the post-processed KNN graph to produce cell cluster labels)

## Examples

```
s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, 30, njobs=8, r=None); labels_leiden = scmomat.leiden_cluster(X=None, knn_indices=knn_indices, knn_dists=knn_dists, resolution=0.9)
```

## Evaluation signals

- scmomat.post_process returns s_pair_dist, knn_indices, and knn_dists defined for every cell
- Every cell receives exactly one Leiden label from labels_leiden
- The UMAP embedding x_umap has shape (ncells, 2)
- n_neighbors is chosen within the recommended 30-50 range (e.g. 30)
- Leiden clusters are rendered coherently in the joint UMAP plot via scmomat.plot_latent with mode='joint'

## Limitations

- The optional radius pruning r must lie in (0,1) and is None by default; incorrect values invalidate the graph
- scMoMaT versions before v0.2.2 suffered a memory overflow issue in the post-processing step
- Only KNN graph inputs (knn_indices, knn_dists) are passed to leiden_cluster (X=None); a raw feature matrix cannot be substituted directly

## Evidence

- [readme] post-processing: "s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, n_neighbors, njobs = 8, r = r)"
- [methods] leiden clustering: "labels_leiden = scmomat.leiden_cluster(X = None, knn_indices = knn_indices, knn_dists = knn_dists, resolution = resolution)"
- [readme] umap visualization: "umap_op = scmomat.UMAP(n_components = 2, n_neighbors = n_neighbors, min_dist = 0.20, random_state = 0, metric='precomputed', knn_dists=knn_dists, knn_indices=knn_indices)"
- [readme] memory fix: "scMoMaT v0.2.2 **Updated the post-processing step to solve the memory overflow issue**"
