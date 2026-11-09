---
name: louvain-graph-clustering
description: Use when a scRNA-seq clustering step must discover data-driven cell groups
  from a PCA/KNN graph — e.g. generating I-clusters prior to scCAD's cluster decomposition
  or splitting clusters larger than R*n cells.
when_to_use_negative:
- The input is already a cluster/partition assignment and no graph-based re-discovery
  of cell groups is needed
- The goal is differential expression or anomaly scoring on fixed clusters rather
  than discovering or splitting clusters via a KNN graph
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCAD
  role: Wraps the PCA(40 PCs) -> KNN(k=15) -> Louvain rapid clustering module and
    applies it iteratively to decompose large clusters
  repo: https://github.com/xuyp-csu/scCAD
- name: python-louvain
  role: Provides Louvain community detection executed on the 15-NN Euclidean graph
    to define clusters
- name: Scanpy
  role: Dependency (version 1.9.1) of the scCAD environment used for the clustering/preprocessing
    backbone
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:120ca2db8a473744d07dac94d58d7a1ed3db809cc678208feb290281002bdf33
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  repo_url: https://github.com/xuyp-csu/scCAD
  tools:
  - Scanpy
  - python-louvain
  - scCAD
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# louvain-graph-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Louvain community detection on a k-nearest-neighbor graph built from top principal components, used within scCAD both to obtain initial I-clusters and, inside the rapid clustering module, to iteratively decompose large clusters into D-clusters for rare cell detection.

## When to use

Use when a scRNA-seq clustering step must discover data-driven cell groups from a PCA/KNN graph — e.g. generating I-clusters prior to scCAD's cluster decomposition or splitting clusters larger than R*n cells.

## When NOT to use

- The input is already a cluster/partition assignment and no graph-based re-discovery of cell groups is needed
- The goal is differential expression or anomaly scoring on fixed clusters rather than discovering or splitting clusters via a KNN graph

## Inputs

- Log-normalized scRNA-seq expression matrix (cells x genes), e.g. from an .h5 file like 1%Jurkat.h5
- Top 40 principal components of the normalized matrix
- KNN graph parameters: k=15 neighbors, Euclidean metric

## Outputs

- Initial I-cluster labels per cell (Louvain communities)
- Decomposed D-cluster labels per cell after iterative re-clustering of clusters larger than R*n
- Per-cell sub-cluster label assignments (np.array[n cells]) returned by scCAD

## How to apply

Log-normalize the count matrix with cell-specific size factors (log2(x+1) after a pseudo-count of 1), retaining genes expressed in at least three cells, then run PCA to the top 40 principal components. Build a 15-nearest-neighbor Euclidean graph on the PC embedding and run Louvain (python-louvain 0.16 in the scCAD environment) to obtain the initial I-clusters. During decomposition, re-apply this same rapid clustering module to any cluster containing more than R*n cells (R=0.01, or 30/n when the dataset has fewer than 3000 cells), continuing until no new clusters are produced or all clusters fall below R*n, yielding D-clusters. The rationale is that a single global Louvain pass can miss rare populations, so decomposing based on the most differential signals in each cluster lets small groups (e.g. 16 Jurkat cells among 1540 293T cells in the 1%Jurkat.h5 demo) emerge as their own graph communities.

## Related tools

- **scCAD** (Wraps the PCA(40 PCs) -> KNN(k=15) -> Louvain rapid clustering module and applies it iteratively to decompose large clusters) — https://github.com/xuyp-csu/scCAD
- **python-louvain** (Provides Louvain community detection executed on the 15-NN Euclidean graph to define clusters)
- **Scanpy** (Dependency (version 1.9.1) of the scCAD environment used for the clustering/preprocessing backbone)

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- The cluster label CSV (e.g. cluster_decomposition_labels.csv) has one row per input cell — 1556 rows for the 1%Jurkat demo
- Iterative decomposition terminates: no cluster larger than R*n cells remains (R=0.01, or 30/n for datasets with fewer than 3000 cells)
- Decomposition isolates at least one small (sub-1%) M-cluster corresponding to the 16 Jurkat cells in the 1%Jurkat.h5 example
- Re-running with the same seed (default seed=2023) reproduces the same community assignments

## Limitations

- One-time global Louvain clustering can overlook rare cell types, which is why scCAD iterates the decomposition based on the most differential signals in each cluster
- Cluster decomposition only recurses on clusters larger than R*n cells, so populations below this threshold are never split further
- Results depend on the random seed (default 2023) and on preprocessing choices (genes retained in at least three cells, 40 PCs, k=15)

## Evidence

- [discussion] PCA to top 40 principal components, KNN graph (k=15), Louvain clustering: [evidence span withheld — anchor status: not_found]
- [discussion] R threshold and decomposition trigger: [evidence span withheld — anchor status: not_found]
