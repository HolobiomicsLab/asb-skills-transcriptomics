---
name: knn-graph-construction-from-pca-embeddings
description: Use when you need a cell-cell similarity graph for label propagation
  or GNN classification on scRNA-seq data, and cells are represented as PCA-reduced
  expression embeddings.
when_to_use_negative:
- Input is raw counts without a dimensionality-reduction step — the README instructs
  users to 'input raw counts' to scSHARP, which computes its own embeddings; feeding
  a pre-built unrelated graph bypasses this pipeline.
- The goal is the GCN consensus model itself rather than the nonparametric neighbor-majority
  baseline or graph construction for it.
- Single-cell data is not expression-based (e.g. image or spatial-only modalities)
  where PCA over transcriptomic profiles is not meaningful.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: pytorch-cluster
  role: Builds the k-NN graph over PCA embeddings; installed via 'conda install pytorch-cluster
    -c pyg'
  repo: https://github.com/lewinsohndp/scSHARP
- name: scSHARP
  role: End-to-end pipeline (written in PyTorch) whose label propagation consumes
    the k-NN graph; installed with pip into a Python 3.9 conda environment
  repo: https://github.com/lewinsohndp/scSHARP
- name: PyTorch
  role: Underlying tensor/GNN framework in which the graph and GCN label propagation
    are implemented
  repo: https://github.com/lewinsohndp/scSHARP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:a32a56f625ca78b3b5b5776581645aacf199613d420fab2fa3e44618c2211c5e
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
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# kNN graph construction from PCA embeddings

## Summary

Builds a k-nearest-neighbor graph over single-cell PCA embeddings using pytorch-cluster, with batched processing to keep memory bounded, as the graph substrate for neighbor-majority label propagation and GCN-based cell type annotation in scSHARP.

## When to use

Use when you need a cell-cell similarity graph for label propagation or GNN classification on scRNA-seq data, and cells are represented as PCA-reduced expression embeddings.

## When NOT to use

- Input is raw counts without a dimensionality-reduction step — the README instructs users to 'input raw counts' to scSHARP, which computes its own embeddings; feeding a pre-built unrelated graph bypasses this pipeline.
- The goal is the GCN consensus model itself rather than the nonparametric neighbor-majority baseline or graph construction for it.
- Single-cell data is not expression-based (e.g. image or spatial-only modalities) where PCA over transcriptomic profiles is not meaningful.

## Inputs

- scRNA-seq expression matrix (raw counts) with reference cell-type labels
- PCA embeddings of cells reduced to 500 dimensions
- k neighbor count (swept over multiple values)

## Outputs

- k-nearest-neighbor graph (adjacency/edge list) over cell PCA vectors
- Per-cell propagated label predictions for each k and convergence strategy
- Accuracy table against reference labels (e.g. neighbor_majority_accuracy.csv)

## How to apply

Load the scRNA-seq dataset with reference cell-type labels and compute PCA embeddings reduced to 500 dimensions. Build a k-NN graph on the PCA vectors with pytorch-cluster (installed from the pyg conda channel), processing cells in batches of 1000 so memory stays bounded on large datasets. Choose k by sweeping over neighbor values, since label-propagation accuracy varies with k; in the scSHARP experiments the same graph feeds the nonparametric neighbor-majority baseline, where each cell's label is updated by majority vote over its labeled neighbors. This graph construction step is shared infrastructure: the GCN model propagates labels from component tool predictions over the same neighborhood structure.

## Related tools

- **pytorch-cluster** (Builds the k-NN graph over PCA embeddings; installed via 'conda install pytorch-cluster -c pyg') — https://github.com/lewinsohndp/scSHARP
- **scSHARP** (End-to-end pipeline (written in PyTorch) whose label propagation consumes the k-NN graph; installed with pip into a Python 3.9 conda environment) — https://github.com/lewinsohndp/scSHARP
- **PyTorch** (Underlying tensor/GNN framework in which the graph and GCN label propagation are implemented) — https://github.com/lewinsohndp/scSHARP

## Examples

```
conda install pytorch-cluster -c pyg
```

## Evaluation signals

- Accuracy under the selected k and convergence settings must match the neighbor-majority accuracies reported in Table 1 of the paper.
- Sweep results must be comparable to the Supplementary Appendix SD sweep table.
- On the testis dataset, neighbor-majority accuracy must fall within the GCN's standard deviation for both total and unconfident cell accuracy.
- Batched k-NN construction (batch size 1000) must complete without out-of-memory errors on the four benchmark datasets.
- Output accuracies assemble into a well-formed table written to neighbor_majority_accuracy.csv.

## Limitations

- Additional PyTorch Geometric dependencies such as torch-cluster may need to be manually installed depending on the system.
- Software version numbers for scSHARP and dependencies are not specified in the methods.
- R tools (R4scSHARP) are a necessary dependency for running the full scSHARP pipeline, adding an environment constraint.

## Evidence

- [methods] Build a k-NN graph on the PCA vectors using pytorch-cluster, processing cells in batches of 1000 to keep memory bounded.: [evidence span withheld — anchor status: not_found]
- [methods] ``conda install pytorch-cluster -c pyg``: "``conda install pytorch-cluster -c pyg``"
- [readme] Depending on your system, additional PyTorch Geometric dependences such as torch-cluster may need to be manually installed.: [evidence span withheld — anchor status: external_doc]
- [readme] knn_experiment.ipynb - Results of non-parametric neighbor majority approach for simulated and testis data sets.: [evidence span withheld — anchor status: external_doc]
