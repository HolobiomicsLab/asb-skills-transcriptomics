---
name: synthetic-scrnaseq-graph-generation
description: Use when implementing or validating a graph-based scRNA-seq model (e.g.
  scGAE) and you need a small synthetic count matrix plus a KNN cell graph to run
  dimension-checked forward-pass tests before touching real datasets.
when_to_use_negative:
- You already have a real preprocessed scRNA-seq dataset (e.g. GSE84133 pancreatic
  cells or HRA000084 HSPCs) and an empirical adjacency matrix — use it directly instead
  of synthetic data
- Your model consumes only raw count matrices with no graph/adjacency input — the
  synthetic KNN graph adds nothing
- You need biologically valid trajectory or cluster structure for benchmarking against
  t-SNE/DCA/scVI-class methods; synthetic random data cannot support the paper's performance
  comparisons
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGAE
  role: Target model whose expected (X, adjacency) input contract the synthetic data
    must satisfy; get_adj builds the adjacency matrix
  repo: github.com/ZixiangLuo1161/scGAE
- name: Spektral
  role: Python package used to implement the graph attention layers that consume the
    synthetic cell graph
- name: scanpy
  role: Python package (1.8.0) usable for single-cell preprocessing compatible with
    scGAE inputs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:09dc85519dc26925178ff07e5520d2cbd4a0f284a1f07d75c2421bcfff00353e
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
yes_rate: 0.6
n_claims: 10
---

# synthetic-scrnaseq-graph-generation

## Summary

Generate a synthetic scRNA-seq count matrix (e.g. 100 cells × 1000 genes) together with a KNN cell-graph adjacency matrix (default K=35 per the scGAE paper) so that graph-based single-cell models like scGAE can be smoke-tested end-to-end without real data.

## When to use

Use when implementing or validating a graph-based scRNA-seq model (e.g. scGAE) and you need a small synthetic count matrix plus a KNN cell graph to run dimension-checked forward-pass tests before touching real datasets.

## When NOT to use

- You already have a real preprocessed scRNA-seq dataset (e.g. GSE84133 pancreatic cells or HRA000084 HSPCs) and an empirical adjacency matrix — use it directly instead of synthetic data
- Your model consumes only raw count matrices with no graph/adjacency input — the synthetic KNN graph adds nothing
- You need biologically valid trajectory or cluster structure for benchmarking against t-SNE/DCA/scVI-class methods; synthetic random data cannot support the paper's performance comparisons

## Inputs

- Synthetic scRNA-seq count matrix X (cells × genes, e.g. 100 × 1000 numpy array)
- KNN parameter K for cell-graph construction (default K=35 per scGAE)

## Outputs

- KNN cell-graph adjacency matrix A (cells × cells, e.g. 100 × 100)
- Paired (X, A) input tensors ready for a graph autoencoder forward-pass smoke test

## How to apply

Generate a synthetic count matrix X with shape cells × genes (e.g. 100 × 1000) that mimics scRNA-seq counts, since scGAE expects preprocessed count data as input. Build the cell graph with a KNN algorithm over cells — the scGAE paper sets default K=35, noting 'the optimal K varies among different datasets and different parts of a dataset' — and export the adjacency matrix A (cells × cells) via the repository's get_adj function. Pair X and A as the dual inputs to the multitask-oriented graph autoencoder, whose feature decoder reconstructs X and whose graph decoder reconstructs A via sigma(ZZ^T). Run a forward-pass smoke test asserting the latent embedding, reconstructed-count (100 × 1000) and reconstructed-adjacency (100 × 100) tensors have correct dimensions and finite values. The scGAE reference stack is TensorFlow 2.3.0 with Spektral 0.6.1 for the graph attention layers, so synthetic tensors should be numpy arrays compatible with Spektral's Graph/adjacency conventions.

## Related tools

- **scGAE** (Target model whose expected (X, adjacency) input contract the synthetic data must satisfy; get_adj builds the adjacency matrix) — github.com/ZixiangLuo1161/scGAE
- **Spektral** (Python package used to implement the graph attention layers that consume the synthetic cell graph)
- **scanpy** (Python package (1.8.0) usable for single-cell preprocessing compatible with scGAE inputs)

## Examples

```
import numpy as np; X = np.random.rand(100, 1000); from scgae_utils import get_adj; A = get_adj(X, k=35); model = SCGAE(X, A); model.train()
```

## Evaluation signals

- Latent embedding, reconstructed count matrix (100 × 1000), and reconstructed adjacency (100 × 100) all have the expected shapes
- All forward-pass tensors are finite (no NaN/Inf) on the synthetic (X, A) input pair
- Adjacency matrix A is square (cells × cells) and consistent with the chosen K=35 neighborhood size
- Joint loss L = L_X + 0.6·L_A computes to a finite scalar on the synthetic inputs

## Limitations

- The optimal K for KNN graph construction varies among datasets and even across regions of one dataset, so synthetic graphs built with K=35 may not reflect real cell-graph topology
- scGAE cannot directly embed complex data structure into two dimensions; it embeds to an intermediate dimension before t-SNE, so synthetic 2D validation is not meaningful
- No random seed or stochastic-variation control is reported in the paper, so synthetic-data results may vary between runs

## Evidence

- [abstract] scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information: "scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information"
- [discussion] The default K is predened as 35 in this study: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [readme] Compute the adjacency matrix using get_adj function and build the SCGAE model.: "Compute the adjacency matrix using get_adj function and build the SCGAE model."
- [discussion] We used the Python package Spektral32 to implement our model.: "We used the Python package Spektral32 to implement our model."
