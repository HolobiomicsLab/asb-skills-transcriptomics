---
name: gene-expression-decoder-implementation
description: 'Use when re-implementing or extending the scGAE decoder stack: you have
  a preprocessed scRNA-seq count matrix plus a KNN cell-graph adjacency matrix and
  need a single autoencoder that reconstructs both X and A from a shared latent embedding.'
when_to_use_negative:
- 'You only need a 2D plot: the article notes the graph autoencoder cannot directly
  embed into two-dimensional space, so a t-SNE step on an intermediate embedding is
  required rather than this decoder stack.'
- 'The data is raw/unprocessed counts: scGAE takes preprocessed data as input, so
  run scTransform (Seurat) preprocessing first.'
- You need only feature-based embedding with no topology preservation — a plain autoencoder
  or DCA/scVI would avoid the graph construction and edge-decoder overhead.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGAE
  role: The multitask graph autoencoder whose feature and graph decoders are implemented;
    source files scgae.py, layers.py, losses.py, utils.py
  repo: github.com/ZixiangLuo1161/scGAE
- name: Spektral
  role: Python package used to implement the graph attention (GAT) encoder layers
    feeding the decoders
- name: Seurat (scTransform)
  role: Used to preprocess the count matrix before graph construction and model training
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:cde165324a2c1ac6edf92cc49bd2027b92f6a610cad14a4b1ed5dae6cf9192e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat (scTransform)
  - Spektral
  - scGAE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# gene-expression-decoder-implementation

## Summary

Implements the scGAE multitask graph autoencoder's decoding side: a fully connected feature decoder that reconstructs the scRNA-seq count matrix and a graph (edge) decoder that reconstructs the cell–cell adjacency matrix from the latent embedding, jointly trained so one model preserves both feature and topological information.

## When to use

Use when re-implementing or extending the scGAE decoder stack: you have a preprocessed scRNA-seq count matrix plus a KNN cell-graph adjacency matrix and need a single autoencoder that reconstructs both X and A from a shared latent embedding.

## When NOT to use

- You only need a 2D plot: the article notes the graph autoencoder cannot directly embed into two-dimensional space, so a t-SNE step on an intermediate embedding is required rather than this decoder stack.
- The data is raw/unprocessed counts: scGAE takes preprocessed data as input, so run scTransform (Seurat) preprocessing first.
- You need only feature-based embedding with no topology preservation — a plain autoencoder or DCA/scVI would avoid the graph construction and edge-decoder overhead.

## Inputs

- Preprocessed scRNA-seq count matrix X (cells x genes; preprocessed with scTransform in the Seurat package)
- Cell–cell KNN adjacency matrix A computed with get_adj (default K=35)

## Outputs

- Latent embedding Z of the scRNA-seq data (via model.embedding())
- Reconstructed count matrix from the feature decoder (same dimensions as X)
- Reconstructed adjacency matrix A_r = sigma(ZZ^T) from the graph decoder
- Trained scGAE model (scgae.py, layers.py, losses.py, utils.py modules)

## How to apply

First build a KNN cell graph from the scTransform-preprocessed count matrix using the get_adj function (default K=35, adjustable per dataset). Implement the graph attention encoder E(X, A) with Spektral (v0.6.1, TensorFlow 2.3.0 backend) to produce the latent embedding Z. Then implement the feature decoder as a 4-layer fully connected network (64, 256, 512 hidden nodes) mapping Z back to the reconstructed count matrix, and the edge decoder as A_r = sigma(ZZ^T) reconstructing the adjacency. Train with the joint reconstruction loss L = L_X + lambda*L_A using lambda = 0.6, so the feature decoder preserves feature information while the graph decoder preserves topological structure. For 2D visualization, embed into an intermediate dimension and apply t-SNE rather than forcing a direct 2D graph-autoencoder output.

## Related tools

- **scGAE** (The multitask graph autoencoder whose feature and graph decoders are implemented; source files scgae.py, layers.py, losses.py, utils.py) — github.com/ZixiangLuo1161/scGAE
- **Spektral** (Python package used to implement the graph attention (GAT) encoder layers feeding the decoders)
- **Seurat (scTransform)** (Used to preprocess the count matrix before graph construction and model training)

## Examples

```
from preprocessing import get_adj; from scgae import SCGAE; A = get_adj(X, k=35); model = SCGAE(X, A); model.train(); Z = model.embedding()
```

## Evaluation signals

- Forward-pass smoke test: latent embedding, reconstructed count matrix (e.g. 100 x 1000 for a 100-cell x 1000-gene input) and reconstructed adjacency (100 x 100) all have correct dimensions.
- All reconstructed tensors contain finite values (no NaN/Inf) after the joint forward pass.
- Joint loss L = L_X + 0.6*L_A decreases during model.train() and the reconstructed adjacency resembles the KNN graph built with K=35.
- The final embedding (model.embedding()) preserves cluster structure and inter-cluster distances in downstream visualization/clustering, per the paper's evaluation on simulated and empirical data.

## Limitations

- The optimal K for KNN graph construction varies among datasets and even across parts of a dataset, so K=35 is a default, not a universal setting.
- The graph autoencoder cannot directly produce a 2D embedding; an intermediate dimension plus t-SNE is needed, so a better visualization method is still open.
- No software/package versions, random seed, or stochastic-variation control are specified beyond the README requirements (e.g. spektral 0.6.1, tensorflow 2.3.0), so results may vary across environments.

## Evidence

- [abstract] multitask-oriented graph autoencoder preserves both feature and topology: "uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information in scRNA-seq data simultaneously"
- [discussion] Spektral used to implement the model: "We used the Python package Spektral32 to implement our model."
- [discussion] KNN default K=35: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [discussion] scTransform preprocessing in Seurat: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
- [readme] Model building with get_adj: "Compute the adjacency matrix using get_adj function and build the SCGAE model."
- [readme] Model source files to download: "Download the model files (preprocessing.py, scgae.py, layers.py, losses.py, and utils.py) in this repository and import functions in them."
