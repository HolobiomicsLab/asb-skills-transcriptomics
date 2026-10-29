---
name: knn-cell-graph-construction
description: Use when a normalized scRNA-seq expression matrix (one row per cell)
  must be converted into a cell graph before graph-based contrastive learning or clustering,
  e.g. for scGCL on an h5 dataset such as 'adam'.
when_to_use_negative:
- The expression matrix is raw/unnormalized — cosine distances on unnormalized sparse
  counts will not reflect the pre-processed cell relationships the pipeline expects.
- You only need gene-level imputation or expression reconstruction without a cell
  graph — scGCL's graph contrastive learning stage is unnecessary overhead here.
- The downstream method is not graph-based (e.g. non-topological imputation methods
  the paper notes 'ignore the topological structure information of scRNA-seq data').
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: faiss-cpu
  role: Computes pairwise cosine distances between all cell expression profiles (version
    1.7.3)
  repo: https://github.com/zehaoxiong123/scGCL
- name: torch-cluster
  role: Performs the kNN search (k=15) to retrieve each cell's nearest neighbors (version
    1.6.0)
  repo: https://github.com/zehaoxiong123/scGCL
- name: torch-geometric
  role: Holds the symmetrized kNN edges as the adjacency matrix in edge format consumed
    by graph contrastive learning (version 2.2.0)
  repo: https://github.com/zehaoxiong123/scGCL
- name: PyTorch
  role: Tensor backend for the expression matrix and graph data (version 1.10.0)
  repo: https://github.com/zehaoxiong123/scGCL
- name: scGCL
  role: Downstream consumer of the cell graph for graph contrastive learning and clustering
  repo: https://github.com/zehaoxiong123/scGCL
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:0ba54027b3f9286de215c522738bc45ecf01769313176140bfabbfc546a8b504
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# knn-cell-graph-construction

## Summary

Constructs a cell-cell graph from a normalized scRNA-seq expression matrix by connecting each cell to its k=15 nearest neighbors under cosine distance, providing the topological input for scGCL's graph contrastive learning.

## When to use

Use when a normalized scRNA-seq expression matrix (one row per cell) must be converted into a cell graph before graph-based contrastive learning or clustering, e.g. for scGCL on an h5 dataset such as 'adam'.

## When NOT to use

- The expression matrix is raw/unnormalized — cosine distances on unnormalized sparse counts will not reflect the pre-processed cell relationships the pipeline expects.
- You only need gene-level imputation or expression reconstruction without a cell graph — scGCL's graph contrastive learning stage is unnecessary overhead here.
- The downstream method is not graph-based (e.g. non-topological imputation methods the paper notes 'ignore the topological structure information of scRNA-seq data').

## Inputs

- Normalized HVG scRNA-seq expression matrix (one row per cell; h5 file containing expression values, true labels, and other information, e.g. the adam dataset)
- k (number of neighbors; 15 in this work)
- Distance measure (cosine; alternatives city-block, euclidean, L2, manhattan)

## Outputs

- Sparse adjacency matrix A (N-cells x N-cells) in torch-geometric edge format, saved as adjacency_matrix.npz
- (Optional, for hyperparameter sweeps) ARI/NMI sweep table over k-distance combinations

## How to apply

Load the normalized HVG expression matrix into a PyTorch tensor with one row per cell, then compute pairwise cosine distances between cell expression profiles using faiss-cpu 1.7.3 — cosine is the distance measure that best captures cell-cell relationships on sparse scRNA-seq data. Retrieve the k=15 nearest neighbors per cell via torch-cluster 1.6.0 kNN search; k=15 was chosen from a sweep over k in {5,10,15,20,25} and distance measures in {city-block, cosine, euclidean, L2, manhattan} where both scGCL and scTAG peaked at k=15. Assemble the adjacency matrix A in torch-geometric 2.2.0 edge format with one node per cell and symmetrize the directed kNN edges, then save the sparse adjacency to adjacency_matrix.npz for downstream graph contrastive learning. If re-deriving the choice rather than applying the published setting, rerun the sweep via 'python main.py --dataset adam' with early stopping and score each k-distance combination by ARI/NMI against true labels, averaged over repeated runs.

## Related tools

- **faiss-cpu** (Computes pairwise cosine distances between all cell expression profiles (version 1.7.3)) — https://github.com/zehaoxiong123/scGCL
- **torch-cluster** (Performs the kNN search (k=15) to retrieve each cell's nearest neighbors (version 1.6.0)) — https://github.com/zehaoxiong123/scGCL
- **torch-geometric** (Holds the symmetrized kNN edges as the adjacency matrix in edge format consumed by graph contrastive learning (version 2.2.0)) — https://github.com/zehaoxiong123/scGCL
- **PyTorch** (Tensor backend for the expression matrix and graph data (version 1.10.0)) — https://github.com/zehaoxiong123/scGCL
- **scGCL** (Downstream consumer of the cell graph for graph contrastive learning and clustering) — https://github.com/zehaoxiong123/scGCL

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- Adjacency matrix A is square N-cells x N-cells.
- Every cell node retains exactly its k=15 cosine-nearest neighbors after symmetrization.
- Adjacency is stored as a sparse matrix in adjacency_matrix.npz in torch-geometric edge format.
- When sweeping k in {5,10,15,20,25}, average ARI/NMI against true labels peaks at k=15 with the cosine distance.
- scGCL clustering on the resulting graph consistently outperforms scTAG across all tested k-distance settings.

## Limitations

- k=15 and cosine distance are dataset-level optima found on the adam Alzheimer's disease dataset via ARI/NMI sweep; other datasets may prefer different values.
- Building the graph requires a normalized HVG expression matrix as input; it is the output of the upstream pre-processing filter, not of raw data.
- The kNN graph is symmetrized from directed nearest-neighbor relations, so the final graph is not a pure 15-neighbor-per-node structure before validation.

## Evidence

- [methods] Cell graph construction with k=15 cosine neighbors: [evidence span withheld — anchor status: not_found]
- [readme] README argument documentation for the h5 dataset input: "dataset | A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [results] Hyperparameter sweep finding on distance measure and k: [evidence span withheld — anchor status: not_found]
- [abstract] Pipeline motivation: topological structure ignored by prior methods: "existing scRNA-seq data imputation methods ignore the topological structure information of scRNA-seq data and cannot comprehensively utilize the relationships between cells"
