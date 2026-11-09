---
name: pseudotime-oriented-trajectory-modeling
description: Use when paired single-cell multimodal data (e.g., scRNA-seq + scATAC-seq
  from the same cells) captures a dynamic process such as differentiation, and a temporally
  oriented cell-state graph is needed for causal or trajectory modeling.
when_to_use_negative:
- Modalities are not assayed for the same cells simultaneously (Schema is designed
  for jointly profiled multimodal data), so unpaired scRNA-seq and scATAC-seq cannot
  be integrated this way
- The biological system is static or lacks a dynamic process (e.g., differentiation
  or disease progression), so no meaningful pseudotemporal context exists for orienting
  the DAG
- The dataset is too large for available memory/GPU capacity — the article notes the
  computational burden of large single-cell datasets may exceed GPU memory; sketching
  tools like Geosketch or Hopper are recommended instead
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Schema
  role: Learns joint multimodal cell representations that unify RNA and ATAC modalities,
    providing the Euclidean feature space for kNN graph construction and pseudotime
    inference
  repo: https://github.com/rs239/schema
- name: GrID-Net (gridnet)
  role: Provides schema_representations and construct_dag utilities that assemble
    the Schema embedding, diffusion pseudotime, and kNN orientation into a DAG adjacency
    matrix for downstream Granger inference
  repo: https://github.com/alexw16/gridnet
- name: PyTorch
  role: Framework in which the downstream GrID-Net GNN models consuming the DAG are
    implemented and trained
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:d523c02adb56c14c226562aaf5fa2825556c05aef03e8c35f23c3bded6f40e93
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
  - GrID-Net (gridnet)
  - NVIDIA Tesla V100 GPU
  - PyTorch
  - Schema
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pseudotime-oriented-trajectory-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a directed acyclic graph (DAG) of cell states by embedding paired scRNA-seq/scATAC-seq cells with Schema multimodal integration, building a k=15 k-nearest-neighbor graph, inferring diffusion pseudotime, and orienting each kNN edge from lower to higher pseudotime so downstream Granger causal inference respects the partial ordering of cell states.

## When to use

Use when paired single-cell multimodal data (e.g., scRNA-seq + scATAC-seq from the same cells) captures a dynamic process such as differentiation, and a temporally oriented cell-state graph is needed for causal or trajectory modeling.

## When NOT to use

- Modalities are not assayed for the same cells simultaneously (Schema is designed for jointly profiled multimodal data), so unpaired scRNA-seq and scATAC-seq cannot be integrated this way
- The biological system is static or lacks a dynamic process (e.g., differentiation or disease progression), so no meaningful pseudotemporal context exists for orienting the DAG
- The dataset is too large for available memory/GPU capacity — the article notes the computational burden of large single-cell datasets may exceed GPU memory; sketching tools like Geosketch or Hopper are recommended instead

## Inputs

- Paired AnnData objects: unprocessed RNA-seq AnnData (with gene TSS annotations: chr_no, txstart in adata.var) and ATAC-seq AnnData (with peak annotations: chr_no, start, end in adata.var)
- Root cell specification: either a root cell marker gene name (e.g., 'TOP2A') or a root cell index (iroot)
- Schema joint cell representation matrix X_joint (N x d), produced by gridnet.schema_representations

## Outputs

- DAG adjacency matrix (N x N) whose rows and columns are ordered consistently with the rows of the X and Y matrices passed to gridnet.run_gridnet
- Diffusion pseudotime values per cell used to orient kNN edges

## How to apply

First learn a joint cell representation with Schema (gridnet.schema_representations(rna_adata, atac_adata)), which unifies information from the RNA and ATAC modalities via metric learning formulated as a quadratic programming problem and runs efficiently on CPU without a GPU. Then build the DAG with gridnet.construct_dag(X_joint, iroot, n_neighbors=15, pseudotime_algo='dpt'), supplying either a root cell index or a root cell identified by a marker gene (e.g., TOP2A for cycling progenitors); the kNN graph uses Euclidean distances between Schema representations with k=15 neighbors, and pseudotime is inferred with the scanpy implementation of diffusion pseudotime (DPT). Finally, orient every kNN edge in the direction of increasing pseudotime to yield a DAG whose adjacency matrix rows/columns are ordered consistently with the rows of the peak (X) and gene (Y) count matrices, since GrID-Net's lagged message passing requires this pairing. The DAG is the structural precondition that lets Granger causality generalize from total orderings to partial orderings on the cell trajectory.

## Related tools

- **Schema** (Learns joint multimodal cell representations that unify RNA and ATAC modalities, providing the Euclidean feature space for kNN graph construction and pseudotime inference) — https://github.com/rs239/schema
- **GrID-Net (gridnet)** (Provides schema_representations and construct_dag utilities that assemble the Schema embedding, diffusion pseudotime, and kNN orientation into a DAG adjacency matrix for downstream Granger inference) — https://github.com/alexw16/gridnet
- **PyTorch** (Framework in which the downstream GrID-Net GNN models consuming the DAG are implemented and trained)

## Examples

```
X_joint = gridnet.schema_representations(rna_adata, atac_adata); dag_adjacency_matrix = gridnet.construct_dag(X_joint, iroot, n_neighbors=15, pseudotime_algo='dpt')
```

## Evaluation signals

- Verify the DAG is acyclic: all kNN edges must be oriented strictly from lower to higher diffusion pseudotime, so no directed cycles exist
- Confirm adjacency matrix dimensions are N x N and its row/column ordering matches the row ordering of the paired X (peaks) and Y (genes) matrices
- Check that the kNN graph used k=15 neighbors on Euclidean distances between Schema joint representations (matching the paper's settings)
- Root-cell choice (marker gene such as TOP2A or explicit index) should place the pseudotime origin in the expected progenitor/root cell type
- Downstream GrID-Net run should execute without pairing errors, confirming DAG and data matrices are consistently indexed

## Limitations

- Requires multiomic single-cell data from matching tissue undergoing a dynamic biological process to provide pseudotemporal context
- Diffusion pseudotime provides a partial, not total, ordering; mis-estimated pseudotime or an incorrect root cell can misorient edges and corrupt causal inference
- Large datasets may exceed GPU memory capacity, motivating sketching (Geosketch or Hopper) before graph construction

## Evidence

- [discussion] We constructed a k-nearest neighbor graph (k=15) based on the Euclidean distances between these cell representations and inferred pseudotime using the scanpy91 implementation of diffusion pseudotime92: [evidence span withheld — anchor status: not_found]
