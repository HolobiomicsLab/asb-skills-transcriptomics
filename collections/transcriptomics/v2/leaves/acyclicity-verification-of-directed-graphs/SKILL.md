---
name: acyclicity-verification-of-directed-graphs
description: Use when a kNN cell graph has been edge-oriented by pseudotime (e.g.,
  DPT rooted at a given iroot) and the N×N adjacency matrix must be confirmed acyclic
  before Granger causal inference on cell states.
when_to_use_negative:
- The graph is undirected or edge weights have no orientation information — acyclicity
  is undefined without directed edges.
- The edge orientations come from cyclic processes with no meaningful pseudotime ordering
  (e.g., cell-cycle-only data), where forcing a DAG would misrepresent biology rather
  than merely require cycle-breaking.
tools:
- name: GrID-Net
  role: Constructs the pseudotime-oriented kNN cell-state DAG (gridnet.construct_dag)
    whose acyclicity is verified before Granger inference
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Provides the joint RNA+ATAC cell representations on which the kNN graph (and
    hence the DAG) is built
  repo: https://github.com/rs239/schema
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:340fa7669cd0c4c4bdad73e6403eb1125895f72cf53c1f5f434347fa397eef9b
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# acyclicity-verification-of-directed-graphs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verifies that a directed graph over cells—such as the pseudotime-oriented kNN graph used by GrID-Net—is a true directed acyclic graph (DAG) before it is passed to Granger causal inference, since Granger causality on partial orderings relies on the graph having no cycles.

## When to use

Use when a kNN cell graph has been edge-oriented by pseudotime (e.g., DPT rooted at a given iroot) and the N×N adjacency matrix must be confirmed acyclic before Granger causal inference on cell states.

## When NOT to use

- The graph is undirected or edge weights have no orientation information — acyclicity is undefined without directed edges.
- The edge orientations come from cyclic processes with no meaningful pseudotime ordering (e.g., cell-cycle-only data), where forcing a DAG would misrepresent biology rather than merely require cycle-breaking.

## Inputs

- Directed adjacency matrix over cells (scipy.sparse.csr_matrix or numpy.ndarray, N x N, e.g., N=5000) produced by gridnet.construct_dag
- Root cell index (iroot) used for diffusion pseudotime rooting

## Outputs

- Boolean acyclicity verdict (or successful topological ordering) for the cell-state DAG
- Validation report confirming node count (N matching the cell sample) and absence of directed cycles

## How to apply

After building a k-nearest neighbor graph (k=15, Euclidean distances) on Schema joint RNA+ATAC cell representations, orient each edge from lower to higher pseudotime using scanpy's diffusion pseudotime rooted at a supplied root cell (gridnet.construct_dag with pseudotime_algo='dpt'). Orienting by a scalar pseudotime should yield a DAG, but any edge set (e.g., RNA-velocity-based orientations) can introduce cycles that must be detected and broken. Load the resulting N×N adjacency matrix (scipy.sparse.csr_matrix or numpy.ndarray, N cells), run a topological sort or equivalent acyclicity check over the directed graph, and confirm the node count matches the cell number (e.g., N=5000 for the 10x Multiome corticogenesis sample). Only an acyclic adjacency matrix is a valid input to gridnet.run_gridnet, which performs lagged message passing on the DAG structure.

## Related tools

- **GrID-Net** (Constructs the pseudotime-oriented kNN cell-state DAG (gridnet.construct_dag) whose acyclicity is verified before Granger inference) — https://github.com/alexw16/gridnet
- **Schema** (Provides the joint RNA+ATAC cell representations on which the kNN graph (and hence the DAG) is built) — https://github.com/rs239/schema

## Examples

```
import networkx as nx; G = nx.from_scipy_sparse_array(dag_adjacency_matrix, create_using=nx.DiGraph); assert nx.is_directed_acyclic_graph(G) and G.number_of_nodes() == 5000
```

## Evaluation signals

- A topological-sort script completes without error, confirming the directed graph is acyclic.
- The adjacency matrix has exactly N rows/columns matching the cell sample (N=5000 for the 10x Multiome task).
- No self-loops and no directed cycles present when checking each strongly connected component.
- Every directed edge points from the lower-pseudotime cell to the higher-pseudotime cell under the DPT values rooted at iroot.
- The saved cell_state_dag.npz loads as a scipy.sparse.csr_matrix or numpy.ndarray of shape (N, N).

## Limitations

- Acyclicity of the kNN DAG depends on the pseudotime orientation; alternative edge orientations (e.g., RNA-velocity-based) can produce cycles that must be explicitly broken.
- Verification guarantees structural validity of the DAG but not correctness of the underlying pseudotime ordering or biological trajectory.

## Evidence

- [intro] DAG construction from kNN + pseudotime: "a DAG of cell states is constructed from a k-nearest neighbor graph of cells, with edges oriented in accordance with pseudotime computed after a multimodal integration with Schema"
- [discussion] kNN and DPT parameters: [evidence span withheld — anchor status: not_found]
- [readme] construct_dag call: "dag_adjacency_matrix = gridnet.construct_dag(X_joint,iroot,n_neighbors=15,pseudotime_algo='dpt')"
- [readme] DAG required for inference: "GrID-Net (Granger Inference on DAGs) is a graph neural network framework for Granger causal inference on directed acyclic graph (DAG)-structured dynamical systems"
