---
name: sparse-adjacency-matrix-serialization
description: Use when a pre-processed scRNA-seq expression matrix (normalized HVGs,
  one row per cell) must be converted into a cell-cell graph for graph-based learning,
  and the adjacency needs to be persisted in sparse .npz format.
when_to_use_negative:
- Input is raw unnormalized scRNA-seq counts — the cell graph should be built on the
  normalized HVG matrix from upstream pre-processing, not raw values.
- The downstream method does not operate on a cell-cell graph (e.g. non-graph imputation
  or clustering methods), so serializing a kNN adjacency adds no value.
- Cells must not be connected by symmetric mutual-neighbor edges — an asymmetric directed
  kNN structure is required instead.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: faiss-cpu
  role: computes pairwise cosine distances between all cell expression profiles (version
    1.7.3)
  repo: https://github.com/zehaoxiong123/scGCL
- name: torch-cluster
  role: performs the kNN search to retrieve the k=15 nearest neighbors per cell (version
    1.6.0)
  repo: https://github.com/zehaoxiong123/scGCL
- name: torch-geometric
  role: edge format used to assemble the adjacency matrix A (version 2.2.0)
  repo: https://github.com/zehaoxiong123/scGCL
- name: PyTorch
  role: holds the normalized HVG expression matrix as a tensor, one row per cell (version
    1.10.0)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:853eb37b92e74b21ff364533b7813430901aa3baef8fecf4c570471bc3933b7b
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# sparse-adjacency-matrix-serialization

## Summary

Assembles the kNN cell graph (one node per cell, edges to the k=15 cosine-nearest neighbors, symmetrized) as a torch-geometric adjacency matrix and saves it as a sparse .npz file (adjacency_matrix.npz) so downstream graph contrastive learning can consume it.

## When to use

Use when a pre-processed scRNA-seq expression matrix (normalized HVGs, one row per cell) must be converted into a cell-cell graph for graph-based learning, and the adjacency needs to be persisted in sparse .npz format.

## When NOT to use

- Input is raw unnormalized scRNA-seq counts — the cell graph should be built on the normalized HVG matrix from upstream pre-processing, not raw values.
- The downstream method does not operate on a cell-cell graph (e.g. non-graph imputation or clustering methods), so serializing a kNN adjacency adds no value.
- Cells must not be connected by symmetric mutual-neighbor edges — an asymmetric directed kNN structure is required instead.

## Inputs

- normalized HVG expression matrix (output of upstream pre-processing), one row per cell, loaded as a PyTorch tensor
- k parameter (k=15 in scGCL)
- distance metric choice (cosine distance)

## Outputs

- adjacency_matrix.npz — sparse N-cells x N-cells adjacency matrix in torch-geometric edge format with symmetrized kNN edges

## How to apply

Load the normalized HVG expression matrix produced by upstream pre-processing into a PyTorch tensor with one row per cell. Compute pairwise cosine distances between all cell expression profiles using faiss-cpu 1.7.3, then retrieve the k=15 nearest neighbors for every cell via torch-cluster 1.6.0 kNN search. Assemble the adjacency matrix A in torch-geometric 2.2.0 edge format with one node per cell, symmetrizing the kNN edges so that the relation between cells is mutual. Save the sparse adjacency to adjacency_matrix.npz for downstream graph contrastive learning; k=15 is the value used in the scGCL work.

## Related tools

- **faiss-cpu** (computes pairwise cosine distances between all cell expression profiles (version 1.7.3)) — https://github.com/zehaoxiong123/scGCL
- **torch-cluster** (performs the kNN search to retrieve the k=15 nearest neighbors per cell (version 1.6.0)) — https://github.com/zehaoxiong123/scGCL
- **torch-geometric** (edge format used to assemble the adjacency matrix A (version 2.2.0)) — https://github.com/zehaoxiong123/scGCL
- **PyTorch** (holds the normalized HVG expression matrix as a tensor, one row per cell (version 1.10.0))

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- Adjacency A is a square N-cells x N-cells matrix.
- Every cell node has exactly its k=15 cosine-nearest neighbors retained (after symmetrization, degree >= 15).
- Edges are symmetrized: if cell b is a neighbor of cell a, the reverse edge is present.
- Distances used for neighbor selection are cosine distances computed with faiss-cpu.
- File adjacency_matrix.npz loads as a sparse matrix and round-trips the torch-geometric edge list.

## Limitations

- Existing scRNA-seq imputation methods that ignore topological structure cannot comprehensively utilize relationships between cells — building this graph is specific to methods that exploit cell-cell topology (the missing signal motivating scGCL).
- k=15 is the value chosen in this work; other k values would need re-validation.
- The graph is built from the pre-processed expression profile, so it inherits any biases of the upstream normalization/HVG selection.

## Evidence

- [methods] knn graph construction: [evidence span withheld — anchor status: not_found]
- [intro] toolchain versions: "torch-cluster version: 1.6.0"
- [readme] torch-geometric version: "torch-geometric version: 2.2.0"
- [abstract] motivation for cell graph: "existing scRNA-seq data imputation methods ignore the topological structure information of scRNA-seq data and cannot comprehensively utilize the relationships between cells"
