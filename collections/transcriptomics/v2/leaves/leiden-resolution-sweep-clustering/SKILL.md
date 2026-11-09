---
name: leiden-resolution-sweep-clustering
description: Use when you have a numeric cell-embedding matrix (e.g. from Mowgli,
  MOFA+, or integrative NMF) and need clusterings at multiple granularities to compute
  resolution-dependent clustering metrics such as ARI.
when_to_use_negative:
- The input comes from Seurat v4 WNN, which provides no cell embedding and no silhouette
  score — cluster with Seurat's FindClusters instead
- The data are raw counts or preprocessed Muon objects rather than a fitted embedding
  — run the integration method (Mowgli, MOFA+, etc.) first
- Only a single fixed cluster granularity is needed with no resolution-dependent metric
  — a single Leiden call suffices
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Builds the kNN graph (neighbors, k=20, Euclidean) and runs Leiden clustering
    across resolutions on the embedding
- name: Scikit-learn
  role: Computes silhouette_score on the embedding and adjusted_rand_score across
    resolutions
- name: Mowgli
  role: Source of the cell embedding (shared W matrix from integrative NMF + optimal
    transport) being clustered
  repo: github.com/cantinilab/mowgli
- name: Muon
  role: Provides the preprocessed Muon objects (paired RNA/ADT/ATAC) whose embeddings
    are clustered
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:8671f7c85b032f1425e4ffbb9f3d27aef320e61d6d17bc3d3a0bd993d9cb3ecd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cobolt
  - MOFA+
  - Mowgli
  - Multigrate
  - Muon
  - Python
  - Scanpy
  - Scikit-learn
  - Seurat v4
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# Leiden resolution sweep clustering

## Summary

Clusters cell embeddings by building a kNN graph (k=20, Euclidean distance) with Scanpy and running Leiden across a sweep of resolutions so that benchmark metrics like ARI can be computed across cluster granularities, as done in the Mowgli vs. state-of-the-art benchmark on paired single-cell multi-omics datasets.

## When to use

Use when you have a numeric cell-embedding matrix (e.g. from Mowgli, MOFA+, or integrative NMF) and need clusterings at multiple granularities to compute resolution-dependent clustering metrics such as ARI.

## When NOT to use

- The input comes from Seurat v4 WNN, which provides no cell embedding and no silhouette score — cluster with Seurat's FindClusters instead
- The data are raw counts or preprocessed Muon objects rather than a fitted embedding — run the integration method (Mowgli, MOFA+, etc.) first
- Only a single fixed cluster granularity is needed with no resolution-dependent metric — a single Leiden call suffices

## Inputs

- Cell embedding matrix from Mowgli, integrative NMF, or MOFA+ (cells x latent dimensions, e.g. 50 dims or 5 for Liu-derived data)
- Reference cell labels/annotations (e.g. from Azimuth RNA-based annotation or cell-line identity in scCAT-seq) for ARI and purity computation

## Outputs

- Leiden cluster assignments per cell for each resolution value in the sweep
- Benchmark metrics table (e.g. benchmark_metrics.csv) with silhouette score, ARI across resolutions, and kNN purity score per method x dataset

## How to apply

Take the cell embedding produced by the integration method (e.g. the shared W matrix from Mowgli trained with entropic-regularized OT, typically 50 latent dimensions, or 5 for the Liu/scCAT-seq-derived data). Build a k-nearest-neighbors graph on the embedding with Scanpy's neighbors function using k=20 and Euclidean distance, mirroring the Mowgli paper's benchmark protocol. Run the Leiden algorithm over a sweep of resolution values, storing each resolution's cluster assignment under a distinct key. Compute the Adjusted Rand Index across resolutions with Scikit-learn's adjusted_rand_score against reference labels, plus silhouette_score on the embedding and a kNN purity score. Do not apply this to Seurat v4 outputs, which have no embedding; cluster Seurat results instead with Seurat's FindClusters.

## Related tools

- **Scanpy** (Builds the kNN graph (neighbors, k=20, Euclidean) and runs Leiden clustering across resolutions on the embedding)
- **Scikit-learn** (Computes silhouette_score on the embedding and adjusted_rand_score across resolutions)
- **Mowgli** (Source of the cell embedding (shared W matrix from integrative NMF + optimal transport) being clustered) — github.com/cantinilab/mowgli
- **Muon** (Provides the preprocessed Muon objects (paired RNA/ADT/ATAC) whose embeddings are clustered)

## Examples

```
import scanpy as sc; sc.pp.neighbors(adata, n_neighbors=20, metric='euclidean'); sc.tl.leiden(adata, resolution=1.0, key_added='leiden_1.0')
```

## Evaluation signals

- A kNN graph with exactly k=20 neighbors and Euclidean distance was built on the embedding before Leiden clustering
- Multiple Leiden clusterings are produced, one per resolution value, each stored under a distinct AnnData key
- ARI across resolutions is computed with sklearn's adjusted_rand_score and compared against the paper's pattern (e.g. Seurat v4 best on PBMC 10X, OP Multiome, OP CITE; Mowgli comparable to Seurat v4 on BM CITE)
- Silhouette score is reported for embedding-based methods (Mowgli, MOFA+, integrative NMF, Cobolt, Multigrate) but omitted for Seurat v4
- Purity score computed on the embedding matches or exceeds the paper's reported pattern (Mowgli best on OP CITE, comparable to MOFA+ and Multigrate on PBMC 10X)

## Limitations

- The silhouette score cannot be computed for Seurat v4 because it requires a cell embedding that method does not provide
- Clustering quality depends on the embedding, which in turn depends on the number of latent dimensions; the paper notes Mowgli offers no straightforward way to choose k, so several values should be tested
- ARI is resolution-dependent, so a method can rank differently at different resolutions; the benchmark therefore reports ARI across resolutions rather than at a single point

## Evidence

- [results] We benchmarked Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate based on natural metrics for embedding and clustering performance: "We benchmarked Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate based on natural metrics for embedding and clustering performance"
- [readme] Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy.: "Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy."
- [results] The silhouette score cannot be computed for Seurat because it requires an embedding of the cells not provided by this method.: "The silhouette score cannot be com- puted for Seurat because it requires an embedding of the cells not provided by this method."
- [results] For each method, we choose the overall best-performing number of latent dimensions (see Methods): "For each method, we choose the overall best-performing number of latent dimensions (see Methods)"
