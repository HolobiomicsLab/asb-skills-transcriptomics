---
name: batched-graph-computation
description: Use when building k-NN graphs over thousands of single cells (or other
  large point sets) where full pairwise distance computation would exhaust memory,
  and downstream label propagation must run on the graph.
when_to_use_negative:
- The dataset is small enough that a dense pairwise distance matrix fits comfortably
  in memory — batching adds complexity without benefit.
- The goal is the GCN-based consensus model itself rather than the nonparametric neighbor
  majority baseline on a pre-built k-NN graph.
- The input is already a graph (e.g., a precomputed adjacency file); only the label
  propagation, not graph construction, is needed.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: pytorch-cluster
  role: Performs the batched k-NN graph construction on PCA embeddings; installed
    from the pyg conda channel
  repo: https://github.com/lewinsohndp/scSHARP
- name: PyTorch
  role: Underlying framework in which the batching and label propagation are implemented
  repo: https://github.com/lewinsohndp/scSHARP
- name: scSHARP
  role: Package providing the pipeline whose nonparametric neighbor majority baseline
    uses the batched k-NN graph
  repo: https://github.com/lewinsohndp/scSHARP
- name: conda
  role: Environment management (Python 3.9 environment with pytorch-cluster from the
    pyg channel)
  repo: https://github.com/mperozek11/scSHARP_tool
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:3e2e37af7838b31c4e404771a0a84930739978cfb53429330fbe2b5496a313ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lewinsohndp/scSHARP
  tools:
  - PyTorch
  - Python
  - conda
  - pip
  - pytorch-cluster
  - scSHARP
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# batched-graph-computation

## Summary

Bounded-memory construction and processing of large cell-cell graphs by chunking operations into batches (e.g., 1000 cells at a time) using pytorch-cluster on PCA-reduced scRNA-seq embeddings, as used in scSHARP's nonparametric neighbor majority experiments.

## When to use

Use when building k-NN graphs over thousands of single cells (or other large point sets) where full pairwise distance computation would exhaust memory, and downstream label propagation must run on the graph.

## When NOT to use

- The dataset is small enough that a dense pairwise distance matrix fits comfortably in memory — batching adds complexity without benefit.
- The goal is the GCN-based consensus model itself rather than the nonparametric neighbor majority baseline on a pre-built k-NN graph.
- The input is already a graph (e.g., a precomputed adjacency file); only the label propagation, not graph construction, is needed.

## Inputs

- scRNA-seq expression matrix with reference cell-type labels (raw counts)
- PCA embedding of cells reduced to 500 dimensions
- k neighbor values to sweep (k-NN parameter)

## Outputs

- k-NN graph over cells (edge list built with pytorch-cluster)
- Propagated cell-type labels per convergence strategy and k
- Accuracy table per dataset (e.g., neighbor_majority_accuracy.csv)

## How to apply

First reduce the scRNA-seq expression matrix to PCA embeddings (down to 500 dimensions in the scSHARP benchmarks). Build a k-NN graph on these PCA vectors using pytorch-cluster, iterating over cells in batches of 1000 so neighbor-search memory stays bounded rather than computing all pairwise distances at once. On the resulting graph, run downstream computations such as neighbor-majority label propagation, sweeping over k neighbor values and convergence strategies (single round; stop when fewer than 5% of labels change; run until all cells labeled or 50 epochs). Batching preserves exact neighbor sets for each cell while keeping the peak memory footprint roughly proportional to batch size plus the edge list, enabling the same analysis on datasets of different sizes.

## Related tools

- **pytorch-cluster** (Performs the batched k-NN graph construction on PCA embeddings; installed from the pyg conda channel) — https://github.com/lewinsohndp/scSHARP
- **PyTorch** (Underlying framework in which the batching and label propagation are implemented) — https://github.com/lewinsohndp/scSHARP
- **scSHARP** (Package providing the pipeline whose nonparametric neighbor majority baseline uses the batched k-NN graph) — https://github.com/lewinsohndp/scSHARP
- **conda** (Environment management (Python 3.9 environment with pytorch-cluster from the pyg channel)) — https://github.com/mperozek11/scSHARP_tool

## Examples

```
conda create -n scsharp python=3.9; conda activate scsharp; conda install pytorch-cluster -c pyg
```

## Evaluation signals

- Accuracy values under the selected k and convergence settings must match the neighbor-majority accuracies reported in Table 1 of the paper.
- The assembled sweep table must be comparable to the Supplementary Appendix SD sweep table across the four benchmark datasets.
- Peak memory during k-NN construction should stay bounded (batch size 1000) rather than scaling with the square of cell count.
- Every cell in the input PCA embedding must receive a label after propagation (all-cells-labeled or 50-epoch termination).
- On the testis dataset, neighbor-majority accuracy should fall within the GCN's standard deviation for both total and unconfident cell accuracy, serving as a sanity anchor.

## Limitations

- Additional PyTorch Geometric dependencies such as torch-cluster may need to be manually installed depending on the system.
- Version numbers for scSHARP and its dependencies are not specified in the methods, so exact reproducibility of the batched pipeline may vary across versions.
- R tools are necessary for running scSHARP's full pipeline, though the batched k-NN baseline itself runs in Python.
- Convergence strategy choice (single round, <5% label change, or 50-epoch cap) affects results and must be swept rather than fixed a priori.

## Evidence

- [methods] Build a k-NN graph on the PCA vectors using pytorch-cluster, processing cells in batches of 1000 to keep memory bounded.: [evidence span withheld — anchor status: not_found]
- [methods] Run neighbor-majority label propagation, iteratively updating each cell's label by majority vote over its labeled neighbors: [evidence span withheld — anchor status: not_found]
- [methods] ``conda install pytorch-cluster -c pyg``: "``conda install pytorch-cluster -c pyg``"
- [readme] Depending on your system, additional PyTorch Geometric dependences such as torch-cluster may need to be manually installed.: [evidence span withheld — anchor status: external_doc]
- [abstract] We also compare our method to a nonparametric neighbor majority approach, showing comparable results.: "We also compare our method to a nonparametric neighbor majority approach, showing comparable results."
