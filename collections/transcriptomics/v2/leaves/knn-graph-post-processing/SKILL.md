---
name: knn-graph-post-processing
description: Use when you have extracted cell factors (zs) from a first-stage scMoMaT
  model and need a KNN graph plus Leiden cluster labels for benchmarking (NMI/ARI/GC)
  or for second-stage retraining.
when_to_use_negative:
- The goal is to obtain cluster-specific biomarkers — that requires the second-stage
  scmomat_retrain step with labels and extract_feature_factors(), not the KNN post-processing
  alone.
- The input is a precomputed distance or adjacency graph rather than raw cell factor
  embeddings — post_process expects the zs cell factors directly.
- The analysis needs cross-batch cell factor alignment rather than neighborhood construction
  (e.g. consider alternative alignment methods such as LIGER's centroidAlign).
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMoMaT
  role: Provides scmomat.post_process to build the shared-neighbor KNN graph from
    cell factors, and scmomat.leiden_cluster to cluster it at resolution 0.9
  repo: https://github.com/PeterZZQ/scMoMaT
- name: UMAP
  role: Used with metric='precomputed' on s_pair_dist (n_neighbors=30, min_dist=0.20)
    to visualize the post-processed graph
- name: leidenalg
  role: Leiden clustering dependency invoked by scmomat.leiden_cluster on the knn_indices/knn_dists
    graph
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:d0bf164330e2e4f28b267f6859f4133703f44803d26546adb88130030a2be107
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - NumPy
  - PyTorch
  - UMAP
  - leidenalg
  - scMoMaT
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# knn-graph-post-processing

## Summary

Post-process scMoMaT first-stage cell factors into a shared-neighbor KNN graph and Leiden-cluster it to obtain cell labels. This step converts the latent embedding (zs) into discrete clustering labels that are evaluated with NMI, ARI, and graph connectivity.

## When to use

Use when you have extracted cell factors (zs) from a first-stage scMoMaT model and need a KNN graph plus Leiden cluster labels for benchmarking (NMI/ARI/GC) or for second-stage retraining.

## When NOT to use

- The goal is to obtain cluster-specific biomarkers — that requires the second-stage scmomat_retrain step with labels and extract_feature_factors(), not the KNN post-processing alone.
- The input is a precomputed distance or adjacency graph rather than raw cell factor embeddings — post_process expects the zs cell factors directly.
- The analysis needs cross-batch cell factor alignment rather than neighborhood construction (e.g. consider alternative alignment methods such as LIGER's centroidAlign).

## Inputs

- Cell factor embedding zs (list of numpy arrays from model.extract_cell_factors())
- Neighborhood size k (e.g. 30, tested over {15,30,50})
- Optional neighborhood radius r (in [0,1], tested over {0.7,0.9,1}, or None)
- Leiden clustering resolution (0.9)

## Outputs

- s_pair_dist (pairwise distance matrix for UMAP visualization)
- knn_indices and knn_dists defining the post-processed KNN graph
- labels_leiden (Leiden cluster assignments at resolution 0.9)
- UMAP 2-D embedding of the graph (x_umap)

## How to apply

Take the cell factors zs returned by model.extract_cell_factors() after first-stage training (K=30 latent dimensions, T=4000 iterations, lambda=0.001 by default) and call scmomat.post_process with a neighborhood size n_neighbors of 30–50, njobs=8, and optionally a neighborhood radius r between 0 and 1 (or r=None to skip the radius constraint). The function returns s_pair_dist, knn_indices, and knn_dists. Feed the KNN graph into scmomat.leiden_cluster at resolution=0.9 to obtain labels_leiden, and optionally visualize with UMAP using metric='precomputed' on s_pair_dist (n_neighbors=30, min_dist=0.20). Sensitivity analysis on the human PBMC dataset shows results are robust to lambda, k, and r, so moderate defaults are defensible; NMI/ARI stabilize once latent dimension d is large enough (e.g. K=30).

## Related tools

- **scMoMaT** (Provides scmomat.post_process to build the shared-neighbor KNN graph from cell factors, and scmomat.leiden_cluster to cluster it at resolution 0.9) — https://github.com/PeterZZQ/scMoMaT
- **UMAP** (Used with metric='precomputed' on s_pair_dist (n_neighbors=30, min_dist=0.20) to visualize the post-processed graph)
- **leidenalg** (Leiden clustering dependency invoked by scmomat.leiden_cluster on the knn_indices/knn_dists graph)

## Examples

```
s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, n_neighbors=30, njobs=8, r=None); labels_leiden = scmomat.leiden_cluster(X=None, knn_indices=knn_indices, knn_dists=knn_dists, resolution=0.9)
```

## Evaluation signals

- Leiden labels computed against reference cell type annotations (original Mimitou et al. labels for PBMC) should yield high NMI and ARI.
- Graph connectivity (GC) of the post-processed graph should be high, indicating well-connected clusters.
- Sweeping k over {15,30,50} and r over {0.7,0.9,1} on the PBMC dataset should produce stable NMI/ARI — large metric swings indicate a mis-parameterized graph.
- n_neighbors must lie within the README-recommended range of 30–50, and the radius (if used) must lie between 0 and 1.
- The shapes of knn_indices/knn_dists must match the total cell count across all batches in the mosaic.

## Limitations

- A neighborhood radius that is too restrictive on a low-quality embedding can fragment the graph; r=None (radius-free post-processing) is a documented default.
- Post-processing operates on the first-stage cell factors only; its quality therefore depends on the latent dimension K (NMI/ARI plateau only once d is large enough for the dataset, per the PBMC sensitivity analysis).
- The feature/cluster assignment stage (biomarker detection) is not part of this step and requires second-stage retraining with scmomat_retrain.

## Evidence

- [methods] Post-process cell factors into KNN graph: "s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, n_neighbors, njobs = 8, r = r)"
- [methods] Leiden clustering on post-processed graph: "labels_leiden = scmomat.leiden_cluster(X = None, knn_indices = knn_indices, knn_dists = knn_dists, resolution = resolution)"
- [readme] Recommended neighborhood size range: "# neighborhood size (30~50)"
- [readme] Optional radius parameter: "# [Optional] neighborhood radius (between 0 and 1)"
