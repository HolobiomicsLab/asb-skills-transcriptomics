---
name: cell-graph-topology-reconstruction
description: Use when a scRNA-seq count matrix must be embedded in a latent space
  that preserves inter-cell topological/trajectory structure, not just per-cell features
  — e.g. clustering and trajectory inference where inter-cluster distances matter.
when_to_use_negative:
- The goal is a direct 2D embedding from the graph autoencoder itself — the paper
  notes complex data structure is hard to embed directly into two dimensions, so an
  intermediate dimension plus t-SNE is used instead
- Input is already an embedded/visualized representation (e.g. a 2D t-SNE or PCA output)
  rather than a preprocessed count matrix from which a cell graph can be built
- Cell–cell relationships are irrelevant and only feature-level embedding is needed
  — a feature-only autoencoder would suffice
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3473
tools:
- name: scGAE
  role: The multitask-oriented graph autoencoder being reconstructed (scgae.py, layers.py,
    losses.py, utils.py)
  repo: github.com/ZixiangLuo1161/scGAE
- name: Spektral
  role: Python package used to implement the graph attention layers of the scGAE model
- name: TensorFlow
  role: Backend framework (version 2.3.0 per README) on which scGAE is built
- name: scanpy
  role: Python package for single-cell data preprocessing before model building (version
    1.8.0 per README)
- name: t-SNE
  role: Used to visualize the intermediate-dimension scGAE embedding in 2D
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:804bfc6c7ff66bd79694f5714625d40eb39d60e7c1cbd82082cc9a18a12ae9a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Spektral
  - TensorFlow
  - scGAE
  - scanpy
  - t-SNE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cell-graph-topology-reconstruction

## Summary

Reconstruct scGAE, a multitask-oriented graph autoencoder that embeds preprocessed scRNA-seq count matrices while jointly reconstructing both the feature matrix (counts) and the cell–cell KNN adjacency matrix, preserving topological structure that feature-only deep embeddings ignore.

## When to use

Use when a scRNA-seq count matrix must be embedded in a latent space that preserves inter-cell topological/trajectory structure, not just per-cell features — e.g. clustering and trajectory inference where inter-cluster distances matter.

## When NOT to use

- The goal is a direct 2D embedding from the graph autoencoder itself — the paper notes complex data structure is hard to embed directly into two dimensions, so an intermediate dimension plus t-SNE is used instead
- Input is already an embedded/visualized representation (e.g. a 2D t-SNE or PCA output) rather than a preprocessed count matrix from which a cell graph can be built
- Cell–cell relationships are irrelevant and only feature-level embedding is needed — a feature-only autoencoder would suffice

## Inputs

- Preprocessed scRNA-seq count matrix (cells × genes), preprocessed with scTransform via Seurat or scanpy
- KNN cell–cell adjacency matrix computed from the count matrix via get_adj (default K=35)

## Outputs

- Latent cell embedding Z (low-dimensional representation via model.embedding())
- Reconstructed feature (count) matrix from the feature decoder
- Reconstructed cell–cell adjacency matrix A_r = sigma(ZZ^T) from the graph decoder
- 2D visualization of the intermediate-dimension embedding via t-SNE
- Cell cluster assignments after optional clustering training

## How to apply

Preprocess the count matrix with scTransform (Seurat or scanpy), then build a cell graph with a KNN algorithm (default K=35, adjusted per dataset) using the get_adj function. Implement the graph autoencoder with graph attention layers via the Spektral Python package on TensorFlow 2.3.0: a GAT encoder E(X, A) maps cells to a latent embedding Z, a 4-layer fully connected feature decoder (64, 256, 512 hidden nodes) reconstructs the count matrix, and an edge decoder A_r = sigma(ZZ^T) reconstructs the adjacency. Train with the joint reconstruction loss L = L_X + lambda·L_A (lambda = 0.6) so feature information and topology are preserved simultaneously. Optionally obtain initial centroids with get_centers_louvain (Louvain clustering of pre-trained embeddings) and run model.clustering_train(), then extract the low-dimensional representation with model.embedding(); embed to intermediate dimension and use t-SNE for 2D visualization rather than forcing a direct 2D graph-autoencoder embedding.

## Related tools

- **scGAE** (The multitask-oriented graph autoencoder being reconstructed (scgae.py, layers.py, losses.py, utils.py)) — github.com/ZixiangLuo1161/scGAE
- **Spektral** (Python package used to implement the graph attention layers of the scGAE model)
- **TensorFlow** (Backend framework (version 2.3.0 per README) on which scGAE is built)
- **scanpy** (Python package for single-cell data preprocessing before model building (version 1.8.0 per README))
- **t-SNE** (Used to visualize the intermediate-dimension scGAE embedding in 2D)

## Examples

```
A = get_adj(X, k=35); model = SCGAE(A); model.train(); Z = model.embedding()
```

## Evaluation signals

- Forward-pass smoke test: latent embedding, reconstructed count matrix (e.g. 100 × 1000) and reconstructed adjacency (100 × 100) tensors have correct dimensions and all values are finite
- The joint loss L = L_X + 0.6·L_A decreases during training and both reconstruction terms improve
- The reconstructed adjacency A_r recovers the KNN (K=35) cell-graph structure, i.e. inter-cluster distances and trajectory topology are preserved in the embedding
- Downstream check: scGAE embedding accurately reconstructs developmental trajectory and separates discrete cell clusters compared to competing deep methods
- Reproduce the README workflow order: get_adj → SCGAE model → model.train() → optional get_centers_louvain + model.clustering_train() → model.embedding()

## Limitations

- The optimal K for KNN graph construction varies among different datasets and different parts of a dataset; the default K=35 must be tuned
- A direct 2D embedding by the graph autoencoder is not feasible; an intermediate dimension plus t-SNE is required, and the paper notes a better 2D visualization method is still needed
- No random seed or stochastic-variation control was reported, so runs may vary
- No software/package versions are specified in the paper itself; only the README pins versions (e.g. tensorflow 2.3.0, spektral 0.6.1)

## Evidence

- [abstract] scGAE decodes with two decoders preserving feature and topology: "uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information in scRNA-seq data simultaneously"
- [discussion] Default KNN K is 35: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [discussion] Direct 2D embedding infeasible: "because the complex data structure is hard to be directly embedded into two-dimensional space by graph autoencoder"
- [intro] Deep methods ignore cell–cell relationships: "these deep learning-based models only embedded the distinct cell features while ignoring the cell–cell relationships, which limited their ability to reveal the complex topological structure"
- [readme] README model-building workflow: "Compute the adjacency matrix using get_adj function and build the SCGAE model."
- [discussion] Spektral used for implementation: "We used the Python package Spektral32 to implement our model."
