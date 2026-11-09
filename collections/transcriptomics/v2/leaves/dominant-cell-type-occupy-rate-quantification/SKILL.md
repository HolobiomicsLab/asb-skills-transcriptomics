---
name: dominant-cell-type-occupy-rate-quantification
description: 'Use when auditing whether an iterative cluster decomposition/merging
  pipeline (e.g., scCAD) increased the purity of rare cell type assignment: you have
  cell-type annotations plus I-cluster and M-cluster labels and need dominant-type
  occupy rates per cluster.'
when_to_use_negative:
- When only one clustering partition is available with no decomposed/merged counterpart
  — the metric requires comparing I-clusters against M-clusters
- When the goal is calling which clusters are rare (use scCAD's independence score
  threshold I=0.7 instead); this skill only quantifies purity of already-assigned
  clusters
- When cell-type ground-truth annotations are absent, since P_i, p_t,i, and q_t,i
  all require per-cell type labels
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCAD
  role: Produces the I-clusters and M-clusters (via iterative decomposition with rare_h=0.01
    and merging with merge_h) whose dominant-type occupy rates are quantified
  repo: github.com/xuyp-csu/scCAD
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:95a235d49939413d6e2459da3018647b4a6fc0d96ec8a1c16b86108f28caeb01
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dominant-cell-type-occupy-rate-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies cluster purity by computing, for each cluster, the occupy rate P_i = max(N_i,1,...,N_i,t)/N_i of its dominant cell type, and compares these proportions between initial (I-clusters, Louvain on 40 PCs) and decomposed-and-merged (M-clusters) partitions to verify that scCAD's cluster decomposition enriches rare cell types in their dominant clusters.

## When to use

Use when auditing whether an iterative cluster decomposition/merging pipeline (e.g., scCAD) increased the purity of rare cell type assignment: you have cell-type annotations plus I-cluster and M-cluster labels and need dominant-type occupy rates per cluster.

## When NOT to use

- When only one clustering partition is available with no decomposed/merged counterpart — the metric requires comparing I-clusters against M-clusters
- When the goal is calling which clusters are rare (use scCAD's independence score threshold I=0.7 instead); this skill only quantifies purity of already-assigned clusters
- When cell-type ground-truth annotations are absent, since P_i, p_t,i, and q_t,i all require per-cell type labels

## Inputs

- scRNA-seq count matrix (cells x genes, e.g., .h5 with 'X', 'Y', 'gn', 'cn' arrays) with cell-type annotations from the original study
- I-cluster labels from PCA(40 PCs) + KNN(k=15) + Louvain initial clustering
- M-cluster labels after cluster decomposition (rare_h/size threshold R=1%) and merging (THM = median nearest-neighbor center distance)

## Outputs

- Per-cluster dominant-type occupy rates P_i for I-clusters and M-clusters
- Per-dataset average dominant-type proportions over all cell types and over rare cell types (I-cluster vs M-cluster, e.g., 0.283 -> 0.704)
- p_t,i (fraction of rare-type cells in cluster i) and q_t,i (fraction of cluster i occupied by rare type t) for the top-ten clusters at both stages
- Comparison table replicating Supplementary Table 9

## How to apply

For every I-cluster and M-cluster, identify the dominant cell type as the type with the highest cell count and compute P_i = max(N_i,1,...,N_i,t)/N_i; then average dominant-type proportions over all cell types and over rare cell types separately, per dataset. Complement this with p_t,i (fraction of rare-type cells in cluster i) and q_t,i (fraction of cluster i occupied by rare type t) for the top-ten clusters at both stages. The I-clusters come from PCA to 40 PCs, a KNN graph with k=15, and Louvain; the M-clusters come after iteratively decomposing clusters larger than R=1% of total cells (R=30/n when n<3000) and merging clusters whose center distance is below THM, the median of nearest-neighbor center distances. Accept the result when the average rare-type dominant-cluster proportion rises from 0.283 (I-clusters) to 0.704 (M-clusters), matching Supplementary Table 9 and Supplementary Fig. 3b, with q_t,i medians of 18% (I-clusters) vs 81% (M-clusters) and mean p_t,i of 88% vs 79%.

## Related tools

- **scCAD** (Produces the I-clusters and M-clusters (via iterative decomposition with rare_h=0.01 and merging with merge_h) whose dominant-type occupy rates are quantified) — github.com/xuyp-csu/scCAD

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- Average rare-cell-type dominant-cluster proportion increases from 0.283 (I-clusters) to 0.704 (M-clusters), matching Supplementary Table 9 and Supplementary Fig. 3b
- Median q_t,i across top-ten clusters shifts from 18% (I-clusters) to 81% (M-clusters)
- Mean p_t,i across top-ten clusters is 88% (I-clusters) vs 79% (M-clusters)
- Sanity check: for each cluster, P_i equals the count of the most frequent cell type divided by the cluster's total cell count N_i
- Decomposition threshold applied: clusters with more than R=1% of total cells (R=30/n when n<3000) are iteratively decomposed before merging at THM

## Limitations

- Results depend on the decomposition size threshold (R=1%, or 30/n when n<3000) and merge threshold THM (median of nearest-neighbor center distances); non-default parameters change the M-cluster partition
- The occupy-rate comparison requires gold-standard cell-type annotations from the original studies (catalogued in Supplementary Table 1), which are not always available
- Supplementary Table 9 and Supplementary Fig. 3b, the reference benchmarks for this analysis, are not included in the main document body

## Evidence

- [results] average rare-type proportion in dominant clusters rises from 0.283 (I-clusters) to 0.704 (M-clusters): [evidence span withheld — anchor status: not_found]
- [methods] iterative decomposition threshold: [evidence span withheld — anchor status: not_found]
- [methods] merging threshold THM definition: [evidence span withheld — anchor status: not_found]
- [discussion] independence score threshold for rare calls (contrast with purity quantification): [evidence span withheld — anchor status: not_found]
