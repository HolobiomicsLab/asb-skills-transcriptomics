---
name: graph-autoencoder-training-on-single-cell-rna-seq
description: Use when you have a preprocessed single-cell RNA-seq count matrix and
  need a topology-preserving low-dimensional embedding for cell clustering or trajectory/visualization
  tasks where deep methods like DCA, scVI, or scDeepCluster lose cell-cell relationships.
when_to_use_negative:
- When the goal is a direct 2D embedding from the graph autoencoder itself - the authors
  note the complex data structure is hard to embed directly into two dimensions, so
  an intermediate-dimension embedding plus t-SNE is required instead.
- When input is already a low-dimensional feature/embedding table - scGAE requires
  the raw (preprocessed) count matrix to build the KNN cell graph.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scGAE
  role: Core graph autoencoder that builds a cell graph and trains the multitask-oriented
    autoencoder to embed cells
  repo: github.com/ZixiangLuo1161/scGAE
- name: Spektral
  role: Python package used to implement the scGAE graph autoencoder model
- name: Louvain
  role: Clustering of initial embedded data to obtain initial centroids before clustering
    training
- name: t-SNE
  role: 2D visualization of the intermediate-dimension embedding
- name: scanpy
  role: Python preprocessing of single-cell data into the format scGAE expects
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:b2600358b10f4ac9c5bc727a56123750bbff56634bfed5740a5dc30b452a73e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Louvain
  - Spektral
  - scGAE
  - scanpy
  - t-SNE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# graph-autoencoder-training-on-single-cell-rna-seq

## Summary

Train scGAE, a multitask-oriented graph autoencoder (implemented with the Python package Spektral), on preprocessed scRNA-seq count matrices to produce low-dimensional embeddings that preserve both topological cell-cell structure and feature information, for downstream clustering (e.g. Louvain) and NMI evaluation.

## When to use

Use when you have a preprocessed single-cell RNA-seq count matrix and need a topology-preserving low-dimensional embedding for cell clustering or trajectory/visualization tasks where deep methods like DCA, scVI, or scDeepCluster lose cell-cell relationships.

## When NOT to use

- When the goal is a direct 2D embedding from the graph autoencoder itself - the authors note the complex data structure is hard to embed directly into two dimensions, so an intermediate-dimension embedding plus t-SNE is required instead.
- When input is already a low-dimensional feature/embedding table - scGAE requires the raw (preprocessed) count matrix to build the KNN cell graph.

## Inputs

- Preprocessed scRNA-seq count matrix (per-cell gene expression; preprocessing via Seurat/scTransform or scanpy)
- Ground-truth cell labels (for NMI evaluation on datasets with known clusters)
- KNN graph parameter K (default 35)

## Outputs

- Low-dimensional cell embedding (model.embedding()) preserving global and local topological structure
- Predicted cell cluster labels from Louvain clustering / clustering training
- Clustering quality metrics such as NMI against ground-truth labels

## How to apply

First preprocess the count matrix with scTransform (Seurat) or scanpy, since scGAE takes preprocessed data as input. Compute the cell graph adjacency matrix with the KNN-based get_adj function (default K=35, adjustable per dataset), then build the scGAE model from preprocessing.py, scgae.py, layers.py, losses.py, and utils.py and train it with model.train() using the Spektral implementation. After obtaining initial centroids via get_centers_louvain (Louvain clustering), optionally refine embeddings with model.clustering_train(), then extract the latent embedding with model.embedding(). Embeddings can be taken at an intermediate dimension for downstream clustering/trajectory inference, or pushed to 2D with t-SNE for visualization. Judge success by clustering agreement with ground truth (e.g. NMI within ~0.55-0.70 tolerance of reported values such as 0.62-0.65 on simulated data) and by preserved inter-cluster distances and developmental trajectories.

## Related tools

- **scGAE** (Core graph autoencoder that builds a cell graph and trains the multitask-oriented autoencoder to embed cells) — github.com/ZixiangLuo1161/scGAE
- **Spektral** (Python package used to implement the scGAE graph autoencoder model)
- **Louvain** (Clustering of initial embedded data to obtain initial centroids before clustering training)
- **t-SNE** (2D visualization of the intermediate-dimension embedding)
- **scanpy** (Python preprocessing of single-cell data into the format scGAE expects)

## Examples

```
from scgae import get_adj; adj = get_adj(data); model = SCGAE(); model.train(); centers = get_centers_louvain(model.embedding()); model.clustering_train(); emb = model.embedding()
```

## Evaluation signals

- NMI between Louvain-predicted clusters and ground-truth labels falls within the reported tolerance band (e.g. 0.55-0.70 around values 0.62, 0.62, 0.65, 0.61 for dropout rates 12%, 17%, 23%, 30%)
- Adjacency matrix is built with the KNN graph construction using K=35 (or a justified per-dataset K)
- Pipeline stages match the README sequence: get_adj -> model.train() -> get_centers_louvain -> model.clustering_train() -> model.embedding()
- Environment matches pinned versions (tensorflow 2.3.0, spektral 0.6.1, scanpy 1.8.0, anndata 0.7.4)
- Embedded data preserves inter-cluster distances and reconstructs developmental trajectories on simulated/empirical datasets

## Limitations

- The optimal K for KNN cell-graph construction varies among datasets and across parts of a dataset, and remains unresolved.
- No random seed or stochastic-variation control is reported, so clustering results may vary between runs.
- No software/package versions were specified in the paper itself (versions only appear in the README requirements).
- The graph autoencoder cannot directly produce 2D embeddings; an intermediate-dimension embedding plus t-SNE is needed, and a better visualization method than t-SNE is still needed.
- NMI is stochastic and depends on Louvain initialization; expected values are approximate (e.g. ~0.61-0.65 under 12-30% dropout).

## Evidence

- [abstract] scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information in scRNA-seq data simultaneously: "scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information in scRNA-seq data simultaneously"
- [discussion] default K=35: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [readme] Louvain clustering training: "After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function."
- [discussion] Spektral implementation: "We used the Python package Spektral32 to implement our model."
- [other] NMI robustness to dropout: [evidence span withheld — anchor status: not_found]
