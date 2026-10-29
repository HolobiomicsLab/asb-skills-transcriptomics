---
name: joint-reconstruction-loss-implementation
description: 'Use when reproducing or extending scGAE: a graph autoencoder for scRNA-seq
  must simultaneously reconstruct the expression count matrix and the KNN cell-graph
  adjacency matrix in a single multitask loss.'
when_to_use_negative:
- The model must output a direct 2D embedding — scGAE embeds to an intermediate dimension
  and relies on tSNE for 2D visualization because 'the complex data structure is hard
  to be directly embedded into two-dimensional space by graph autoencoder'
- Only feature reconstruction is needed (e.g. plain denoising autoencoders like DCA)
  — the joint loss adds the topology term L_A, which is only meaningful when a cell-graph
  adjacency matrix is supplied
- No cell–cell graph can be constructed (e.g. spatially unstructured data without
  a meaningful neighborhood definition) — the adjacency-based decoder has no input
  to reconstruct
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGAE
  role: The multitask-oriented graph autoencoder whose dual feature decoder and graph
    decoder define the joint loss; implemented in losses.py, scgae.py, layers.py
  repo: github.com/ZixiangLuo1161/scGAE
- name: Spektral
  role: Python package used to implement the graph attention layers and graph convolution
    ops behind the encoder/decoders
- name: scanpy
  role: Python package for preprocessing the scRNA-seq input data before model building
    (per README quick start)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:597fa6a01494e8c0a3217b3b27d9ace9d2a9fbe2e3f96b37e9711d736ca81d32
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Spektral
  - scGAE
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# joint-reconstruction-loss-implementation

## Summary

Implement scGAE's joint reconstruction objective L = L_X + lambda·L_A (lambda = 0.6), which combines a feature decoder that reconstructs the count matrix with a graph decoder that reconstructs the cell–cell adjacency from the latent embedding Z, so the model preserves both feature and topological structure information in scRNA-seq data.

## When to use

Use when reproducing or extending scGAE: a graph autoencoder for scRNA-seq must simultaneously reconstruct the expression count matrix and the KNN cell-graph adjacency matrix in a single multitask loss.

## When NOT to use

- The model must output a direct 2D embedding — scGAE embeds to an intermediate dimension and relies on tSNE for 2D visualization because 'the complex data structure is hard to be directly embedded into two-dimensional space by graph autoencoder'
- Only feature reconstruction is needed (e.g. plain denoising autoencoders like DCA) — the joint loss adds the topology term L_A, which is only meaningful when a cell-graph adjacency matrix is supplied
- No cell–cell graph can be constructed (e.g. spatially unstructured data without a meaningful neighborhood definition) — the adjacency-based decoder has no input to reconstruct

## Inputs

- Preprocessed scRNA-seq count matrix X (cells × genes), e.g. scTransform-normalized via Seurat or scanpy (anndata/.h5ad)
- KNN cell-graph adjacency matrix A computed from the count matrix (get_adj, default K=35)
- Latent embedding Z produced by the GAT encoder

## Outputs

- Reconstructed count matrix from the feature decoder D_X (cells × genes)
- Reconstructed adjacency matrix A_r = sigma(ZZ^T) from the edge decoder (cells × cells)
- Joint loss value L = L_X + 0.6·L_A per training step

## How to apply

After the GAT encoder E(X, A) produces the latent embedding Z, attach two decoders: a 4-layer fully connected feature decoder D_X (64, 256, and 512 hidden nodes) mapping Z back to the reconstructed count matrix, and an edge decoder A_r = sigma(ZZ^T) that reconstructs the cell–cell adjacency from the inner product of the embedding. Compute the joint reconstruction loss L = L_X + lambda·L_A with lambda = 0.6, combining the feature reconstruction term (losses.py) and the topology reconstruction term. Build the cell graph adjacency with get_adj (KNN, default K=35 per the paper) and implement the layers in Spektral 0.6.1 / TensorFlow 2.3.0 as specified in the repository. Validate with a forward-pass smoke test asserting the latent, reconstructed-count (cells × genes) and reconstructed-adjacency (cells × cells) tensors have correct dimensions and finite values.

## Related tools

- **scGAE** (The multitask-oriented graph autoencoder whose dual feature decoder and graph decoder define the joint loss; implemented in losses.py, scgae.py, layers.py) — github.com/ZixiangLuo1161/scGAE
- **Spektral** (Python package used to implement the graph attention layers and graph convolution ops behind the encoder/decoders)
- **scanpy** (Python package for preprocessing the scRNA-seq input data before model building (per README quick start))

## Examples

```
from scgae import SCGAE; model = SCGAE(); adj = get_adj(X, k=35); model.train([X, adj]); Z = model.embedding()
```

## Evaluation signals

- Forward-pass smoke test: latent embedding, reconstructed counts (100 × 1000 on the synthetic test input) and reconstructed adjacency (100 × 100) all have correct dimensions and finite values
- Feature decoder architecture check: 4 fully connected layers with 64, 256, and 512 hidden nodes mapping Z back to the gene dimension
- Adjacency reconstruction shape check: A_r = sigma(ZZ^T) is cells × cells and symmetric
- Loss configuration check: total loss equals L_X + 0.6·L_A with both terms strictly decreasing during training
- Environment pinning check: spektral==0.6.1 and tensorflow==2.3.0 per the README requirements

## Limitations

- Optimal K for KNN graph construction varies among datasets and different parts of a dataset, changing the adjacency that L_A reconstructs
- No software/package versions are specified in the paper itself (only the README pins them), and no random seed or stochastic-variation control is reported, so loss values may not be exactly reproducible
- The joint loss trains an intermediate-dimension embedding; direct 2D embedding by the graph autoencoder is not supported and tSNE is needed for visualization

## Evidence

- [methods] dual decoders: [evidence span withheld — anchor status: not_found]
- [methods] joint loss weight: [evidence span withheld — anchor status: not_found]
- [discussion] KNN graph K: [evidence span withheld — anchor status: not_found]
- [discussion] spektral backend: "We used the Python package Spektral32 to implement our model."
- [readme] multitask objective: "scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information in scRNA-seq data simultaneously"
