---
name: annotation-based-cluster-composition-comparison
description: 'Use when comparing cluster purity before and after a decomposition/merging
  step: you have cell type annotations for each cell and two clusterings of the same
  cells, and need to know if rare types become more dominant within their clusters.'
when_to_use_negative:
- Input lacks ground-truth cell type annotations, so dominant types and occupy rates
  P_i cannot be computed
- You need a single rare-type detection score rather than a purity comparison between
  two clusterings of the same cells (use scCAD's independence score with threshold
  I=0.7 instead)
- The second clustering is an independent re-clustering rather than the decomposition/merging
  output of the first (M-clusters are derived from I-clusters by iterative decomposition
  and THM-based merging)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCAD
  role: Provides the I-cluster (initial Louvain) and M-cluster (post-decomposition/merge)
    assignments whose composition is compared using the annotation-based occupy rate
    metric
  repo: github.com/xuyp-csu/scCAD
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:2ed7bf8adb9d1b62e80f055faa05d6f2c6cdd966c89d217bae0ed4530470b1d3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  tools:
  - scCAD
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# annotation-based-cluster-composition-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how a cluster-decomposition and merging step (scCAD's I-clusters vs. M-clusters) changes the purity of cell-type composition by computing dominant-type occupy rates P_i = max(N_i,1,...,N_i,t)/N_i per cluster. Demonstrates that decomposition raises the average dominant-type proportion of rare cell types from 0.283 (I-clusters) to 0.704 (M-clusters).

## When to use

Use when comparing cluster purity before and after a decomposition/merging step: you have cell type annotations for each cell and two clusterings of the same cells, and need to know if rare types become more dominant within their clusters.

## When NOT to use

- Input lacks ground-truth cell type annotations, so dominant types and occupy rates P_i cannot be computed
- You need a single rare-type detection score rather than a purity comparison between two clusterings of the same cells (use scCAD's independence score with threshold I=0.7 instead)
- The second clustering is an independent re-clustering rather than the decomposition/merging output of the first (M-clusters are derived from I-clusters by iterative decomposition and THM-based merging)

## Inputs

- scRNA-seq expression matrices (cells x genes) for the 25 benchmark datasets with original cell type annotations from source studies
- I-cluster labels: initial Louvain cluster assignment per cell (PCA 40 PCs, KNN k=15)
- M-cluster labels: post-decomposition/merging cluster assignment per cell (R=1%, THM=median nearest-neighbor center distance)

## Outputs

- Per-cluster dominant-type occupy rates P_i for I-clusters and M-clusters
- Average dominant-type proportion over all cell types and over rare cell types, per dataset and clustering stage (I-clusters vs. M-clusters), replicating Supplementary Table 9
- p_t,i and q_t,i statistics for the top-ten clusters at both stages, with medians of q_t.i and mean p_t,i comparable to Supplementary Fig. 3b

## How to apply

Load each scRNA-seq dataset with its original cell type annotations, preprocess per scCAD (retain genes expressed in at least 3 cells, log-normalization with cell-specific size factors and log2 transform after pseudo-count 1), and run initial clustering (PCA to 40 PCs, KNN graph with k=15, Louvain) to obtain I-clusters. Iteratively decompose clusters larger than R*n (R=1%, or 30/n when n<3000) and merge clusters whose center distance falls below THM=median of nearest-neighbor distances to obtain M-clusters. For every I-cluster and M-cluster, identify the dominant cell type as the type with the highest cell count and compute its occupy rate P_i = max(N_i,1,...,N_i,t)/N_i. Average the dominant-type proportions over all cell types and over rare cell types separately for both clustering stages, per dataset. Optionally compute p_t,i (fraction of rare-type cells in cluster i) and q_t,i (fraction of cluster i occupied by rare type t) for top-ten clusters at both stages to compare medians of q_t,i and mean p_t,i.

## Related tools

- **scCAD** (Provides the I-cluster (initial Louvain) and M-cluster (post-decomposition/merge) assignments whose composition is compared using the annotation-based occupy rate metric) — github.com/xuyp-csu/scCAD

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- The average rare-cell-type proportion in dominant clusters rises from 0.283 (I-clusters) to 0.704 (M-clusters), matching Supplementary Table 9 and Supplementary Fig. 3b
- Median q_t,i (fraction of cluster i occupied by rare type t) for top-ten clusters shifts from 18% (I-Clusters) to 81% (M-Clusters)
- Mean p_t,i (fraction of rare-type cells in cluster i) for top-ten clusters shifts from 88% to 79% between stages
- Every P_i must lie in (0, 1] and equal 1 for a cluster whose cells all share one annotated type
- I-cluster and M-cluster partitions cover the identical set of cells, so per-cluster cell counts N_i sum to the same total at both stages

## Limitations

- Requires original-study cell type annotations catalogued in Supplementary Table 1; cannot be computed on unannotated data
- Depends on scCAD decomposition parameters (R=1% size threshold, THM=median nearest-neighbor center distance); different thresholds change the M-cluster partition and hence the occupy rates
- No statistical pre-determination of sample size and no randomization for most experiments, per the authors' reporting statement

## Evidence

- [discussion] iteratively decomposes each cluster containing more than R=1% of the total number of cells: [evidence span withheld — anchor status: not_found]
- [discussion] applies PCA to obtain the top 40 principal components ... nearest neighbor parameter k set to 15: [evidence span withheld — anchor status: not_found]
- [supplementary] After cluster decomposition and merging, the average proportion of cell types within their dominant clusters significantly increased, especially for rare types, rising from 0.283 (I-clusters) to 0.704 (M-clusters).: [evidence span withheld — anchor status: not_found]
- [readme] Cluster decomposition-based Anomaly Detection method (scCAD) is used to effectively identify rare cell types in scRNA-seq data.: "Cluster decomposition-based Anomaly Detection method (scCAD) is used to effectively identify rare cell types in scRNA-seq data."
