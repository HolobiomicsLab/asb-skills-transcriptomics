---
name: single-cell-rnaseq-dimensionality-reduction
description: Use when you have a raw scRNA-seq count matrix (e.g., GSE84133 Baron
  pancreas) and need a low-dimensional embedding that preserves inter-population centroid
  distances and developmental trajectory structure rather than only local neighbor
  structure.
when_to_use_negative:
- When only a cheap 2D visualization of local neighborhoods is needed and inter-population
  topological structure is not a concern — plain t-SNE suffices and scGAE adds graph-building
  and training overhead
- 'When the goal is direct 2D embedding: the graph autoencoder cannot embed complex
  data structure directly into two dimensions; it yields an intermediate dimension
  that still requires t-SNE for visualization'
- When a matching K for the KNN graph cannot be justified — the optimal K varies among
  datasets and parts of a dataset and is not resolved by the method
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scGAE
  role: 'Core method: builds the cell graph and trains the multitask-oriented graph
    autoencoder to produce the topology-preserving latent embedding'
  repo: https://github.com/ZixiangLuo1161/scGAE
- name: Spektral
  role: Python package used to implement the graph autoencoder model layers and training
- name: Seurat (scTransform)
  role: Preprocessing of the scRNA-seq count matrix before graph construction (scTransform
    in the Seurat R package)
- name: t-SNE
  role: Projects the intermediate-dimensional scGAE embedding to 2D for visualization
- name: Louvain
  role: Clusters the initial embedded data to obtain initial centroids for optional
    clustering training
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:9b832c77be23c3e87692bfc9e326e68f2cd0515771255d9789e9440115b0f6b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZixiangLuo1161/scGAE
  tools:
  - Louvain
  - Seurat (scTransform)
  - Spektral
  - scGAE
  - t-SNE
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# single-cell-rnaseq-dimensionality-reduction

## Summary

Apply scGAE, a single-cell graph autoencoder, to reduce scRNA-seq count matrices to a low-dimensional (e.g., 20-D) embedding that preserves the topological/inter-cluster structure among cell populations, outperforming t-SNE, scVI, DCA, PHATE and other deep-learning methods on trajectory and cluster-distance preservation.

## When to use

Use when you have a raw scRNA-seq count matrix (e.g., GSE84133 Baron pancreas) and need a low-dimensional embedding that preserves inter-population centroid distances and developmental trajectory structure rather than only local neighbor structure.

## When NOT to use

- When only a cheap 2D visualization of local neighborhoods is needed and inter-population topological structure is not a concern — plain t-SNE suffices and scGAE adds graph-building and training overhead
- When the goal is direct 2D embedding: the graph autoencoder cannot embed complex data structure directly into two dimensions; it yields an intermediate dimension that still requires t-SNE for visualization
- When a matching K for the KNN graph cannot be justified — the optimal K varies among datasets and parts of a dataset and is not resolved by the method

## Inputs

- Raw or preprocessed scRNA-seq count matrix (e.g., Baron human pancreas from GEO GSE84133, downloaded via scTransform-preprocessed Seurat output)
- Preprocessed normalized expression matrix (scTransform/Seurat or scanpy output) as input to get_adj

## Outputs

- 20-dimensional latent embedding of cells (embedding_coordinates.csv) from model.embedding()
- Cell adjacency matrix from get_adj
- 2D t-SNE projection of the intermediate-dimensional embedding
- Pairwise population centroid distance table (centroid_distances.csv) for topology-preservation checks

## How to apply

Preprocess the count matrix with scTransform (Seurat R package or scanpy) as scGAE expects preprocessed data. Compute the cell-cell adjacency with the get_adj function, building a KNN graph with default K=35 (adjusted per dataset). Build and train the multitask-oriented graph autoencoder (implemented with TensorFlow 2.3.0 and the Spektral 0.6.1 package) to embed cells into an intermediate/20-dimensional latent space; optionally run clustering training by obtaining initial centroids with get_centers_louvain (Louvain clustering) and calling model.clustering_train(). Extract the latent embedding via model.embedding(), and for 2D visualization project the intermediate embedding with t-SNE since the graph autoencoder does not directly embed into 2D. To verify topology preservation, compute population centroids in the 20-D space and all pairwise Euclidean centroid distances (e.g., the 14 Baron pancreas populations) and check that transcriptionally similar pairs (activated vs quiescent stellate cells, ductal subtypes) are at short distances.

## Related tools

- **scGAE** (Core method: builds the cell graph and trains the multitask-oriented graph autoencoder to produce the topology-preserving latent embedding) — https://github.com/ZixiangLuo1161/scGAE
- **Spektral** (Python package used to implement the graph autoencoder model layers and training)
- **Seurat (scTransform)** (Preprocessing of the scRNA-seq count matrix before graph construction (scTransform in the Seurat R package))
- **t-SNE** (Projects the intermediate-dimensional scGAE embedding to 2D for visualization)
- **Louvain** (Clusters the initial embedded data to obtain initial centroids for optional clustering training)

## Examples

```
import preprocessing, scgae; adj = preprocessing.get_adj(data); model = scgae.SCGAE(adj, data); model.train(); centers = preprocessing.get_centers_louvain(model.embedding()); model.clustering_train(centers); emb = model.embedding()
```

## Evaluation signals

- On Baron pancreas (GSE84133), compute the 14 population centroids in the 20-D embedding and all pairwise Euclidean distances: the activated–quiescent stellate pair and the two ductal subtypes must show shorter centroid distances than unrelated population pairs (Fig. 4c,d pattern)
- scGAE's t-SNE projection of ductal subtypes must keep them close, whereas t-SNE alone projected them far apart
- Pipeline sanity: adjacency from get_adj with K=35 (default, dataset-adjusted), then model.train(), optional model.clustering_train(), then model.embedding() must all run with the pinned versions (tensorflow 2.3.0, spektral 0.6.1, scanpy 1.8.0)
- Embedding dimension matches the expected intermediate/20-dimensional latent space saved as embedding_coordinates.csv
- Simulated-data check: discrete cell clusters are separated and developmental trajectory is accurately reconstructed, matching reported outperformance over DCA, scVI, scDeepCluster, PHATE, SAUCIE, scGNN, ZINB-WaVE and Ivis

## Limitations

- The optimal K for KNN graph construction varies among datasets and different parts of a dataset and remains unresolved
- The graph autoencoder cannot directly embed complex structure into 2D; an intermediate dimension plus t-SNE is used, and a better 2D visualization method than tSNE is still needed
- No random seed or stochastic-variation control is reported, and no software/package versions were specified in the paper (versions only from the README requirements list)

## Evidence

- [abstract] we present the single-cell graph autoencoder (scGAE), a dimensionality reduction method that preserves topological structure in scRNA-seq data: "we present the single-cell graph autoencoder (scGAE), a dimensionality reduction method that preserves topological structure in scRNA-seq data"
- [discussion] The default K is predened as 35 in this study and adjusted according to the datasets in our experiments.: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [discussion] The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41.: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
- [discussion] because the complex data structure is hard to be directly embedded into two-dimensional space by graph autoencoder: "because the complex data structure is hard to be directly embedded into two-dimensional space by graph autoencoder"
- [readme] scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information in scRNA-seq data simultaneously.: "scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information in scRNA-seq data simultaneously."
