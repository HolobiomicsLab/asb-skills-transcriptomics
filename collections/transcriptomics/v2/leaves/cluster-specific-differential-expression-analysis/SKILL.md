---
name: cluster-specific-differential-expression-analysis
description: Use when an scRNA-seq dataset has already been QC-filtered, normalized,
  scaled, and partitioned into Louvain clusters (e.g. PBMC3k via Seurat/SCALA), and
  per-cluster marker genes ranked by log2FC are needed.
when_to_use_negative:
- Input is raw counts that have not been QC-filtered, normalized and scaled, or cells
  are not yet assigned Louvain cluster labels — run QC, normalization, PCA/SVD and
  clustering first
- The goal is cell-cell communication or ligand-receptor inference rather than per-cluster
  marker discovery — use a dedicated framework such as NicheNet instead
- Analysis is on organisms other than human or mouse — SCALA currently supports only
  human and mouse datasets
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCALA
  role: 'GUI pipeline that exposes the markers'' identification module: one-vs-rest
    cluster DE testing on Louvain-clustered scRNA-seq data'
- name: Seurat
  role: Engine used to build the SNN graph from PCA-space Euclidean distances and
    to run the Wilcoxon rank sum, MAST and DESeq2 cluster marker tests
  repo: https://github.com/scottgigante/seurat
- name: R
  role: Language in which SCALA and the clustering/markers module are developed
- name: presto
  role: Fast Wilcoxon rank-sum implementation for marker discovery in single-cell
    data, referenced among SCALA's integrated packages
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:6aeca3d0f8a1d92bc07aec4894183c3185bd5f44651902c6ca330160c4d9c144
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
  - presto
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- clustering-independent-differential-expression-analysis
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Cluster-specific differential expression analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies marker genes characterizing each Louvain cluster of an scRNA-seq dataset by testing each cluster's cells against all remaining cells using the Wilcoxon rank sum test on normalized expression (with MAST and DESeq2 available in Seurat), as implemented in SCALA.

## When to use

Use when an scRNA-seq dataset has already been QC-filtered, normalized, scaled, and partitioned into Louvain clusters (e.g. PBMC3k via Seurat/SCALA), and per-cluster marker genes ranked by log2FC are needed.

## When NOT to use

- Input is raw counts that have not been QC-filtered, normalized and scaled, or cells are not yet assigned Louvain cluster labels — run QC, normalization, PCA/SVD and clustering first
- The goal is cell-cell communication or ligand-receptor inference rather than per-cluster marker discovery — use a dedicated framework such as NicheNet instead
- Analysis is on organisms other than human or mouse — SCALA currently supports only human and mouse datasets

## Inputs

- QC-filtered, normalized, scaled scRNA-seq Seurat object with variable features (default 2000) and PCA coordinates
- SNN graph partitioned into Louvain cluster labels per cell (e.g. PBMC3k scRNA-seq dataset)

## Outputs

- cluster_assignments.csv — cell barcode-to-Louvain-cluster assignment table
- marker_genes_per_cluster.csv — per-cluster marker table ranked by Log2FC (top 10 markers per cluster as in Fig. 2E)

## How to apply

Load the QC-filtered, normalized, scaled object with its variable features (default 2000) and PCA coordinates (here determined by 10-fold SVD cross-validation, yielding 25 informative PCs in the PBMC3k use-case). Embed cells in a shared-nearest-neighbour (SNN) graph from Euclidean distances in PCA space, then partition it with the Louvain algorithm to obtain per-cell cluster labels. Run cluster-specific differential expression in Seurat, testing each cluster's cells against all remaining cells (one-vs-rest); SCALA applies the Wilcoxon rank sum test on normalized expression values, with MAST and DESeq2 as alternative test modalities. Rank resulting markers per cluster by Log2FC and retain the top markers (top 10 per cluster, matching Fig. 2E of the paper). Export the cell barcode-to-cluster table as cluster_assignments.csv and the per-cluster marker table as marker_genes_per_cluster.csv.

## Related tools

- **SCALA** (GUI pipeline that exposes the markers' identification module: one-vs-rest cluster DE testing on Louvain-clustered scRNA-seq data)
- **Seurat** (Engine used to build the SNN graph from PCA-space Euclidean distances and to run the Wilcoxon rank sum, MAST and DESeq2 cluster marker tests) — https://github.com/scottgigante/seurat
- **R** (Language in which SCALA and the clustering/markers module are developed)
- **presto** (Fast Wilcoxon rank-sum implementation for marker discovery in single-cell data, referenced among SCALA's integrated packages) — https://github.com/immunogenomics/presto

## Examples

```
markers <- FindMarkers(pbmc, ident.1 = "0", test.use = "wilcox")
```

## Evaluation signals

- Both cluster_assignments.csv and marker_genes_per_cluster.csv exist and contain non-zero rows
- Every barcode in cluster_assignments.csv carries a Louvain cluster label
- marker_genes_per_cluster.csv contains per-cluster entries with log2FC-ranked genes matching the paper's per-cluster marker depiction (Fig. 2E)
- Marker tests are one-vs-rest: each cluster's cells are compared against all remaining cells' transcriptomes on normalized expression values
- The chosen statistical test (Wilcoxon rank sum by default in SCALA; MAST or DESeq2 selectable) is recorded and applied consistently across clusters

## Limitations

- The online SCALA version restricts dataset size to < 2 GB and 2 CPU threads per session; for datasets larger than 50,000 cells some steps cannot be completed online and the desktop version is recommended
- Software version numbers for Seurat and ArchR employed in SCALA are not stated, so reproducing exact test behaviour may require matching versions
- Analysis of only human and mouse datasets is currently supported

## Evidence

- [results] SCALA identifies cluster marker genes by comparing each cluster's transcriptomes against the rest of the cells' transcriptomes through the Wilcoxon rank sum test on the normalized expression values.: [evidence span withheld — anchor status: not_found]
- [methods] Run cluster-specific differential expression in Seurat, testing each cluster's cells against all remaining cells, with the Wilcoxon rank sum test, MAST, and DESeq2.: [evidence span withheld — anchor status: not_found]
- [methods] Partition the SNN graph into highly interconnected communities with the Louvain algorithm to obtain cluster labels per cell.: [evidence span withheld — anchor status: not_found]
- [readme] wilcoxauc() — Wilcoxon rank-sum test and area-under-the-ROC across groups, fast enough to run on whole-genome × hundred-thousand-cell matrices in seconds.: [evidence span withheld — anchor status: not_found]
