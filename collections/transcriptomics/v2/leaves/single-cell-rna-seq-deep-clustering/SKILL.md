---
name: single-cell-rna-seq-deep-clustering
description: Use when scRNA-seq count data have already been embedded by a pre-trained
  graph autoencoder and cell-level cluster labels are needed, with simultaneous learning
  of embedding and cluster assignment.
when_to_use_negative:
- Only a 2D visualization embedding is needed — the article notes the graph autoencoder
  cannot directly embed into 2D and instead uses an intermediate dimension plus tSNE,
  not the clustering stage
- The model has not been pre-trained on reconstruction loss L_r — centroids are obtained
  by Louvain clustering of the pre-trained embedded data, so skipping pre-training
  gives no valid initialization
- Trajectory inference is the goal rather than discrete clustering — scGAE and scGNN
  perform similarly on trajectory inference, so this skill's advantage is specifically
  clustering performance
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGAE
  role: Provides the multitask-oriented graph autoencoder whose clustering_train()
    performs the deep-clustering alter-training with joint L_r + gamma*L_c loss; repo
    includes get_centers_louvain for centroid initialization
  repo: github.com/ZixiangLuo1161/scGAE
- name: scanpy
  role: Python package for preprocessing single-cell data before model building (README
    requirement, version 1.8.0)
  repo: github.com/ZixiangLuo1161/scGAE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:8e3ec0d89ec87eb681992793b451089be3036a47ca16443188499584a2eef632
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGAE
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# single-cell-rna-seq-deep-clustering

## Summary

Refines a pre-trained scGAE graph autoencoder into a clustering-friendly latent embedding via DEC-style alter-training: Student's-t soft assignment Q to Louvain-initialized centroids, a DEC target distribution P, and joint minimization of L = L_r + gamma*L_c with gamma = 2.5.

## When to use

Use when scRNA-seq count data have already been embedded by a pre-trained graph autoencoder and cell-level cluster labels are needed, with simultaneous learning of embedding and cluster assignment.

## When NOT to use

- Only a 2D visualization embedding is needed — the article notes the graph autoencoder cannot directly embed into 2D and instead uses an intermediate dimension plus tSNE, not the clustering stage
- The model has not been pre-trained on reconstruction loss L_r — centroids are obtained by Louvain clustering of the pre-trained embedded data, so skipping pre-training gives no valid initialization
- Trajectory inference is the goal rather than discrete clustering — scGAE and scGNN perform similarly on trajectory inference, so this skill's advantage is specifically clustering performance

## Inputs

- Pre-trained scGAE model and its latent embedding of preprocessed scRNA-seq count data (input preprocessed with Seurat/scanpy)
- KNN cell graph adjacency matrix (get_adj, default K=35)
- Initial cluster centroids from Louvain clustering of the pre-trained embedded data (get_centers_louvain)

## Outputs

- Trained deep-clustering scGAE model with clustering-friendly latent representation
- CSV file with one row per cell containing a cluster-label column
- Soft assignment matrix Q and DEC target distribution P computed during alter-training

## How to apply

First pre-train the scGAE graph autoencoder on the reconstruction loss L_r (after building the KNN cell graph with default K=35 and preprocessing the count matrix with scTransform), producing an initial embedded representation. Obtain initial cluster centroids by running Louvain clustering (get_centers_louvain) on the pre-trained embedded data. Then run alter-training via model.clustering_train(): compute the Student's-t soft assignment Q of embeddings to the centroids, derive the DEC target distribution P from Q, and jointly minimize L = L_r + gamma*L_c with gamma = 2.5 so embedding and cluster assignment are learned simultaneously. Finally export the trained model, write per-cell cluster assignments to a CSV with one row per cell and a cluster-label column, and validate the script end-to-end (CSV exists, label column covers all cells).

## Related tools

- **scGAE** (Provides the multitask-oriented graph autoencoder whose clustering_train() performs the deep-clustering alter-training with joint L_r + gamma*L_c loss; repo includes get_centers_louvain for centroid initialization) — github.com/ZixiangLuo1161/scGAE
- **scanpy** (Python package for preprocessing single-cell data before model building (README requirement, version 1.8.0)) — github.com/ZixiangLuo1161/scGAE

## Examples

```
adj = get_adj(data, K=35); model.train(); centers = get_centers_louvain(model.embedding()); model.clustering_train(); emb = model.embedding()
```

## Evaluation signals

- Output CSV exists after end-to-end execution and contains a cluster-label column covering all cells (one row per cell)
- model.embedding() returns a latent representation on which Louvain-initialized centroids were obtainable via get_centers_louvain
- Alter-training loss decreases and equals L_r + gamma*L_c with gamma = 2.5, using Q/P from Student's-t soft assignment and DEC target distribution
- Clustering results improve over the pre-training-only initialization (scGAE reports better clustering performance than scGNN on empirical data)
- Documented package versions from the README (tensorflow 2.3.0, spektral 0.6.1, scanpy 1.8.0, etc.) are pinned in the environment

## Limitations

- The optimal K for KNN graph construction varies among different datasets and different parts of a dataset, so K=35 may need adjustment
- No random seed or stochastic-variation control is reported, so clustering runs may vary stochastically
- No software/package versions are specified in the paper itself (versions appear only in the README), and no DOI/changelog metadata is stated in the document

## Evidence

- [readme] clustering training after Louvain initialization: "After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function."
- [discussion] Louvain clustering of pre-trained embedded data: "we use the pre-trained network to generate the initial embedded data which are clustered to obtain the initial centroids by Louvain37."
- [discussion] KNN graph default K: [evidence span withheld — anchor status: not_found]
- [results] simultaneous embedding and clustering: [evidence span withheld — anchor status: not_found]
