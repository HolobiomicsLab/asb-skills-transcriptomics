---
name: louvain-centroid-initialization
description: Use when training a graph autoencoder for scRNA-seq cell clustering (e.g.
  scGAE) and initial cluster centers are needed before soft-assignment/KL clustering
  training; also relevant to any DEC-style pipeline requiring centroid initialization.
when_to_use_negative:
- Input data has not been pre-trained/embedded yet — Louvain should be run on the
  embedded data, not the raw high-dimensional count matrix.
- The analysis goal is pure visualization/trajectory inference without clustering;
  the clustering-training stage (and its centroid initialization) is optional in scGAE.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scGAE
  role: Graph autoencoder whose get_centers_louvain and model.clustering_train() functions
    implement the Louvain centroid initialization and subsequent KL-loss clustering
    training
  repo: github.com/ZixiangLuo1161/scGAE
- name: Louvain
  role: Community-detection algorithm applied to the pre-trained embedded data to
    obtain the initial cluster centroids
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:80a8cc917d668d82e88740d21ea455955b347dcd966c4c76caae8e5a08349775
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Louvain
  - scGAE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# louvain-centroid-initialization

## Summary

Obtains initial cluster centroids for scGAE's deep-clustering stage by running Louvain clustering on the pre-trained autoencoder's embedded data, which are then used to initialize the DEC-style KL-loss clustering training.

## When to use

Use when training a graph autoencoder for scRNA-seq cell clustering (e.g. scGAE) and initial cluster centers are needed before soft-assignment/KL clustering training; also relevant to any DEC-style pipeline requiring centroid initialization.

## When NOT to use

- Input data has not been pre-trained/embedded yet — Louvain should be run on the embedded data, not the raw high-dimensional count matrix.
- The analysis goal is pure visualization/trajectory inference without clustering; the clustering-training stage (and its centroid initialization) is optional in scGAE.

## Inputs

- Pre-trained SCGAE model and its latent embedded representation of scRNA-seq cells (from model.train() / model.embedding())
- KNN cell graph (default K=35) adjacency matrix from get_adj on preprocessed count data

## Outputs

- Initial cluster centroids (from get_centers_louvain) for the clustering training stage
- Trained deep-clustering model after model.clustering_train() with per-cell cluster assignments

## How to apply

First pre-train the SCGAE model on the reconstruction loss L_r over the KNN cell graph (default K=35) to obtain an initial embedded representation of the cells. Then run Louvain community detection on this pre-trained embedded data via the get_centers_louvain function to derive the initial cluster centroids — the article reports that centroids are obtained by 'Louvain37' clustering of the initial embedded data. Feed these centroids into model.clustering_train(), where Student's-t soft assignments Q of embeddings to centroids are computed and a DEC target distribution P is derived for joint minimization of L = L_r + gamma*L_c with gamma=2.5. The rationale is that deep clustering learns the embedding and cluster assignment simultaneously, yielding a clustering-friendly latent representation; Louvain on the pre-trained embedding supplies a data-driven starting point so the KL loss converges from a sensible partition rather than random centroids.

## Related tools

- **scGAE** (Graph autoencoder whose get_centers_louvain and model.clustering_train() functions implement the Louvain centroid initialization and subsequent KL-loss clustering training) — github.com/ZixiangLuo1161/scGAE
- **Louvain** (Community-detection algorithm applied to the pre-trained embedded data to obtain the initial cluster centroids)

## Examples

```
centers = get_centers_louvain(model.embedding()); model.clustering_train()
```

## Evaluation signals

- get_centers_louvain returns centroids for a non-trivial number of communities in the pre-trained embedding (not a single cluster covering all cells).
- model.clustering_train() runs end-to-end and the joint loss L = L_r + 2.5*L_c decreases over training epochs.
- The exported CSV contains one row per cell with a populated cluster-label column (no NaN labels).
- Per-cell cluster assignments correspond to the soft-assignment matrix Q/argmax over the initialized centroids.
- Downstream clustering metrics (e.g. ARI/NMI vs. known labels) should be no worse than, and ideally better than, those from a randomly initialized clustering run.

## Limitations

- No random seed or stochastic-variation control is reported, so Louvain initialization and clustering results may vary between runs.
- The optimal K for the KNN cell graph varies among different datasets and dataset subsets, affecting the embedding on which Louvain is run.
- No software/package versions are pinned beyond the README requirements list, which may affect reproducibility of the clustering stage.

## Evidence

- [readme] readme: "After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function."
- [discussion] discussion: "In experiments, we use the pre-trained network to generate the initial embedded data which are clustered to obtain the initial centroids by Louvain37."
- [results] task_finding: [evidence span withheld — anchor status: not_found]
- [abstract] abstract: "scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information in scRNA-seq data simultaneously"
