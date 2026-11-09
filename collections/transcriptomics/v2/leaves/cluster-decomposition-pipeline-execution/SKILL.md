---
name: cluster-decomposition-pipeline-execution
description: Use when analyzing a scRNA-seq expression matrix to identify rare cell
  subpopulations that one-time clustering (PCA + Louvain) may overlook, and you need
  decomposed/merged sub-clusters with anomaly scores.
when_to_use_negative:
- When the goal is comprehensive identification of all major cell types via standard
  one-time clustering (e.g. Seurat-style PCA + Louvain) rather than rare cell identification
- When the input is already a feature/embedding table or cell-level metadata rather
  than a cell-by-gene expression matrix, since scCAD requires raw gene expression
  for feature selection and decomposition
- When data cannot be preprocessed per scCAD (genes expressed in at least three cells
  retained, log-normalization with cell-specific size factors), e.g. non-expression
  modalities
edam_operation: http://edamontology.org/operation_3432
tools:
- name: scCAD
  role: 'Primary implementation: iteratively decomposes clusters based on the most
    differential signals, merges sub-clusters, and computes anomaly/independence scores
    to flag rare clusters'
  repo: https://github.com/xuyp-csu/scCAD
- name: Scanpy
  role: Dependency (version 1.9.1) used within scCAD's preprocessing and clustering
    pipeline (PCA, KNN graph, Louvain)
- name: python-louvain
  role: Dependency (version 0.16) providing Louvain community detection for initial
    cluster assignment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:2adef27c263a1968ab98678ee4f9fb6e32477cdfd3dcae53cf617d30ea9dd8e4
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cluster-decomposition-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs scCAD's iterative cluster decomposition and merging pipeline to separate rare cell types in scRNA-seq data: clusters larger than R*n (R=1%, or 30/n when n<3000) are iteratively decomposed on the most differential signals, then decomposed clusters whose center distance is below THM (median nearest-neighbor distance) are merged.

## When to use

Use when analyzing a scRNA-seq expression matrix to identify rare cell subpopulations that one-time clustering (PCA + Louvain) may overlook, and you need decomposed/merged sub-clusters with anomaly scores.

## When NOT to use

- When the goal is comprehensive identification of all major cell types via standard one-time clustering (e.g. Seurat-style PCA + Louvain) rather than rare cell identification
- When the input is already a feature/embedding table or cell-level metadata rather than a cell-by-gene expression matrix, since scCAD requires raw gene expression for feature selection and decomposition
- When data cannot be preprocessed per scCAD (genes expressed in at least three cells retained, log-normalization with cell-specific size factors), e.g. non-expression modalities

## Inputs

- Raw or normalized scRNA-seq count/expression matrix with cells in rows and genes in columns (e.g. loaded from an .h5 file such as 1%Jurkat.h5)
- Optional cell names, gene names, and dataset name arrays
- Optional cell type annotations for downstream validation of identified sub-clusters

## Outputs

- result: list of rare sub-clusters (cell name lists) identified by scCAD
- score: np.array of anomaly scores for every sub-cluster
- sub_clusters: np.array[n cells] of sub-cluster label assignments
- degs_list: list of differentially expressed genes used for each rare sub-cluster

## How to apply

Preprocess by retaining genes expressed in at least three cells, then apply log-normalization with cell-specific size factors and log2 transform after a pseudo-count of 1 (the scCAD 'normalization=True' default handles this). Run initial clustering via PCA to the top 40 principal components, a KNN graph with k=15, and Louvain clustering to obtain I-clusters. Iteratively decompose each cluster containing more than R=1% of total cells (rare_h=0.01; or 30/n when n<3000) based on the most differential signals, then merge decomposed clusters whose center distance falls below THM=median(d1, d2, ..., dm) (merge_h default 0.3 in the implementation). On the resulting sub-clusters, select the top 20 genes with the largest differences, feed them into an isolation forest model for anomaly scores, and flag clusters with independence score I above the default 0.7 as rare (overlap_h=0.7). Provide the expression matrix with cells in rows and genes in columns; scCAD returns the rare sub-clusters, per-sub-cluster scores, sub-cluster label assignments, and the differential gene lists.

## Related tools

- **scCAD** (Primary implementation: iteratively decomposes clusters based on the most differential signals, merges sub-clusters, and computes anomaly/independence scores to flag rare clusters) — https://github.com/xuyp-csu/scCAD
- **Scanpy** (Dependency (version 1.9.1) used within scCAD's preprocessing and clustering pipeline (PCA, KNN graph, Louvain))
- **python-louvain** (Dependency (version 0.16) providing Louvain community detection for initial cluster assignment)

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- Every cluster larger than R=1% of total cells (or 30/n when n<3000) is decomposed before merging; verify no oversized cluster remains in the final M-clusters
- Flagged rare clusters must have independence score exceeding the default threshold I=0.7
- On the demo 1%Jurkat.h5 dataset (1540 293T cells + 16 Jurkat cells), execution completes in approximately 40 seconds; sub-cluster label assignments must cover all n cells
- Ensemble feature selection must yield top 2000 variable genes plus top 2000 random-forest-importance genes, with the top 20 differential genes per cluster feeding the isolation forest
- Rare cell type-specific genes should account for over 86% on average among selected genes; identified sub-clusters should show enrichment of the rare type when cross-tabulated against labels (Counter(labels[indices]))

## Limitations

- Only clusters larger than R*n are decomposed; rare cell types that form clusters below this size threshold at the initial stage may still be overlooked
- The independence score threshold I=0.7 is a fixed default applied across all datasets and may require adjustment
- Tested only on Linux (Ubuntu 16.04.7) and macOS (Monterey 12.6.3) with Python 3.7.13 and pinned dependency versions; memory requirements scale with the in-memory expression matrix
- The method identifies candidate rare clusters for annotation review — it may need follow-up to correct or confirm annotations (e.g. in the ccRCC dataset scCAD rectified an annotation)

## Evidence

- [abstract] Iteratively decomposes clusters based on the most differential signals in each cluster: "which iteratively decomposes clusters based on the most differential signals in each cluster to effectively separate rare cell types and achieve accurate identification"
- [methods] Cluster decomposition threshold R=1%: [evidence span withheld — anchor status: not_found]
- [methods] Merge threshold THM = median nearest-neighbor distance: [evidence span withheld — anchor status: not_found]
- [discussion] Independence score threshold I=0.7: [evidence span withheld — anchor status: not_found]
- [readme] scCAD function signature and returned values: "result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')"
- [discussion] Preprocessing: retain genes in at least three cells: "genes that are expressed in at least three cells are retained for downstream analysis"
