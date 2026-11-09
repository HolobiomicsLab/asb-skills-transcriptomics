---
name: knn-graph-construction-from-cell-embeddings
description: Use when paired scRNA-seq/scATAC-seq data need a cell-state DAG (not
  just a clustering or trajectory) so that Granger causal locus-gene inference can
  be run over an acyclic cell ordering.
when_to_use_negative:
- The data are not paired multimodal single-cell observations from a dynamic process
  (e.g. differentiation or disease) — pseudotime orientation requires a dynamic biological
  process to provide temporal context
- You only need cell clustering or visualization, not a directed acyclic structure
  for causal inference — a plain undirected kNN graph suffices
- Cells lack a biologically meaningful root cell type, making diffusion pseudotime
  rooting and edge orientation unreliable
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Schema
  role: Learns joint cell representations that unify RNA and ATAC modalities; the
    kNN graph is built from these embeddings
  repo: https://github.com/rs239/schema
- name: GrID-Net
  role: Provides construct_dag and dag_orient_edges to build the kNN graph (n_neighbors=15,
    pseudotime_algo='dpt') and orient edges by pseudotime into a DAG
  repo: https://github.com/alexw16/gridnet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:9fddb09611938f285ed547e32b029ffe99c10626d252d68a934d09bda51bf6d4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  repo_url: https://github.com/rs239/schema
  tools:
  - GrID-Net
  - Schema
  - scMoMaT
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# knn-graph-construction-from-cell-embeddings

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a directed acyclic graph (DAG) of cell states by building a k-nearest-neighbor graph over Schema-derived joint RNA+ATAC cell embeddings and orienting each edge by diffusion pseudotime, providing the DAG structure GrID-Net requires for Granger causal inference.

## When to use

Use when paired scRNA-seq/scATAC-seq data need a cell-state DAG (not just a clustering or trajectory) so that Granger causal locus-gene inference can be run over an acyclic cell ordering.

## When NOT to use

- The data are not paired multimodal single-cell observations from a dynamic process (e.g. differentiation or disease) — pseudotime orientation requires a dynamic biological process to provide temporal context
- You only need cell clustering or visualization, not a directed acyclic structure for causal inference — a plain undirected kNN graph suffices
- Cells lack a biologically meaningful root cell type, making diffusion pseudotime rooting and edge orientation unreliable

## Inputs

- Joint multimodal cell embeddings X_joint from Schema (one embedding per cell, unifying RNA-seq and ATAC-seq)
- Root cell index iroot (or a root cell-type marker gene such as TOP2A for pseudotime rooting)
- Paired, log(1+x)-normalized RNA-seq and ATAC-seq AnnData objects (preprocessed with gridnet.preprocess_multimodal)

## Outputs

- DAG adjacency matrix (scipy.sparse.csr_matrix or numpy.ndarray, N x N, e.g. 5000 x 5000 for a 5,000-cell 10x Multiome sample), saved e.g. as cell_state_dag.npz
- Pseudotime-ordered, acyclic cell-state graph whose node ordering matches the row order of X and Y passed to gridnet.run_gridnet

## How to apply

First learn joint multimodal cell representations with Schema (via gridnet.schema_representations), which unifies the RNA-seq and ATAC-seQ modalities for the same cells. Build a k-nearest neighbor graph from these embeddings using Euclidean distances with k=15 (gridnet.construct_dag with n_neighbors=15). Infer pseudotime with the scanpy implementation of diffusion pseudotime ('dpt'), rooted at a supplied root cell index (iroot) or a marker gene for the root cell type (e.g. TOP2A). Orient every kNN edge from lower to higher pseudotime (gridnet.dag_orient_edges) so the resulting adjacency matrix is acyclic; alternatively, RNA-velocity-based edge orientations may be supplied provided any resulting cycles are broken. Save the N x N DAG adjacency matrix (scipy.sparse.csr_matrix or numpy.ndarray, rows/columns ordered to match the observations in X and Y) for downstream gridnet.run_gridnet.

## Related tools

- **Schema** (Learns joint cell representations that unify RNA and ATAC modalities; the kNN graph is built from these embeddings) — https://github.com/rs239/schema
- **GrID-Net** (Provides construct_dag and dag_orient_edges to build the kNN graph (n_neighbors=15, pseudotime_algo='dpt') and orient edges by pseudotime into a DAG) — https://github.com/alexw16/gridnet

## Examples

```
X_joint = gridnet.schema_representations(rna_adata, atac_adata); dag = gridnet.construct_dag(X_joint, iroot=0, n_neighbors=15, pseudotime_algo='dpt')
```

## Evaluation signals

- The saved adjacency matrix has exactly N rows and columns matching the number of cells in the sample (e.g. 5000 for the 10x Multiome sample)
- A topological-sort script confirms the oriented graph is acyclic (no directed cycles)
- Every DAG edge is oriented from lower to higher pseudotime (endpoint pseudotime values are strictly increasing along each edge)
- The adjacency-matrix row/column order matches the observation order of X and Y matrices later passed to gridnet.run_gridnet

## Limitations

- GrID-Net relies on multiomic single-cell data from matching tissue undergoing a dynamic process (e.g. differentiation or disease) to provide the pseudotemporal context; without it, edge orientation is unfounded
- Large single-cell datasets may impose computational burden exceeding GPU memory capacity
- Root-cell choice for diffusion pseudotime critically determines edge orientation; the RNA-velocity alternative requires any resulting cycles to be broken manually

## Evidence

- [discussion] Concrete construction parameters (kNN k=15, Euclidean, diffusion pseudotime): [evidence span withheld — anchor status: not_found]
