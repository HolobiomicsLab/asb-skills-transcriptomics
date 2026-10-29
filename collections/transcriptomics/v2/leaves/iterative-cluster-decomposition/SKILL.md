---
name: iterative-cluster-decomposition
description: Use when a scRNA-seq workflow needs to split large initial clusters (>R*n
  cells, R=0.01 or 30/n for small datasets) into finer sub-clusters to expose rare
  cell populations that one-time global clustering overlooks.
when_to_use_negative:
- Clusters are already small (all below R*n, i.e. under 1% of cells) — decomposition
  will not trigger and adds no value
- The goal is de novo rare-type detection without clustering context (e.g. FiRE/CIARA-style
  cell-level scoring) rather than refining an existing clustering
- Data is not raw count-level expression (e.g. already a feature/embedding table),
  since the skill expects gene-by-cell counts it can filter (>=3 cells) and normalize
  itself
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCAD
  role: 'Implements the iterative cluster decomposition (rapid clustering module:
    PCA 40 PCs, KNN k=15, Louvain) and the distance-based merge into M-clusters'
  repo: https://github.com/xuyp-csu/scCAD
- name: Scanpy
  role: Dependency providing PCA, nearest-neighbor graph, and Louvain clustering used
    inside the decomposition loop
  repo: https://github.com/xuyp-csu/scCAD
- name: python-louvain
  role: Louvain community detection used to produce initial I-clusters and each iteration's
    sub-clusters
  repo: https://github.com/xuyp-csu/scCAD
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:0bb07d6aa6e0f5b614d226afe2f6678e13e8bd467ca0f571f68d1769596a5a90
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
yes_rate: 0.7
n_claims: 10
---

# iterative-cluster-decomposition

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs scCAD's cluster decomposition pipeline: major I-clusters from initial Louvain clustering are iteratively re-clustered based on the most differential signals to yield D-clusters, which are then merged by cluster-center distance into M-clusters before rare cluster detection.

## When to use

Use when a scRNA-seq workflow needs to split large initial clusters (>R*n cells, R=0.01 or 30/n for small datasets) into finer sub-clusters to expose rare cell populations that one-time global clustering overlooks.

## When NOT to use

- Clusters are already small (all below R*n, i.e. under 1% of cells) — decomposition will not trigger and adds no value
- The goal is de novo rare-type detection without clustering context (e.g. FiRE/CIARA-style cell-level scoring) rather than refining an existing clustering
- Data is not raw count-level expression (e.g. already a feature/embedding table), since the skill expects gene-by-cell counts it can filter (>=3 cells) and normalize itself

## Inputs

- scRNA-seq expression matrix with cells in rows and genes in columns (e.g. 1%Jurkat.h5 h5py file with X, Y, gn, cn fields)
- Optional cell name and gene name arrays; scCAD generates them automatically if absent
- Parameters: rare_h (rare/decomposition threshold, default 0.01), merge_h (merge threshold, default 0.3), seed (default 2023), normalization flag (default True)

## Outputs

- M-cluster label assignment for every cell (per-cell sub-cluster labels, np.array[n cells])
- Per-cluster size summary written to cluster_decomposition_labels.csv
- Decomposed D-clusters available as input to downstream anomaly scoring (isolation forest on top 20 differential genes per cluster)

## How to apply

Load the expression matrix (cells x genes, e.g. 1%Jurkat.h5), retain genes expressed in at least three cells, and log-normalize with cell-specific size factors plus log2(x+1). Compute the top 40 principal components, build a 15-nearest-neighbor Euclidean graph, and run Louvain to obtain initial I-clusters. Iteratively re-apply the rapid clustering module (PCA 40 PCs, KNN k=15, Louvain) to any cluster larger than R*n (R=0.01, or 30/n when the dataset has fewer than 3000 cells) until no new clusters emerge or all clusters fall below R*n, yielding D-clusters. Compute each D-cluster's per-gene mean center, set the merge threshold THM to the median of each cluster's nearest-neighbor center distance (THM=median(d1, d2,…, dm); merge_h default 0.3 in the code), and merge pairs whose center distance is below THM to obtain M-clusters, writing per-cell labels to cluster_decomposition_labels.csv.

## Related tools

- **scCAD** (Implements the iterative cluster decomposition (rapid clustering module: PCA 40 PCs, KNN k=15, Louvain) and the distance-based merge into M-clusters) — https://github.com/xuyp-csu/scCAD
- **Scanpy** (Dependency providing PCA, nearest-neighbor graph, and Louvain clustering used inside the decomposition loop) — https://github.com/xuyp-csu/scCAD
- **python-louvain** (Louvain community detection used to produce initial I-clusters and each iteration's sub-clusters) — https://github.com/xuyp-csu/scCAD

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- cluster_decomposition_labels.csv exists and its row count equals the number of input cells (1556 for the 1%Jurkat example)
- Decomposition terminates: no cluster larger than R*n remains unsplit (R=0.01, or 30/n for datasets under 3000 cells)
- The merge step produces M-clusters whose pairwise center distances fall below THM only for merged pairs, with THM defined as the median nearest-neighbor center distance
- On 1%Jurkat.h5, decomposition isolates at least one small (sub-1%) M-cluster corresponding to the 16 Jurkat cells among 1540 293T cells
- Pipeline completes in reasonable runtime (~40 seconds for the 1556-cell demo dataset)

## Limitations

- Decomposition only targets clusters exceeding R*n; rare populations hidden within small initial clusters will not be further split
- Merging by median nearest-neighbor center distance (THM, merge_h default 0.3) may over- or under-merge when cluster center geometry is uninformative
- Results depend on the random seed (default 2023) and on Louvain's initial I-clusters; parameters tested with the pinned dependency versions in requirements.txt (Python 3.7.13, Scanpy 1.9.1, scikit-learn 1.0.2)
- Requires in-memory operations; only a standard computer with enough RAM is assumed, so very large datasets may be constrained

## Evidence

- [discussion] Iteratively decompose clusters > R*n via rapid clustering module: [evidence span withheld — anchor status: not_found]
- [discussion] Merge decomposed clusters by THM: [evidence span withheld — anchor status: not_found]
- [discussion] PCA 40 PCs, KNN k=15, Louvain parameters: [evidence span withheld — anchor status: not_found]
