---
name: multi-objective-loss-optimization
description: 'Use when a graph autoencoder latent embedding must be refined for clustering:
  after pre-training on reconstruction loss alone, when initial cluster centroids
  (e.g. from Louvain on embedded data) are available and soft-assignment KL refinement
  is needed.'
when_to_use_negative:
- The model has not been pre-trained on the reconstruction loss L_r — alter-training
  requires initial centroids from Louvain on pre-trained embedded data
- The goal is pure reconstruction/dimensionality reduction without cluster labels;
  model.train() on L_r alone suffices (clustering training is optional per the README)
- The data lack cell–cell graph structure and a feature-only embedding method is preferred
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scGAE
  role: Provides the multitask-oriented graph autoencoder whose clustering_train()
    stage jointly minimizes reconstruction and KL soft-assignment losses
  repo: github.com/ZixiangLuo1161/scGAE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:268121ace8a21d7333199376977aea4e1701a1749479c5e2732034a91cdfe7e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGAE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# multi-objective-loss-optimization

## Summary

Jointly optimizes reconstruction and clustering objectives in scGAE's alter-training stage by minimizing L = L_r + gamma*L_c (gamma=2.5), using a DEC-style Student's-t soft assignment so the latent embedding and cluster assignment are learned simultaneously for a clustering-friendly representation.

## When to use

Use when a graph autoencoder latent embedding must be refined for clustering: after pre-training on reconstruction loss alone, when initial cluster centroids (e.g. from Louvain on embedded data) are available and soft-assignment KL refinement is needed.

## When NOT to use

- The model has not been pre-trained on the reconstruction loss L_r — alter-training requires initial centroids from Louvain on pre-trained embedded data
- The goal is pure reconstruction/dimensionality reduction without cluster labels; model.train() on L_r alone suffices (clustering training is optional per the README)
- The data lack cell–cell graph structure and a feature-only embedding method is preferred

## Inputs

- Pre-trained scGAE model and embedded data (latent representation of preprocessed scRNA-seq counts, e.g. scTransform-normalized)
- KNN cell-graph adjacency matrix from get_adj (default K=35)
- Initial cluster centroids from Louvain clustering of the pre-trained embedding (get_centers_louvain)

## Outputs

- Trained deep-clustering model with jointly optimized latent embedding
- Per-cell cluster assignments exported to a CSV file with one row per cell and a cluster-label column

## How to apply

After pre-training the joint graph autoencoder on the reconstruction loss L_r (with a KNN cell graph, default K=35), obtain initial cluster centroids by running Louvain clustering on the pre-trained embedded data via get_centers_louvain. Then run alter-training with model.clustering_train(): compute the Student's-t soft assignment Q of the embeddings to the centroids, derive the DEC target distribution P, and jointly minimize L = L_r + gamma*L_c with the weighting constant gamma=2.5 so that data embedding and cluster assignment are learned simultaneously. Rationale: reconstruction alone ignores cell-cluster structure, and the multitask-oriented objective preserves topological structure information and feature information simultaneously while shaping the latent space for clustering.

## Related tools

- **scGAE** (Provides the multitask-oriented graph autoencoder whose clustering_train() stage jointly minimizes reconstruction and KL soft-assignment losses) — github.com/ZixiangLuo1161/scGAE

## Examples

```
centers = get_centers_louvain(embedded_data); model.clustering_train()
```

## Evaluation signals

- Confirm initial centroids were obtained by Louvain clustering on the pre-trained embedded data before clustering_train() is invoked
- Check the joint loss was L = L_r + gamma*L_c with gamma = 2.5 (not a different weighting)
- Verify the exported CSV exists and contains a cluster-label column with a label for every cell
- Confirm the soft assignment Q used Student's-t kernel distances to centroids and the target distribution P was the DEC-derived P
- End-to-end reproducibility: execute the script following Run_scGAE.ipynb and confirm the trained model can generate embeddings via model.embedding()

## Limitations

- No random seed or stochastic-variation control is reported, so results may vary across runs
- Optimal K for KNN graph construction varies among datasets and different parts of a dataset; the default K=35 may need adjustment
- Scoring/validity of the resulting clusters depends on Louvain-initialized centroids; no version numbers are specified for the dependency stack (tensorflow 2.3.0, spektral 0.6.1, scanpy 1.8.0, etc.)

## Evidence

- [abstract] simultaneous embedding and clustering: "uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information in scRNA-seq data simultaneously"
- [discussion] initial centroids from Louvain on pre-trained embedding: "In experiments, we use the pre-trained network to generate the initial embedded data which are clustered to obtain the initial centroids by Louvain"
- [readme] clustering_train API: "After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function."
