---
name: graph-autoencoder-model-training
description: Use when scRNA-seq count data must be embedded in a topology-preserving
  low-dimensional space with simultaneous cluster assignment, i.e., a cell graph is
  built and latent features plus cluster labels are learned jointly.
when_to_use_negative:
- Input is already a low-dimensional feature embedding or a non-single-cell dataset
  without cell–cell graph structure — the cell graph construction and topology preservation
  are the core premise
- Only 2D visualization is needed directly — the graph autoencoder embeds into an
  intermediate dimension and t-SNE is used for 2D visualization, so a direct 2D method
  is preferable
- Trajectory inference is the sole goal and clustering performance is irrelevant —
  scGAE matches scGNN on trajectory inference but its advantage is clustering
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scGAE
  role: Multitask-oriented graph autoencoder whose train() and clustering_train()
    routines perform the pre-training and deep-clustering alter-training
  repo: github.com/ZixiangLuo1161/scGAE
- name: scanpy
  role: Python package (v1.8.0) used for single-cell data preprocessing before model
    building
  repo: github.com/ZixiangLuo1161/scGAE
- name: spektral
  role: Python package (v0.6.1) used to implement the graph autoencoder model
  repo: github.com/ZixiangLuo1161/scGAE
- name: t-SNE
  role: Used to project the intermediate-dimension embedding into a 2D visualization
    space after training
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:080ac9accdeb1ed550e03c5ab77fb5bb9b634f799de04b5891e5a3d20421bb0f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGAE
  - scanpy
  - spektral
  - t-SNE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# graph-autoencoder-model-training

## Summary

Trains the scGAE multitask-oriented graph autoencoder on scRNA-seq data: a pre-training stage minimizing the reconstruction loss L_r on a KNN cell graph, followed by deep-clustering alter-training that jointly minimizes L = L_r + gamma*L_c (gamma=2.5) with Student's-t soft assignment and DEC target distribution to yield a clustering-friendly latent embedding.

## When to use

Use when scRNA-seq count data must be embedded in a topology-preserving low-dimensional space with simultaneous cluster assignment, i.e., a cell graph is built and latent features plus cluster labels are learned jointly.

## When NOT to use

- Input is already a low-dimensional feature embedding or a non-single-cell dataset without cell–cell graph structure — the cell graph construction and topology preservation are the core premise
- Only 2D visualization is needed directly — the graph autoencoder embeds into an intermediate dimension and t-SNE is used for 2D visualization, so a direct 2D method is preferable
- Trajectory inference is the sole goal and clustering performance is irrelevant — scGAE matches scGNN on trajectory inference but its advantage is clustering

## Inputs

- Preprocessed scRNA-seq count matrix (e.g., Seurat/scanpy-preprocessed, scanpy/anndata object)
- KNN cell graph adjacency matrix computed with get_adj (default K=35)
- Initial cluster centroids from Louvain clustering of pre-trained embeddings

## Outputs

- Trained scGAE deep-clustering model (model files: scgae.py, layers.py, losses.py, utils.py)
- Latent low-dimensional embedding of cells via model.embedding()
- Per-cell cluster assignments (CSV with one row per cell and a cluster-label column)

## How to apply

First preprocess the scRNA-seq count matrix (e.g., with scTransform in Seurat), then build the KNN cell graph using get_adj with default K=35. Pre-train the SCGAE model with model.train() on the reconstruction loss L_r to obtain the initial embedded representation. Obtain initial cluster centroids by running Louvain clustering on the pre-trained embedded data via get_centers_louvain. Then run alter-training with model.clustering_train(): compute the Student's-t soft assignment Q of embeddings to centroids, derive the DEC target distribution P from Q, and jointly minimize L = L_r + gamma*L_c with gamma=2.5 so that embedding and cluster assignment are learned simultaneously. Finally extract the latent embedding with model.embedding() and export per-cell cluster labels. Rationale: the graph autoencoder preserves cell–cell (topological) relationships that conventional deep-learning embedders ignore, and the DEC-style KL clustering stage reshapes the latent space to be clustering-friendly.

## Related tools

- **scGAE** (Multitask-oriented graph autoencoder whose train() and clustering_train() routines perform the pre-training and deep-clustering alter-training) — github.com/ZixiangLuo1161/scGAE
- **scanpy** (Python package (v1.8.0) used for single-cell data preprocessing before model building) — github.com/ZixiangLuo1161/scGAE
- **spektral** (Python package (v0.6.1) used to implement the graph autoencoder model) — github.com/ZixiangLuo1161/scGAE
- **t-SNE** (Used to project the intermediate-dimension embedding into a 2D visualization space after training)

## Examples

```
model.train(); centers = get_centers_louvain(model.embedding()); model.clustering_train(); emb = model.embedding()
```

## Evaluation signals

- Training script runs end-to-end and the output CSV exists and contains a cluster-label column for every cell
- Louvain clustering on the pre-trained embedded data yields initial centroids before clustering_train() is invoked
- Latent embedding preserves inter-cluster distances and separates discrete cell clusters (assessed on simulated and empirical data such as GSE84133 pancreatic cells or HRA000084 HSPCs)
- Loss during alter-training combines reconstruction and clustering terms as L = L_r + 2.5*L_c with soft assignment Q converging toward target distribution P
- Embedded data lands in an intermediate dimension (not 2D directly), with t-SNE used only for visualization

## Limitations

- The optimal K for KNN graph construction varies among datasets and different parts of a dataset; the default K=35 may need adjustment
- No random seed or stochastic-variation control is reported, so runs may vary stochastically
- The graph autoencoder cannot embed complex data structure directly into two dimensions; an intermediate dimension plus t-SNE is required for visualization
- No software/package versions are stated in the paper itself (README pins tensorflow 2.3.0, spektral 0.6.1, etc.)

## Evidence

- [discussion] the clustering training is performed with model.clustering_train() after obtaining initial cluster centers via Louvain clustering: "In experiments, we use the pre-trained network to generate the initial embedded data which are clustered to obtain the initial centroids by Louvain37."
- [readme] After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function.: "After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function."
- [discussion] The default K is predened as 35 in this study and adjusted according to the datasets in our experiments.: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [discussion] the optimal K varies among different datasets and different parts of a dataset: "the optimal K varies among different datasets and different parts of a dataset."
- [discussion] The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41.: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
