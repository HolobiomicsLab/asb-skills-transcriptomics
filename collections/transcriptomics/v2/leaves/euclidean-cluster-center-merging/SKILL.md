---
name: euclidean-cluster-center-merging
description: Use when an iterative cluster decomposition (as in scCAD) has over-split
  major clusters into D-clusters and you must consolidate them into stable M-clusters
  before scoring rare sub-clusters.
when_to_use_negative:
- When clusters have not first been decomposed via the iterative R*n rule — merging
  raw Louvain I-clusters directly is a different operation
- When cell type populations are already well separated and no over-splitting is suspected;
  an aggressive merge threshold can fuse genuinely distinct rare subtypes into major
  clusters before the independence-score (I=0.7) rare call is made
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: scCAD
  role: Implements cluster decomposition and the center-based merging step (merge_h
    parameter) that produces M-clusters prior to rare cluster detection
  repo: https://github.com/xuyp-csu/scCAD
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:491aaa15d569a51d0b17fe57a63eed4db5fbc2a65bea0e116c21fee6e85ad80e
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
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Euclidean cluster center merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Post-decomposition step of scCAD that merges over-split decomposed D-clusters into M-clusters by computing per-gene mean expression centers, pairwise Euclidean distances between centers, and merging pairs closer than a median-based threshold THM before rare cluster detection.

## When to use

Use when an iterative cluster decomposition (as in scCAD) has over-split major clusters into D-clusters and you must consolidate them into stable M-clusters before scoring rare sub-clusters.

## When NOT to use

- When clusters have not first been decomposed via the iterative R*n rule — merging raw Louvain I-clusters directly is a different operation
- When cell type populations are already well separated and no over-splitting is suspected; an aggressive merge threshold can fuse genuinely distinct rare subtypes into major clusters before the independence-score (I=0.7) rare call is made

## Inputs

- D-cluster assignments from scCAD's iterative rapid clustering module (PCA 40 PCs, KNN k=15, Louvain)
- log-normalized gene expression matrix (genes retained if expressed in at least three cells; cell-specific size factors; log2(x+1))
- merge_h threshold (scCAD default 0.3)

## Outputs

- M-cluster label assignment per cell (e.g. cluster_decomposition_labels.csv with per-cell M-cluster labels and cluster sizes)
- Merged cluster set (M-clusters) fed into rare cluster detection

## How to apply

After iterative decomposition (PCA top 40 PCs, 15-NN Euclidean graph, Louvain with R=0.01 or 30/n for small datasets) yields D-clusters, compute each D-cluster's center as the per-gene mean expression vector over its cells. Compute Euclidean distances between all cluster centers, then set the merge threshold THM to the median of each cluster's nearest-neighbor center distance (THM=median(d1, d2,…, dm)); the scCAD API exposes this as merge_h with default 0.3. Merge any cluster pair whose center distance falls below THM to obtain M-clusters, which are then carried into differential gene selection and isolation-forest anomaly scoring. Rationale: decomposition can fragment a coherent population, and center proximity in the log-normalized expression space (cell-specific size factors, log2(x+1)) indicates the split was spurious rather than a true rare subtype.

## Related tools

- **scCAD** (Implements cluster decomposition and the center-based merging step (merge_h parameter) that produces M-clusters prior to rare cluster detection) — https://github.com/xuyp-csu/scCAD

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./', merge_h=0.3)
```

## Evaluation signals

- Per-cell output file (cluster_decomposition_labels.csv) has one row per input cell (e.g. 1556 for the 1%Jurkat example)
- Every merged pair had cluster-center Euclidean distance below THM, and THM equals the median of the D-clusters' nearest-neighbor center distances
- On the 1%Jurkat.h5 benchmark, the merged decomposition still isolates at least one small (sub-1%) M-cluster corresponding to the 16 Jurkat cells among 1540 293T cells
- Cluster centers are computed on the log-normalized matrix (size factors + log2(x+1)), not raw counts

## Limitations

- Threshold choice matters: merge_h default is 0.3 in the scCAD API, while the paper's THM is data-adaptive (median of nearest-neighbor center distances); an unsuitable threshold can either fail to repair over-splitting or fuse true rare subtypes
- Merging is based solely on mean-expression centers, so shape/overlap information of cluster distributions is not used at this step

## Evidence

- [discussion] THM definition: [evidence span withheld — anchor status: not_found]
