---
name: leiden-clustering-of-low-dimensional-embeddings
description: Use when you have a low-dimensional cell embedding (e.g., a Mowgli, integrative
  NMF, or MOFA+ latent space) and need cell clusters to quantify embedding and clustering
  performance.
when_to_use_negative:
- The method provides cluster labels but no cell embedding (e.g., Seurat v4) — silhouette
  score requires an embedding and cannot be computed, so only ARI and purity apply
- The input is raw count or preprocessed omics data rather than an already-trained
  low-dimensional embedding; dimensionality reduction must happen first
edam_operation: http://edamontology.org/operation_3935
tools:
- name: Scanpy
  role: Builds the kNN graph (neighbors, k=20, Euclidean) and runs Leiden clustering
    on the embeddings
  repo: https://github.com/cantinilab/mowgli
- name: Mowgli
  role: Provides the shared cell embedding across modalities that is clustered and
    evaluated
  repo: https://github.com/cantinilab/mowgli
- name: Seurat v4
  role: Benchmarked via FindClusters clustering; compared with ARI and purity but
    not silhouette
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:0a247ea8373667da8a0298172d880cdce5c43d75f0c3a4686a54d4811c4272de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/mowgli
  tools:
  - Cobolt
  - MOFA+
  - Mowgli
  - Multigrate
  - Python
  - Scanpy
  - Seurat v4
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# Leiden clustering of low-dimensional embeddings

## Summary

Clusters cell embeddings from multi-omics integration methods by building a k-nearest-neighbor graph (k=20, Euclidean) on the latent space and running Leiden community detection across resolutions, so that methods like Mowgli, integrative NMF, and MOFA+ can be scored with silhouette score, Adjusted Rand Index (ARI), and kNN purity.

## When to use

Use when you have a low-dimensional cell embedding (e.g., a Mowgli, integrative NMF, or MOFA+ latent space) and need cell clusters to quantify embedding and clustering performance.

## When NOT to use

- The method provides cluster labels but no cell embedding (e.g., Seurat v4) — silhouette score requires an embedding and cannot be computed, so only ARI and purity apply
- The input is raw count or preprocessed omics data rather than an already-trained low-dimensional embedding; dimensionality reduction must happen first

## Inputs

- Cell-by-latent-factor embedding matrix from Mowgli (e.g., 5 latent factors for controlled settings, 50 for full-scale runs)
- Cell-by-latent-factor embedding from integrative NMF (TorchNMF)
- Cell-by-latent-factor embedding from MOFA+
- Cluster labels from Seurat v4 FindClusters (no embedding, used for ARI/purity only)

## Outputs

- Leiden cluster assignments per cell at each tested resolution
- Silhouette score, maximum ARI (at low resolutions), and kNN purity per method per setting, compiled into a comparison table (e.g., controlled_benchmark_metrics.csv)

## How to apply

Build a kNN graph on the embedding with Scanpy's neighbors function using k=20 neighbors and Euclidean distance, then cluster with the Leiden algorithm while scanning multiple resolutions. Record the maximum ARI attained at low resolutions so the comparison is fair across methods with different granularity defaults. Cluster Seurat v4 outputs separately with FindClusters since that method provides no cell embedding, which also means silhouette score cannot be computed for it. Score each embedding with scikit-learn's adjusted_rand_score, silhouette_score, and the kNN purity score, then tabulate all three metrics per method and dataset/setting.

## Related tools

- **Scanpy** (Builds the kNN graph (neighbors, k=20, Euclidean) and runs Leiden clustering on the embeddings) — https://github.com/cantinilab/mowgli
- **Mowgli** (Provides the shared cell embedding across modalities that is clustered and evaluated) — https://github.com/cantinilab/mowgli
- **Seurat v4** (Benchmarked via FindClusters clustering; compared with ARI and purity but not silhouette)

## Examples

```
import scanpy as sc; sc.pp.neighbors(adata, n_neighbors=20, use_rep='X_mowgli'); sc.tl.leiden(adata, resolution=0.5, key_added='leiden')
```

## Evaluation signals

- Mowgli's clustering performance should be greater than or equal to all benchmarked methods in every controlled setting except the 90% sparse dataset (Fig. 2 expectation)
- Silhouette score must be computable for Mowgli, integrative NMF, and MOFA+ but reported as not computable for Seurat v4
- In the 90% sparse dataset, integrative NMF should show a better silhouette score than Mowgli while matching it in ARI and purity
- Maximum ARI should be recorded at low Leiden resolutions, and varying the resolution should change the number of detected populations

## Limitations

- Seurat v4 provides no cell embedding, so the silhouette score cannot be computed for it, restricting metric comparability
- Mowgli does not offer a straightforward approach to defining the number of latent dimensions (k); several values of k should be tested before clustering
- Optimal transport in Mowgli is expensive to compute; GPU availability is required for Mowgli to scale to the larger datasets presented

## Evidence

- [results] Benchmarked methods using silhouette, ARI, purity: "We benchmarked Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate based on natural metrics for embedding and clustering performance: silhouette score, Adjusted Rand Index (ARI), and"
- [results] Seurat silhouette not computable: "The silhouette score cannot be com- puted for Seurat because it requires an embedding of the cells not provided by this method."
- [results] 90% sparse exception: "In the 90% sparse dataset, integrative NMF has a better silhouette score than Mowgli but the same ARI and purity score."
- [readme] Mowgli is a scverse Python package: "Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy."
