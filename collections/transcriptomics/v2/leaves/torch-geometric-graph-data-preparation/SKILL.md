---
name: torch-geometric-graph-data-preparation
description: Use when a scRNA-seq imputation or representation-learning pipeline needs
  a cell-cell graph (nodes = cells, edges = k nearest neighbors) built from a normalized,
  HVG-filtered expression matrix before graph contrastive learning.
when_to_use_negative:
- The input is already a graph/adjacency structure rather than a tabular expression
  matrix — no kNN construction is needed
- The expression matrix has not been normalized and HVG-filtered yet; building the
  kNN graph on raw counts distorts cosine neighborhoods
- The analysis does not need cell-cell topological structure (e.g. purely gene-level
  statistics with no graph-based model downstream)
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: faiss-cpu
  role: Computes pairwise cosine distances between all cell expression profiles (version
    1.7.3)
  repo: https://github.com/zehaoxiong123/scGCL
- name: torch-cluster
  role: Performs the kNN search to retrieve the k=15 nearest neighbors for every cell
    (version 1.6.0)
  repo: https://github.com/zehaoxiong123/scGCL
- name: torch-geometric
  role: Assembles and stores the adjacency in edge_index (graph) format for downstream
    graph contrastive learning (version 2.2.0)
  repo: https://github.com/zehaoxiong123/scGCL
- name: PyTorch
  role: Holds the normalized HVG expression matrix as a tensor, one row per cell (version
    1.10.0)
  repo: https://github.com/zehaoxiong123/scGCL
- name: scGCL
  role: Downstream graph contrastive learning imputation method that consumes the
    constructed cell graph
  repo: https://github.com/zehaoxiong123/scGCL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:7dd93d58e297f27fd6463fe0f1146f96aa9a869d66bd70d411d1a4e1587c1ceb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - PyTorch
  - Python
  - faiss-cpu
  - scGCL
  - torch-cluster
  - torch-geometric
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# torch-geometric-graph-data-preparation

## Summary

Constructs a kNN cell graph over scRNA-seq cells from the pre-processed expression matrix, using cosine distance and k=15 neighbors, as done in scGCL before graph contrastive learning. This preserves cell-cell topological relationships that imputation methods ignoring structure cannot use.

## When to use

Use when a scRNA-seq imputation or representation-learning pipeline needs a cell-cell graph (nodes = cells, edges = k nearest neighbors) built from a normalized, HVG-filtered expression matrix before graph contrastive learning.

## When NOT to use

- The input is already a graph/adjacency structure rather than a tabular expression matrix — no kNN construction is needed
- The expression matrix has not been normalized and HVG-filtered yet; building the kNN graph on raw counts distorts cosine neighborhoods
- The analysis does not need cell-cell topological structure (e.g. purely gene-level statistics with no graph-based model downstream)

## Inputs

- Normalized HVG scRNA-seq expression matrix (one row per cell), e.g. loaded from the .h5 dataset file used by scGCL that contains expression values and true labels
- kNN parameter k (set to 15 in scGCL) and distance metric choice (cosine)

## Outputs

- Sparse cell-cell adjacency matrix (N_cells x N_cells) saved as adjacency_matrix.npz in torch-geometric edge_index format, ready for graph contrastive learning

## How to apply

Load the normalized HVG expression matrix (output of upstream pre-processing, e.g. the .h5 file scGCL consumes containing expression values and true labels) into a PyTorch tensor with one row per cell. Compute pairwise cosine distances between all cell expression profiles using faiss-cpu 1.7.3. Retrieve the k=15 nearest neighbors for every cell with torch-cluster 1.6.0's kNN search — k=15 is the value used in the scGCL work. Assemble the adjacency matrix A in torch-geometric 2.2.0 edge_index format with one node per cell, symmetrizing the directed kNN edges so neighborhood information flows both ways, then save the sparse adjacency to adjacency_matrix.npz for downstream graph contrastive learning.

## Related tools

- **faiss-cpu** (Computes pairwise cosine distances between all cell expression profiles (version 1.7.3)) — https://github.com/zehaoxiong123/scGCL
- **torch-cluster** (Performs the kNN search to retrieve the k=15 nearest neighbors for every cell (version 1.6.0)) — https://github.com/zehaoxiong123/scGCL
- **torch-geometric** (Assembles and stores the adjacency in edge_index (graph) format for downstream graph contrastive learning (version 2.2.0)) — https://github.com/zehaoxiong123/scGCL
- **PyTorch** (Holds the normalized HVG expression matrix as a tensor, one row per cell (version 1.10.0)) — https://github.com/zehaoxiong123/scGCL
- **scGCL** (Downstream graph contrastive learning imputation method that consumes the constructed cell graph) — https://github.com/zehaoxiong123/scGCL

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- The saved adjacency_matrix.npz contains a square N_cells x N_cells sparse matrix whose node count equals the number of rows in the input expression tensor
- Every cell node has exactly its k=15 cosine-nearest neighbors retained in the directed kNN structure before symmetrization
- Distances used for neighbor selection are cosine distances computed with faiss-cpu, not Euclidean or correlation metrics
- Edges are symmetrized: if (a,b) is in edge_index, (b,a) appears after symmetrization, so no cell is isolated in the graph
- The adjacency loads cleanly in torch-geometric 2.2.0 edge format and is compatible with the scGCL graph contrastive learning pipeline

## Limitations

- After symmetrizing the directed kNN edges, individual node degrees exceed 15, so 'every node has exactly k=15 neighbors' only holds for the directed pre-symmetrized graph
- k=15 is the specific value used in this work; other values are untested here and may require validation
- Neighborhood quality depends entirely on upstream pre-processing (normalization and HVG selection) of the scRNA-seq profile

## Evidence

- [other] The cell graph is built with one node per cell; an edge is assigned between cell a and cell b if cell b is one of the k nearest nodes to cell a, with k set to 15 in this work.: [evidence span withheld — anchor status: not_found]
- [abstract] existing scRNA-seq data imputation methods ignore the topological structure information of scRNA-seq data and cannot comprehensively utilize the relationships between cells: "existing scRNA-seq data imputation methods ignore the topological structure information of scRNA-seq data and cannot comprehensively utilize the relationships between cells"
- [readme] A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information.: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [intro] torch-cluster version: 1.6.0: "torch-cluster version: 1.6.0"
