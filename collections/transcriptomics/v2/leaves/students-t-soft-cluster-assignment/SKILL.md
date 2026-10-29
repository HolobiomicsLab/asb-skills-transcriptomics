---
name: students-t-soft-cluster-assignment
description: Use when a pre-trained graph autoencoder embedding of scRNA-seq cells
  must be made clustering-friendly by jointly refining embeddings and cluster assignments
  via soft-assignment KL loss with Louvain-initialized centroids.
when_to_use_negative:
- Input is raw, un-preprocessed counts — the stage expects scTransform-preprocessed
  data and a pre-trained model, not raw matrices
- The goal is 2D visualization only — scGAE embeds to an intermediate dimension and
  uses t-SNE for 2D plots; direct 2D embedding by the graph autoencoder is not supported
- Only a first-pass embedding is needed without cluster labels — pre-training with
  model.train() alone suffices; clustering training is explicitly optional in the
  README
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGAE
  role: Provides the graph autoencoder model whose clustering_train() jointly learns
    soft cluster assignments and embeddings; get_centers_louvain supplies initial
    centroids
  repo: github.com/ZixiangLuo1161/scGAE
- name: scanpy
  role: Preprocessing of single-cell data before model building and clustering (listed
    requirement, version 1.8.0)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:8dd5f28c06635c1ddef5397ea16e2fe68d706cc35be1a3a5575e3d7213bef3d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGAE
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# students-t-soft-cluster-assignment

## Summary

DEC-style alter-training stage for scGAE in which Student's-t soft cluster assignments (Q) and a sharpened target distribution (P) are learned jointly with graph-autoencoder reconstruction, yielding a clustering-friendly latent embedding from scRNA-seq data.

## When to use

Use when a pre-trained graph autoencoder embedding of scRNA-seq cells must be made clustering-friendly by jointly refining embeddings and cluster assignments via soft-assignment KL loss with Louvain-initialized centroids.

## When NOT to use

- Input is raw, un-preprocessed counts — the stage expects scTransform-preprocessed data and a pre-trained model, not raw matrices
- The goal is 2D visualization only — scGAE embeds to an intermediate dimension and uses t-SNE for 2D plots; direct 2D embedding by the graph autoencoder is not supported
- Only a first-pass embedding is needed without cluster labels — pre-training with model.train() alone suffices; clustering training is explicitly optional in the README

## Inputs

- Pre-trained scGAE model and its latent embedding of preprocessed scRNA-seq count data
- KNN cell graph adjacency matrix (K=35 default) from the scRNA-seq data
- Initial cluster centroids obtained via Louvain clustering of the embedded data

## Outputs

- Trained deep-clustering scGAE model checkpoint
- Clustering-friendly latent embedding (model.embedding())
- CSV file with one row per cell and a cluster-label column of per-cell cluster assignments

## How to apply

First pre-train the scGAE graph autoencoder on the reconstruction loss L_r using the KNN cell graph (default K=35) built from the scTransform-preprocessed count matrix, producing the initial embedded representation. Obtain initial cluster centroids by running Louvain clustering on the pre-trained embedded data via the get_centers_louvain function. Then run alter-training with model.clustering_train(): compute the Student's-t soft assignment Q of the latent embeddings to the centroids, derive the DEC target distribution P from Q, and jointly minimize L = L_r + gamma*L_c with gamma=2.5, so clustering and embedding are learned simultaneously. Finally export the trained deep-clustering model and write per-cell cluster labels (one row per cell with a cluster-label column) to a CSV. Rationale: jointly optimizing reconstruction and clustering loss preserves topological structure while making the latent space cluster-friendly.

## Related tools

- **scGAE** (Provides the graph autoencoder model whose clustering_train() jointly learns soft cluster assignments and embeddings; get_centers_louvain supplies initial centroids) — github.com/ZixiangLuo1161/scGAE
- **scanpy** (Preprocessing of single-cell data before model building and clustering (listed requirement, version 1.8.0))

## Examples

```
adj = get_adj(data, k=35); model = SCGAE(adj); model.train(); centers = get_centers_louvain(model.embedding()); model.clustering_train(centers)
```

## Evaluation signals

- The end-to-end script executes and the output CSV exists and contains a cluster-label column for all cells
- Clustering training is invoked only after get_centers_louvain has produced initial cluster centers from the pre-trained embedded data
- The joint loss equals L = L_r + gamma*L_c with gamma = 2.5 as specified for alter-training
- Latent embedding is obtainable post-training via model.embedding() and cluster assignments cover one label per cell
- Target distribution P is derived from the Student's-t soft assignment Q of embeddings to centroids (DEC target distribution)

## Limitations

- The optimal K for KNN graph construction varies among different datasets and different parts of a dataset, affecting the graph used in training
- No random seed or stochastic-variation control is reported, so clustering results may vary between runs
- No software/package versions beyond the README requirements are specified in the article; run-time environment must match tensorflow 2.3.0, spektral 0.6.1, etc.

## Evidence

- [other] clustering training is performed with model.clustering_train() after obtaining initial cluster centers via Louvain clustering (get_centers_louvain) of the pre-trained embedded data: [evidence span withheld — anchor status: not_found]
- [readme] After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function.: "After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function."
- [other] scGAE incorporates a deep clustering stage in which the data embedding and cluster assignment are learned simultaneously, yielding a clustering-friendly latent representation: [evidence span withheld — anchor status: not_found]
- [discussion] In experiments, we use the pre-trained network to generate the initial embedded data which are clustered to obtain the initial centroids by Louvain37.: "In experiments, we use the pre-trained network to generate the initial embedded data which are clustered to obtain the initial centroids by Louvain37."
- [discussion] The default K is predened as 35 in this study and adjusted according to the datasets in our experiments.: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
