---
name: diffusion-pseudotime-inference
description: 'Use when constructing a cell-state DAG from single-cell multimodal (e.g.,
  paired RNA-seq/ATAC-seq) data: a kNN graph needs edge orientations grounded in a
  pseudotemporal process such as differentiation or disease progression.'
when_to_use_negative:
- The single-cell multimodal data come from a static tissue or non-dynamic process
  — the method relies on multiomic data from tissue undergoing a dynamic process,
  such as differentiation or disease, to provide pseudotemporal context.
- A total order over cells is already available from an external source and no cell-state
  DAG needs to be derived from the kNN graph.
- Input is bulk or otherwise not single-cell multimodal data, since Schema requires
  modalities assayed for the same cells simultaneously.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: GrID-Net
  role: Provides construct_dag/dag_orient_edges which build the kNN graph (n_neighbors=15,
    Euclidean distances) and orient edges by diffusion pseudotime to yield the cell-state
    DAG for downstream Granger causal inference.
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint feature embeddings (X_joint) unifying RNA expression and chromatin
    accessibility representations, on which the kNN graph and DPT pseudotime are computed.
  repo: https://github.com/rs239/schema
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:63796b01ddf8f58eba412edd8f1a5220be9a56bb0207d2b8986acdff97774c32
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  repo_url: https://github.com/alexw16/gridnet
  tools:
  - GrID-Net
  - Schema
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# diffusion-pseudotime-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers a pseudotemporal ordering of cells using scanpy's diffusion pseudotime (DPT) on Schema multimodal joint embeddings, then orients kNN graph edges from lower to higher pseudotime to build the directed acyclic cell-state graph required for GrID-Net Granger causal inference.

## When to use

Use when constructing a cell-state DAG from single-cell multimodal (e.g., paired RNA-seq/ATAC-seq) data: a kNN graph needs edge orientations grounded in a pseudotemporal process such as differentiation or disease progression.

## When NOT to use

- The single-cell multimodal data come from a static tissue or non-dynamic process — the method relies on multiomic data from tissue undergoing a dynamic process, such as differentiation or disease, to provide pseudotemporal context.
- A total order over cells is already available from an external source and no cell-state DAG needs to be derived from the kNN graph.
- Input is bulk or otherwise not single-cell multimodal data, since Schema requires modalities assayed for the same cells simultaneously.

## Inputs

- Preprocessed (normalized, log(1+x)) paired RNA-seq and ATAC-seq AnnData objects for the same cells (e.g., 10x Multiome sample)
- Joint multimodal cell embedding matrix X_joint from gridnet.schema_representations (Schema integration of RNA and ATAC modalities)
- Root cell index iroot (or a root cell type marker gene such as TOP2A)

## Outputs

- N x N directed acyclic adjacency matrix of cell states (scipy.sparse.csr_matrix or numpy.ndarray), e.g. N=5000, saved as cell_state_dag.npz
- Per-cell pseudotime values used to orient kNN edges

## How to apply

First preprocess paired RNA and ATAC counts with count normalization and log(1+x) transformation via gridnet.preprocess_multimodal, then learn joint cell representations that unify both modalities with gridnet.schema_representations. Build a k-nearest neighbor graph (k=15) from Euclidean distances between these Schema representations and infer pseudotime using the scanpy implementation of diffusion pseudotime (pseudotime_algo='dpt'), rooted at a supplied root cell index iroot (or a root determined by a marker gene such as TOP2A in the gridnet_multimodal wrapper). Orient each kNN edge from lower to higher pseudotime with gridnet.construct_dag/gridnet.dag_orient_edges so the resulting adjacency matrix is acyclic; alternatively, RNA-velocity-based orientations may be supplied provided any cycles are broken. The rationale is that Granger causal inference requires a temporal ordering, and DPT on multimodal embeddings supplies a partial ordering over cell states that generalizes Granger causality from total to partial orderings.

## Related tools

- **GrID-Net** (Provides construct_dag/dag_orient_edges which build the kNN graph (n_neighbors=15, Euclidean distances) and orient edges by diffusion pseudotime to yield the cell-state DAG for downstream Granger causal inference.) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint feature embeddings (X_joint) unifying RNA expression and chromatin accessibility representations, on which the kNN graph and DPT pseudotime are computed.) — https://github.com/rs239/schema

## Examples

```
X_joint = gridnet.schema_representations(rna_adata, atac_adata); dag_adjacency_matrix = gridnet.construct_dag(X_joint, iroot, n_neighbors=15, pseudotime_algo='dpt')
```

## Evaluation signals

- The saved adjacency matrix (cell_state_dag.npz) has exactly 5000 rows and columns matching the number of cells in the sample.
- A topological-sort check on the DAG adjacency matrix confirms the graph is acyclic.
- Edges are oriented strictly from lower to higher diffusion pseudotime (no back edges relative to DPT values).
- The kNN graph was built with k=15 on Schema joint embeddings using Euclidean distances, matching the paper's parameters.
- Row/column order of the adjacency matrix corresponds to observation order used for X and Y in gridnet.run_gridnet.

## Limitations

- Relies on multiomic single-cell data from matching tissue undergoing a dynamic biological process (e.g., differentiation or disease) to provide pseudotemporal context.
- If RNA-velocity-based edge orientations are used instead of pseudotime, any resulting cycles must be manually broken to obtain a valid DAG.
- Alternative pseudotime/ordering choices (e.g., different roots or algorithms) will change the DAG and thus the inferred causal relationships.

## Evidence

- [discussion] kNN k=15, Euclidean distances, scanpy diffusion pseudotime: [evidence span withheld — anchor status: not_found]
