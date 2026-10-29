---
name: rare-cell-type-cluster-purity-analysis
description: Use when evaluating whether an iterative cluster decomposition/merging
  step improves the purity (occupy rate) of rare cell types in their dominant clusters,
  as in scCAD's I-cluster vs. M-cluster comparison (Supplementary Table 9).
when_to_use_negative:
- Input datasets lack cell type annotations, so no ground-truth dominant type or occupy
  rate can be computed per cluster
- The goal is anomaly/rare-type detection itself rather than evaluation of cluster
  purity — use scCAD's independence score (threshold I=0.7) workflow instead
- Clustering uses a one-shot method without an I-cluster vs. M-cluster decomposition
  stage, making the before/after comparison undefined
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scCAD
  role: Executes initial Louvain clustering, iterative cluster decomposition (rare_h=0.01),
    and cluster merging (merge_h=0.3) to produce M-clusters whose purity is compared
    against I-clusters
  repo: github.com/xuyp-csu/scCAD
- name: Scanpy
  role: Dependency (v1.9.1) used within scCAD for preprocessing and graph-based clustering
    steps (PCA, KNN, Louvain) that generate the I-clusters
  repo: github.com/xuyp-csu/scCAD
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:2b1c2e7d419dd29e6bd0e5845c857a9d229d9059b567ffe6a0defdea00e33a3f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  tools:
  - Scanpy
  - scCAD
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rare-cell-type-cluster-purity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies whether scCAD's cluster decomposition and merging step enriches rare cell types within their dominant clusters, by comparing dominant-type occupy rates (P_i) of initial (I-) clusters vs. merged (M-) clusters across the 25 benchmark scRNA-seq datasets.

## When to use

Use when evaluating whether an iterative cluster decomposition/merging step improves the purity (occupy rate) of rare cell types in their dominant clusters, as in scCAD's I-cluster vs. M-cluster comparison (Supplementary Table 9).

## When NOT to use

- Input datasets lack cell type annotations, so no ground-truth dominant type or occupy rate can be computed per cluster
- The goal is anomaly/rare-type detection itself rather than evaluation of cluster purity — use scCAD's independence score (threshold I=0.7) workflow instead
- Clustering uses a one-shot method without an I-cluster vs. M-cluster decomposition stage, making the before/after comparison undefined

## Inputs

- scRNA-seq expression matrices (cells x genes) from the 25 benchmark datasets with cell type annotations from the original studies (Supplementary Table 1)
- I-cluster labels from PCA (40 PCs) + KNN graph (k=15) + Louvain clustering
- M-cluster labels from scCAD's iterative decomposition (rare_h=0.01) and merging (THM = median nearest-neighbor center distance)

## Outputs

- Per-cluster dominant cell type and occupy rate P_i for I-clusters and M-clusters
- Average dominant-type proportions over all and rare cell types per dataset (Supplementary Table 9 comparison: 0.283 I-clusters vs 0.704 M-clusters)
- Top-ten-cluster p_t,i and q_t,i statistics (median q_t,i = 18% I vs 81% M; mean p_t,i = 88% vs 79%; Supplementary Fig. 3b)

## How to apply

For each of the 25 benchmark datasets, preprocess per scCAD (retain genes expressed in at least 3 cells; log-normalization with cell-specific size factors and log2 transform after pseudo-count 1), then run PCA to 40 PCs, build a KNN graph with k=15, and apply Louvain clustering to obtain I-clusters. Iteratively decompose clusters larger than R*n (R=1%, or 30/n when n<3000) and merge clusters whose center distance is below THM (median of nearest-neighbor center distances) to obtain M-clusters. For every I-cluster and M-cluster, identify the dominant cell type as the type with the highest cell count and compute the occupy rate P_i = max(N_i,1,...,N_i,t)/N_i. Average dominant-type proportions over all cell types and over rare cell types separately per stage, replicating Supplementary Table 9; additionally compute p_t,i (fraction of rare-type cells in cluster i) and q_t,i (fraction of cluster i occupied by rare type t) for top-ten clusters, checking medians of q_t,i (18% I vs. 81% M) and mean p_t,i (88% vs 79%). Accept when the average rare-type proportion in dominant clusters rises from 0.283 (I-clusters) to 0.704 (M-clusters), matching Supplementary Table 9 and Supplementary Fig. 3b.

## Related tools

- **scCAD** (Executes initial Louvain clustering, iterative cluster decomposition (rare_h=0.01), and cluster merging (merge_h=0.3) to produce M-clusters whose purity is compared against I-clusters) — github.com/xuyp-csu/scCAD
- **Scanpy** (Dependency (v1.9.1) used within scCAD for preprocessing and graph-based clustering steps (PCA, KNN, Louvain) that generate the I-clusters) — github.com/xuyp-csu/scCAD

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, rare_h=0.01, merge_h=0.3, save_path='./')
```

## Evaluation signals

- Average rare-type dominant-cluster proportion rises from 0.283 (I-clusters) to 0.704 (M-clusters), matching Supplementary Table 9 and Supplementary Fig. 3b
- Median q_t,i across top-ten clusters is ~18% for I-clusters and ~81% for M-clusters
- Mean p_t,i across top-ten clusters is ~88% (M-clusters) vs 79% (I-clusters)
- Decomposition applies only to clusters larger than R=1% of cells (R=30/n when n<3000), and merging uses THM = median of nearest-neighbor center distances
- Analysis spans all 25 benchmark datasets, so per-dataset averages should reproduce the reported increase for rare types

## Limitations

- The occupy-rate comparison requires ground-truth cell type annotations; it is an evaluation metric, not a detection method
- Decomposition of large clusters (R=1%) and merge threshold THM=median(d1, d2, …, dm) are dataset-size-dependent; for datasets with n<3000 the size cutoff changes to 30/n
- Supplementary Table 9 and Supplementary Fig. 3b are not reproduced in the provided context, so exact per-dataset values cannot be verified from this document alone

## Evidence

- [discussion] Iterative cluster decomposition of clusters larger than R*n: [evidence span withheld — anchor status: not_found]
- [discussion] Merge threshold THM definition: [evidence span withheld — anchor status: not_found]
- [discussion] Initial clustering parameters: [evidence span withheld — anchor status: not_found]
- [discussion] Independence score threshold for rare cluster calling: [evidence span withheld — anchor status: not_found]
