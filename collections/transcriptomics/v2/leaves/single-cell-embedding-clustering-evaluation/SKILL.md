---
name: single-cell-embedding-clustering-evaluation
description: Use when comparing cell embeddings/clusterings from multiple single-cell
  multi-omics integration methods and need quantitative embedding (silhouette, purity)
  and clustering (ARI across resolutions) metrics.
when_to_use_negative:
- When the method under evaluation provides no cell embedding (e.g., Seurat v4's clustering-only
  output) — the silhouette score cannot be computed for such methods
- When only a single method's embedding is available and no cross-method benchmark
  comparison is intended
edam_operation: http://edamontology.org/operation_3432
tools:
- name: Mowgli
  role: Method under evaluation; produces shared cell embeddings via integrative NMF
    with optimal transport
  repo: github.com/cantinilab/mowgli
- name: Scanpy
  role: Computes kNN graphs (neighbors, k=20, Euclidean) and Leiden clustering across
    resolutions on the embeddings
- name: Seurat v4
  role: Baseline method; clusters with FindClusters; excluded from silhouette scoring
    as it provides no embedding
- name: MOFA+
  role: Baseline method producing factor-based cell embeddings clustered via Scanpy
    kNN + Leiden
- name: Scikit-learn
  role: Provides silhouette_score and adjusted_rand_score used to quantify embedding
    and clustering performance
- name: Muon
  role: Loads and preprocesses the paired multi-omics Muon objects fed to each integration
    method
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:3781d152e5fe29192470cced8d6bb7933e211d118c0ce832a0f47dc7abc0a05c
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# single-cell-embedding-clustering-evaluation

## Summary

Benchmarks cell embeddings and clusterings produced by paired single-cell multi-omics integration methods (Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, Multigrate) using silhouette score, Adjusted Rand Index (ARI) across Leiden resolutions, and kNN purity score, as done in the Mowgli paper across five real datasets (Liu, PBMC 10X, OP Multiome, OP CITE, BM CITE).

## When to use

Use when comparing cell embeddings/clusterings from multiple single-cell multi-omics integration methods and need quantitative embedding (silhouette, purity) and clustering (ARI across resolutions) metrics.

## When NOT to use

- When the method under evaluation provides no cell embedding (e.g., Seurat v4's clustering-only output) — the silhouette score cannot be computed for such methods
- When only a single method's embedding is available and no cross-method benchmark comparison is intended

## Inputs

- Cell embedding matrices from Mowgli, integrative NMF, MOFA+, Cobolt, and Multigrate
- Leiden cluster label vectors across resolutions (Scanpy) and Seurat FindClusters labels
- Preprocessed Muon objects for the five real datasets: Liu/scCAT-seq, PBMC 10X, OP Multiome, OP CITE, BM CITE

## Outputs

- benchmark_metrics.csv — method × dataset × metric comparison table (silhouette score, kNN purity, ARI across resolutions)
- Per-dataset UMAP visualizations of the benchmarked embeddings

## How to apply

For methods that yield cell embeddings (Mowgli, integrative NMF, MOFA+, Cobolt, Multigrate), compute a kNN graph with Scanpy's neighbors using k=20 neighbors and Euclidean distance, then cluster with the Leiden algorithm across varying resolutions; cluster Seurat v4 outputs with Seurat's FindClusters instead. Quantify embedding quality with Scikit-learn's silhouette_score on the embeddings and the kNN purity score, omitting the silhouette score for Seurat v4 since it provides no embedding. Quantify clustering agreement with Scikit-learn's adjusted_rand_score computed across Leiden resolutions. Assemble a method × dataset × metric table (saved as benchmark_metrics.csv) and judge competitiveness by purity and ARI rather than a single metric — e.g., Mowgli outperforms other methods on OP CITE by purity and is comparable to MOFA+ and Multigrate on PBMC 10X. The rationale is that different metrics capture embedding geometry, cluster recovery, and label agreement, so a method is only judged competitive when evaluated across all three.

## Related tools

- **Mowgli** (Method under evaluation; produces shared cell embeddings via integrative NMF with optimal transport) — github.com/cantinilab/mowgli
- **Scanpy** (Computes kNN graphs (neighbors, k=20, Euclidean) and Leiden clustering across resolutions on the embeddings)
- **Seurat v4** (Baseline method; clusters with FindClusters; excluded from silhouette scoring as it provides no embedding)
- **MOFA+** (Baseline method producing factor-based cell embeddings clustered via Scanpy kNN + Leiden)
- **Scikit-learn** (Provides silhouette_score and adjusted_rand_score used to quantify embedding and clustering performance)
- **Muon** (Loads and preprocesses the paired multi-omics Muon objects fed to each integration method)

## Examples

```
sc.pp.neighbors(adata, n_neighbors=20, metric="euclidean"); sc.tl.leiden(adata, resolution=res); silhouette_score(adata.obsm["embedding"], adata.obs["leiden"])
```

## Evaluation signals

- The assembled benchmark_metrics.csv contains a complete method × dataset × metric grid with six methods (Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, Multigrate) and five datasets
- Silhouette score cells are empty/omitted only for Seurat v4, matching the fact that it provides no embedding
- kNN purity results reproduce the expected pattern: Mowgli outperforms other methods on OP CITE and is comparable to MOFA+ and Multigrate on PBMC 10X
- ARI values are computed across multiple Leiden resolutions, not at a single fixed resolution, for each embedding-based method
- Seurat v4 clustering is generated with Seurat's FindClusters rather than Scanpy Leiden, consistent with the original benchmark

## Limitations

- The silhouette score cannot be computed for Seurat v4 because it requires a cell embedding that the method does not provide
- Optimal transport in Mowgli is inherently expensive to compute compared to Euclidean distance; GPU availability is required to scale to the larger datasets
- ARI across resolutions favors different methods than purity/silhouette (e.g., Seurat v4 performs best in PBMC 10X, OP Multiome, and OP CITE), so no single metric is decisive

## Evidence

- [results] We benchmarked Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate based on natural metrics for embedding and clustering performance: silhouette score, Adjusted Rand Index (ARI), and: "We benchmarked Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate based on natural metrics for embedding and clustering performance: silhouette score, Adjusted Rand Index (ARI), and"
- [results] The silhouette score cannot be computed for Seurat because it requires an embedding not provided by this method: "The silhouette score cannot be com- puted for Seurat because it requires an embedding of the cells not provided by this method."
- [abstract] Mowgli's performance is competitive with the state-of-the-art in cell clustering and superior in biological interpretability: "Mowgli's performance is competitive with the state-of-the-art in cell clustering and superior to the state-of-the-art once considering biological interpretability"
- [readme] Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy.: "Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy."
