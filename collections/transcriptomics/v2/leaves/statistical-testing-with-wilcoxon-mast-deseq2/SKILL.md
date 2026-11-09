---
name: statistical-testing-with-wilcoxon-mast-deseq2
description: Use when a QC-filtered, normalized, clustered scRNA-seq object (e.g.,
  PBMC3k with Louvain labels) needs per-cluster marker genes ranked by log2 fold-change
  against all remaining cells.
when_to_use_negative:
- The dataset is from an organism other than human or mouse — the article states 'the
  analysis of only human and mouse datasets is currently supported'
- The dataset exceeds 50,000 cells on the online SCALA web server — users are advised
  to use the desktop version as some steps cannot be completed online
- The input already is a ranked marker table or lacks a normalized expression matrix
  and cluster labels — one-vs-rest differential testing cannot be performed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCALA
  role: Interactive pipeline that wraps the clustering and markers-identification
    module; developed in R, Shiny and JavaScript
- name: Seurat
  role: Provides the SNN graph construction, Louvain clustering, and the FindMarkers/FindAllMarkers
    one-vs-rest differential expression tests (Wilcoxon rank sum, MAST, DESeq2)
  repo: https://github.com/scottgigante/seurat
- name: R
  role: Implementation language in which SCALA and the Seurat-based marker testing
    run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:a43d0b9ce7f1dd3a2122b13b7e764745bcb34e98b826f83db867e5f9002729c2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/scottgigante/seurat
  tools:
  - R
  - SCALA
  - Seurat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Statistical testing with Wilcoxon/MAST/DESeq2 for cluster marker identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies marker genes characterizing each Louvain cluster of a scRNA-seq dataset by testing each cluster's cells against all remaining cells using the Wilcoxon rank sum test (with MAST and DESeq2 as alternatives) as implemented in SCALA via Seurat, as demonstrated on the PBMC3k dataset.

## When to use

Use when a QC-filtered, normalized, clustered scRNA-seq object (e.g., PBMC3k with Louvain labels) needs per-cluster marker genes ranked by log2 fold-change against all remaining cells.

## When NOT to use

- The dataset is from an organism other than human or mouse — the article states 'the analysis of only human and mouse datasets is currently supported'
- The dataset exceeds 50,000 cells on the online SCALA web server — users are advised to use the desktop version as some steps cannot be completed online
- The input already is a ranked marker table or lacks a normalized expression matrix and cluster labels — one-vs-rest differential testing cannot be performed

## Inputs

- QC-filtered, normalized, scaled Seurat scRNA-seq object with 2000 variable features and PCA coordinates (PBMC3k in the paper's use-case)
- Louvain cluster labels per cell from SNN graph partitioning of the PCA space

## Outputs

- cluster_assignments.csv — cell barcode-to-Louvain-cluster assignment table
- marker_genes_per_cluster.csv — per-cluster marker table ranked by Log2FC, top 10 markers per cluster (as in Fig. 2E)

## How to apply

Start from the QC-filtered, normalized, scaled Seurat object holding its 2000 default variable features and PCA coordinates determined by 10-fold SVD cross-validation (25 informative PCs in the paper's PBMC3k run). Build a shared-nearest-neighbour (SNN) graph from Euclidean distances in PCA space and partition it with the Louvain algorithm to obtain per-cell cluster labels. Then run cluster-specific differential expression in Seurat, testing each cluster's cells against all remaining cells using the Wilcoxon rank sum test on normalized expression values, with MAST and DESeq2 available as alternative statistical tests. Rank the resulting per-cluster markers by Log2FC and export the top 10 markers per cluster (as in Fig. 2E) together with the barcode-to-cluster assignment table.

## Related tools

- **SCALA** (Interactive pipeline that wraps the clustering and markers-identification module; developed in R, Shiny and JavaScript)
- **Seurat** (Provides the SNN graph construction, Louvain clustering, and the FindMarkers/FindAllMarkers one-vs-rest differential expression tests (Wilcoxon rank sum, MAST, DESeq2)) — https://github.com/scottgigante/seurat
- **R** (Implementation language in which SCALA and the Seurat-based marker testing run)

## Examples

```
markers <- Seurat::FindAllMarkers(pbmc, test.use = "wilcox"); write.csv(markers, "marker_genes_per_cluster.csv")
```

## Evaluation signals

- Both cluster_assignments.csv and marker_genes_per_cluster.csv exist and contain non-zero rows
- Every barcode in cluster_assignments.csv carries a Louvain cluster label
- marker_genes_per_cluster.csv contains per-cluster entries with genes ranked by log2FC
- The exported top-10 markers per cluster match the paper's per-cluster marker depiction (Fig. 2E)
- Marker tests were run one-vs-rest (each cluster against all remaining cells) on normalized expression values

## Limitations

- Only human and mouse datasets are currently supported in both modalities
- Online SCALA restricts dataset size to < 2 GB with no more than 2 CPU threads per session, and some steps cannot be completed online for datasets larger than 50,000 cells
- Software version numbers for Seurat employed in SCALA are not stated in the paper's results section

## Evidence

- [methods] SCALA identifies cluster marker genes by comparing each cluster's transcriptomes against the rest of the cells' transcriptomes through the Wilcoxon rank sum test on the normalized expression values.: [evidence span withheld — anchor status: not_found]
- [methods] Run cluster-specific differential expression in Seurat, testing each cluster's cells against all remaining cells, with the Wilcoxon rank sum test, MAST, and DESeq2.: [evidence span withheld — anchor status: not_found]
