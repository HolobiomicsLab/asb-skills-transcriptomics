---
name: high-dimensional-expression-matrix-neighborhood-search
description: Use when a pre-processed, normalized scRNA-seq HVG expression matrix
  must be converted into a cell-to-cell graph (one node per cell, kNN edges) as input
  to graph-based contrastive learning or downstream imputation.
when_to_use_negative:
- The expression matrix has not yet been normalized and restricted to highly variable
  genes — build it from the pre-processing filter output first
- The downstream method does not consume graph/topological structure over cells, so
  a kNN graph adds no benefit
- Only global cell-level statistics are needed (e.g. distribution fitting without
  graph contrastive learning); a full pairwise kNN graph is unnecessary overhead
tools:
- name: faiss-cpu
  role: computes pairwise cosine distances between all cell expression profiles
  repo: https://github.com/zehaoxiong123/scGCL
- name: torch-cluster
  role: kNN search to retrieve the k=15 nearest neighbors for every cell
  repo: https://github.com/zehaoxiong123/scGCL
- name: torch-geometric
  role: assembles the adjacency matrix A in edge format with one node per cell
  repo: https://github.com/zehaoxiong123/scGCL
- name: PyTorch
  role: tensor representation of the normalized HVG expression matrix (one row per
    cell)
  repo: https://github.com/zehaoxiong123/scGCL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:4a7bfcfcebd834adfa02dddc3f7b36bf92bf6e78217b36239baf2fc1e2c23eb0
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# high-dimensional-expression-matrix-neighborhood-search

## Summary

Constructs the cell graph for scGCL's graph contrastive learning by searching for k=15 nearest neighbors over the normalized HVG expression matrix using cosine distances. This preserves cell-to-cell topological relationships that existing scRNA-seq imputation methods ignore.

## When to use

Use when a pre-processed, normalized scRNA-seq HVG expression matrix must be converted into a cell-to-cell graph (one node per cell, kNN edges) as input to graph-based contrastive learning or downstream imputation.

## When NOT to use

- The expression matrix has not yet been normalized and restricted to highly variable genes — build it from the pre-processing filter output first
- The downstream method does not consume graph/topological structure over cells, so a kNN graph adds no benefit
- Only global cell-level statistics are needed (e.g. distribution fitting without graph contrastive learning); a full pairwise kNN graph is unnecessary overhead

## Inputs

- normalized HVG scRNA-seq expression matrix (one row per cell), e.g. from the h5 dataset file read by scGCL
- k nearest-neighbor count (k=15 in scGCL)

## Outputs

- sparse cell-by-cell adjacency matrix A (N cells x N cells) in torch-geometric edge format
- adjacency_matrix.npz saved for downstream graph contrastive learning

## How to apply

Load the normalized HVG expression matrix (the output of upstream pre-processing, e.g. the h5 file read by scGCL) into a PyTorch tensor with one row per cell. Compute pairwise cosine distances between all cell expression profiles with faiss-cpu 1.7.3. Retrieve the k=15 nearest neighbors for every cell via torch-cluster 1.6.0 kNN search (k=15 is the value used in scGCL). Assemble the adjacency matrix A in torch-geometric 2.2.0 edge format with one node per cell, symmetrizing the directed kNN edges so relationships between cells are mutually represented. Save the sparse adjacency to adjacency_matrix.npz for downstream graph contrastive learning, where it supplies the local structure that scGCL's positive-sample selection and ZINB autoencoder rely on.

## Related tools

- **faiss-cpu** (computes pairwise cosine distances between all cell expression profiles) — https://github.com/zehaoxiong123/scGCL
- **torch-cluster** (kNN search to retrieve the k=15 nearest neighbors for every cell) — https://github.com/zehaoxiong123/scGCL
- **torch-geometric** (assembles the adjacency matrix A in edge format with one node per cell) — https://github.com/zehaoxiong123/scGCL
- **PyTorch** (tensor representation of the normalized HVG expression matrix (one row per cell)) — https://github.com/zehaoxiong123/scGCL

## Examples

```
python main.py --dataset adam --epochs 300
```

## Evaluation signals

- A is a square N-cells x N-cells matrix
- every cell node has exactly its k=15 cosine-nearest neighbors retained
- kNN edges are symmetrized so cell a retaining cell b implies cell b also retains cell a
- distances are cosine distances computed from the normalized HVG matrix, not raw counts
- output file adjacency_matrix.npz exists and loads as a sparse matrix consumable by the graph contrastive learning stage

## Limitations

- k=15 is the specific value used in this work; no guidance is given in the article or README for tuning k on other datasets
- the graph is built over the pre-processed (normalized HVG) profiles, so quality depends entirely on the upstream normalization/feature-selection steps
- the README specifies exact library versions (Python 3.8.8, PyTorch 1.10.0, torch-cluster 1.6.0, torch-geometric 2.2.0, faiss-cpu 1.7.3) and does not guarantee behavior under other versions

## Evidence

- [methods] cell graph is built with one node per cell; an edge is assigned between cell a and cell b if cell b is one of the k nearest nodes to cell a, with k set to 15: [evidence span withheld — anchor status: not_found]
- [abstract] existing methods ignore topological structure: "existing scRNA-seq data imputation methods ignore the topological structure information of scRNA-seq data and cannot comprehensively utilize the relationships between cells"
- [readme] scGCL positive sample selection: "scGCL summarizes global and local semantic information through contrastive learning and selects appropriate positive samples to enhance the representation of target nodes"
- [readme] README dependency versions: "torch-cluster version: 1.6.0"
- [readme] dataset input format: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
