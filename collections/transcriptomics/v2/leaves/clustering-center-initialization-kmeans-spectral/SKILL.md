---
name: clustering-center-initialization-kmeans-spectral
description: Use when training a deep embedding clustering model (e.g. scMGCA's graph
  convolutional autoencoder) where the self-optimizing KL clustering stage needs well-initialized
  cluster centers on latent embeddings of scRNA-seq data.
when_to_use_negative:
- When the clustering stage does not rely on initialized centers (e.g. purely KNN-graph-based
  methods without a KL self-optimization head)
- When only a small subset of cells is available and K-means alone is already stable
  — the spectral comparison adds cost with little benefit
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: scMGCA
  role: Framework whose clustering-head initialization this step implements; selects
    the best strategy from K-means and spectral clustering
  repo: https://pypi.org/project/scMGCA
- name: scikit-learn
  role: Provides the K-means and spectral clustering implementations used to compute
    candidate initial centers
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:b912cf8a9fed27c875aa50eb7d3256989abf8c28fe67ff7e86b6ecf86e71ed1e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCANPY
  - anndata
  - pandas
  - scMGCA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Clustering-center initialization via best-of K-means vs spectral clustering

## Summary

scMGCA initializes its self-optimized clustering centers by running both K-means and spectral clustering on the latent representation and selecting the best strategy between them, providing a stable starting point for the KL-divergence self-optimization stage of the graph convolutional autoencoder.

## When to use

Use when training a deep embedding clustering model (e.g. scMGCA's graph convolutional autoencoder) where the self-optimizing KL clustering stage needs well-initialized cluster centers on latent embeddings of scRNA-seq data.

## When NOT to use

- When the clustering stage does not rely on initialized centers (e.g. purely KNN-graph-based methods without a KL self-optimization head)
- When only a small subset of cells is available and K-means alone is already stable — the spectral comparison adds cost with little benefit

## Inputs

- Latent (embedded) representation of cells from the trained GCN autoencoder (top d highly variable genes, genes not expressed in cells removed)
- Number of target clusters for the clustering head
- Batch partitioning specification for datasets with >25,000 cells

## Outputs

- Initialized clustering centers (from the selected K-means or spectral strategy)
- Initialized cluster assignments used to seed KL-divergence self-optimized clustering

## How to apply

After normalizing the scRNA-seq expression matrix, filtering out genes not expressed in cells, and keeping the top d highly variable genes, train the two-layer GCN autoencoder to obtain the latent representation. Run K-means clustering on this latent space and, in parallel, run spectral clustering; evaluate which strategy yields the better initialization and select it to define the clustering centers feeding the multinomial-based decoder's clustering head. The chosen centers then anchor the self-optimized clustering performed via Kullback-Leibler divergence on the latent representations, jointly optimized with the autoencoder reconstruction losses. For datasets exceeding 25,000 cells (e.g. the 1,306,127-cell mouse brain dataset), perform this initialization per batch as scMGCA divides large datasets into multiple batches for learning. The rationale is that a poor center initialization can trap the KL self-optimization in a bad local optimum, so selecting the best of two candidate strategies de-risks the joint training.

## Related tools

- **scMGCA** (Framework whose clustering-head initialization this step implements; selects the best strategy from K-means and spectral clustering) — https://pypi.org/project/scMGCA
- **scikit-learn** (Provides the K-means and spectral clustering implementations used to compute candidate initial centers)

## Examples

```
from sklearn.cluster import KMeans, SpectralClustering; centers = KMeans(n_clusters=k).fit(z_latent).cluster_centers_  # compare with SpectralClustering(n_clusters=k).fit(z_latent)
```

## Evaluation signals

- Both K-means and spectral clustering candidates were computed on the same latent representation before selection
- The selected centers lead the KL-divergence self-optimization to converge (stopping condition: maximum number of iterations achieved)
- Final cluster assignments for every cell are written out (e.g. scale_clustering_labels.csv for the 1,306,127-cell mouse brain run)
- For >25,000-cell datasets, initialization is applied consistently across the batched learning partitions
- Downstream clustering quality (e.g. ASW on benchmark datasets like QS Trachea and QS Heart) is consistent with the scMGCA paper's reported performance

## Limitations

- The paper does not report numeric values for the loss weights γ1, γ2, γ3 balancing the total objective, so the interaction between initialization choice and loss balance is not fully characterized
- The maximum iteration count for the stopping condition is not specified, so convergence behavior after initialization must be monitored empirically
- The number of highly variable genes d is not given a fixed numeric value, so initialization quality can vary with the choice of d

## Evidence

- [results] Results section support: "we selected the best strategy from K-means and spectral clustering to initialize the clustering centers"
- [results] KL self-optimization stage: "we perform the Kullback-Leibler (KL) divergence on the latent representations to self-optimize the clustering task"
- [discussion] Implementation context: "scMGCA is implemented in Python, and the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA"
- [discussion] Stopping condition: "The stopping condition is that the maximum number of iterations is achieved."
