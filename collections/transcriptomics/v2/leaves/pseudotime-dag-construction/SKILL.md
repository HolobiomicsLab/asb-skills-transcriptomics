---
name: pseudotime-dag-construction
description: Use when single-cell multimodal (paired RNA-seq + ATAC-seq) trajectories
  need a cell-state partial ordering — not a total pseudotime — to support lagged
  message-passing causal inference over peak-gene pairs.
when_to_use_negative:
- Data lack a dynamic biological process (e.g., differentiation or disease trajectory)
  supplying pseudotemporal context — GrID-Net relies on multiomic data from matching
  tissue undergoing a dynamic process
- Modalities were not assayed for the same cells simultaneously — Schema is designed
  for simultaneously profiled multimodal data
- The analysis requires a total ordering (classic linear Granger) rather than a partial
  ordering over branching trajectories
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: GrID-Net
  role: Provides schema_representations and construct_dag functions that build the
    multimodal embedding and orient the kNN graph into a DAG of cells
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Metric-learning multimodal integration that generates the joint RNA+ATAC cell
    embeddings from which the kNN graph and pseudotime are computed
  repo: https://github.com/rs239/schema
- name: PyTorch
  role: Implementation backend for GrID-Net GNN models trained on the constructed
    DAG
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:96b3b92ae6eb402b738a29bf7a14b1d9c96e925a50fe9228f3d719361ba423d7
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
  - PyTorch
  - Schema
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pseudotime-dag-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a directed acyclic graph (DAG) of cell states from single-cell multimodal data by learning joint RNA+ATAC representations with Schema, building a k-nearest neighbor graph (k=15), and orienting edges by diffusion pseudotime so that Granger causal inference (GrID-Net) can operate on partial orderings.

## When to use

Use when single-cell multimodal (paired RNA-seq + ATAC-seq) trajectories need a cell-state partial ordering — not a total pseudotime — to support lagged message-passing causal inference over peak-gene pairs.

## When NOT to use

- Data lack a dynamic biological process (e.g., differentiation or disease trajectory) supplying pseudotemporal context — GrID-Net relies on multiomic data from matching tissue undergoing a dynamic process
- Modalities were not assayed for the same cells simultaneously — Schema is designed for simultaneously profiled multimodal data
- The analysis requires a total ordering (classic linear Granger) rather than a partial ordering over branching trajectories

## Inputs

- Preprocessed RNA-seq AnnData (normalized, log(1+x); adata.var with chr_no, txstart TSS annotations)
- Preprocessed ATAC-seq AnnData (normalized, log(1+x); adata.var with chr_no, start, end annotations)
- Root cell marker gene (e.g., TOP2A) or root cell index for diffusion pseudotime anchoring
- Optional lineage/subset annotations for branch-specific DAG construction

## Outputs

- Joint multimodal cell-representation matrix (X_joint) from Schema
- DAG adjacency matrix (N x N) with edges oriented by increasing pseudotime, rows/columns ordered to match X and Y in run_gridnet

## How to apply

First normalize counts and apply log(1+x) transformation to the RNA and ATAC AnnData objects with gridnet.preprocess_multimodal. Learn a joint cell representation unifying both modalities via gridnet.schema_representations (Schema metric-learning embedding). Build a k-nearest neighbor graph from Euclidean distances between these representations with k=15 neighbors, infer pseudotime using the scanpy implementation of diffusion pseudotime ('dpt') anchored at a root cell (e.g., given by a marker gene such as TOP2A or a root index), then orient each kNN edge from lower to higher pseudotime with gridnet.construct_dag to yield a DAG adjacency matrix. This DAG is the required partial-ordering input for gridnet.run_gridnet; subsetting into branch-specific DAGs (e.g., cortex, medulla, IRS lineages in SHARE-seq) with identical settings is supported for ablation designs.

## Related tools

- **GrID-Net** (Provides schema_representations and construct_dag functions that build the multimodal embedding and orient the kNN graph into a DAG of cells) — https://github.com/alexw16/gridnet
- **Schema** (Metric-learning multimodal integration that generates the joint RNA+ATAC cell embeddings from which the kNN graph and pseudotime are computed) — https://github.com/rs239/schema
- **PyTorch** (Implementation backend for GrID-Net GNN models trained on the constructed DAG)

## Examples

```
X_joint = gridnet.schema_representations(rna_adata, atac_adata); dag_adjacency_matrix = gridnet.construct_dag(X_joint, iroot, n_neighbors=15, pseudotime_algo='dpt')
```

## Evaluation signals

- The DAG adjacency matrix is acyclic and its rows/columns are ordered consistently with the observation order of X and Y passed to gridnet.run_gridnet
- Every directed edge goes from a cell with lower diffusion pseudotime to one with higher pseudotime (orientation check against dpt values)
- The kNN graph is built on Schema joint embeddings with n_neighbors=15 and pseudotime_algo='dpt', matching the reported configuration
- Downstream GrID-Net AUROC on eQTL-supported peak-gene pairs should exceed baseline (random/linear-Granger) performance, e.g., the reported 7-22% AUROC gain over the next best method in 3 of 4 datasets
- Branch ablation sanity check: GrID-Net on the full three-branch SHARE-seq DAG outperforms linear Granger on full trajectory and individual branches

## Limitations

- Requires multiomic single-cell data from matching tissue undergoing a dynamic biological process to provide meaningful pseudotemporal context
- Large single-cell datasets may exceed GPU memory capacity (models trained on a single NVIDIA Tesla V100)
- The DAG represents inferred causal orderings on observational data; associations remain hypotheses needing perturbation validation

## Evidence

- [discussion] kNN k=15 and scanpy DPT: [evidence span withheld — anchor status: not_found]
