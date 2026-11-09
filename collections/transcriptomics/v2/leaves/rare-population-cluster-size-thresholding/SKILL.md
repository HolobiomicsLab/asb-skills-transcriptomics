---
name: rare-population-cluster-size-thresholding
description: Use when a scRNA-seq clustering result may hide rare cell types inside
  large clusters, and you need to recursively decompose oversized clusters (R*n threshold)
  and merge near-identical sub-clusters before rare-population scoring.
when_to_use_negative:
- Input is not raw/normalized single-cell expression data (e.g. already a feature
  table, marker matrix, or bulk expression profile) since decomposition requires per-cell
  expression vectors for PCA and cluster centers
- Clusters are already all below the R*n size threshold and rare populations are expected
  to be cleanly separated by one-time Louvain clustering, making recursive decomposition
  unnecessary
- The goal is differential expression testing between known groups rather than discovery
  of rare, previously unlabeled sub-populations
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCAD
  role: Implements the iterative cluster decomposition (clusters larger than R=1%
    of cells) and merging of D-clusters into M-clusters before rare cluster detection
  repo: https://github.com/xuyp-csu/scCAD
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:0a50b41b3042f264aeba90a64d88ff96441a5147e0e639eb64dc7ebc2fa650e2
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
  - scCAD
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# rare-population-cluster-size-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Iterative cluster decomposition and merging used by scCAD: initial Louvain I-clusters are recursively split while any cluster exceeds R*n cells, then decomposed D-clusters are merged via center-distance threshold THM to yield M-clusters prior to rare cluster detection.

## When to use

Use when a scRNA-seq clustering result may hide rare cell types inside large clusters, and you need to recursively decompose oversized clusters (R*n threshold) and merge near-identical sub-clusters before rare-population scoring.

## When NOT to use

- Input is not raw/normalized single-cell expression data (e.g. already a feature table, marker matrix, or bulk expression profile) since decomposition requires per-cell expression vectors for PCA and cluster centers
- Clusters are already all below the R*n size threshold and rare populations are expected to be cleanly separated by one-time Louvain clustering, making recursive decomposition unnecessary
- The goal is differential expression testing between known groups rather than discovery of rare, previously unlabeled sub-populations

## Inputs

- Preprocessed cell-by-gene expression matrix (e.g. 1%Jurkat.h5 with 1556 cells x 32738 genes), genes retained if expressed in at least three cells, log-normalized with cell-specific size factors then log2(x+1)
- Initial I-cluster labels from PCA (40 PCs) + 15-NN Euclidean graph + Louvain
- Rare threshold R (default rare_h=0.01 in scCAD.scCAD)
- Merge threshold parameter merge_h (default 0.3)

## Outputs

- D-cluster labels from recursive decomposition of oversized clusters
- M-cluster labels after center-distance merging (per-cell M-cluster assignment and cluster sizes written to cluster_decomposition_labels.csv)
- sub_clusters array of sub-cluster labels per cell returned by scCAD.scCAD

## How to apply

Starting from initial I-clusters (PCA to top 40 PCs, 15-nearest-neighbor Euclidean graph, Louvain), iteratively re-apply the rapid clustering module to any cluster containing more than R*n cells, with R=0.01 or 30/n when the dataset has fewer than 3000 cells, stopping when no new clusters appear or all clusters fall below R*n; these are the D-clusters. Compute each D-cluster center as the per-gene mean expression vector over its cells, then compute Euclidean distances between all cluster centers. Set the merge threshold THM to the median of each cluster's nearest-neighbor center distance and merge cluster pairs whose center distance falls below THM to obtain M-clusters. The rationale is that rare cell types are often absorbed into major clusters by one-time global clustering, so decomposition driven by the strongest differential signals exposes them while merging prevents over-fragmentation of the same population.

## Related tools

- **scCAD** (Implements the iterative cluster decomposition (clusters larger than R=1% of cells) and merging of D-clusters into M-clusters before rare cluster detection) — https://github.com/xuyp-csu/scCAD

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, rare_h=0.01, merge_h=0.3, save_path='./')
```

## Evaluation signals

- cluster_decomposition_labels.csv exists and its row count equals the number of input cells (e.g. 1556 for the 1%Jurkat demo)
- Decomposition isolates at least one small (sub-1%) M-cluster corresponding to the 16 Jurkat cells in the 1540x293T + 16 Jurkat example
- No cluster larger than R*n cells remains after the decomposition loop terminates (all clusters below R*n or no new clusters produced)
- Merged M-cluster count is not larger than the D-cluster count, and merged pairs had Euclidean center distances below THM (median of nearest-neighbor center distances)
- sub_clusters array returned by scCAD.scCAD has length equal to the number of cells

## Limitations

- Runtime and RAM scale with dataset size; scCAD requires a standard computer with enough RAM for in-memory operations and is tested only on Linux and macOS
- The decomposition threshold depends on dataset size: R=0.01 normally, but 30/n for datasets with fewer than 3000 cells, so very small datasets use a different effective cutoff
- Merging via THM (median nearest-neighbor center distance) can incorrectly merge genuinely distinct but spatially close sub-populations, and stopping criteria depend on the random seed (default 2023)

## Evidence

- [discussion] Iterative decomposition threshold R=1% of total cells: [evidence span withheld — anchor status: not_found]
- [discussion] Merge threshold THM definition: [evidence span withheld — anchor status: not_found]
- [discussion] PCA/KNN/Louvain rapid clustering module parameters: [evidence span withheld — anchor status: not_found]
