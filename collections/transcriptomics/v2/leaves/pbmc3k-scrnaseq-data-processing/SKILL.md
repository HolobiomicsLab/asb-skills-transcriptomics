---
name: pbmc3k-scrnaseq-data-processing
description: Use when a QC-filtered PBMC3k-style scRNA-seq count matrix must be processed
  in SCALA/Seurat through normalization, dimensionality reduction, Louvain clustering,
  and per-cluster marker-gene testing with Wilcoxon/MAST/DESeq2.
when_to_use_negative:
- Input is scATAC-seq data — SCALA uses a separate scATAC-seq pipeline (e.g., LSI
  dimensionality reduction, ArchR-based BMMC workflow), not the PBMC3k scRNA-seq processing
  route
- Input is a bulk RNA-seq or precomputed feature table rather than per-cell scRNA-seq
  counts; cluster-level one-vs-all Wilcoxon/MAST/DESeq2 testing does not apply
- Dataset exceeds 50,000 cells on the online SCALA version — the article recommends
  the desktop version as some steps cannot be completed online
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCALA
  role: End-to-end R/Shiny/JavaScript pipeline in which QC, normalization, clustering,
    and marker identification are orchestrated for the PBMC3k use-case
- name: Seurat
  role: Builds the SNN graph from PCA-space Euclidean distances, runs Louvain partitioning,
    and executes one-vs-all marker tests (Wilcoxon rank sum, MAST, DESeq2)
- name: presto
  role: Fast Wilcoxon rank-sum differential expression / marker discovery engine referenced
    by the article for single-cell marker testing
  repo: https://github.com/immunogenomics/presto
- name: R
  role: Implementation language of SCALA and the surrounding Seurat-based marker-identification
    workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:697c0c715adb1bb0283f237b749cd8714ada05e65e3ea283debc17fc1249da6c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/immunogenomics/presto
  tools:
  - R
  - SCALA
  - Seurat
  - presto
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pbmc3k-scrnaseq-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces SCALA's scRNA-seq processing pipeline on the PBMC3k dataset: QC filtering, normalization and scaling, variable-feature detection, SVD cross-validated PCA, Seurat SNN-graph Louvain clustering, and per-cluster marker identification with the Wilcoxon rank sum test (plus MAST/DESeq2 alternatives).

## When to use

Use when a QC-filtered PBMC3k-style scRNA-seq count matrix must be processed in SCALA/Seurat through normalization, dimensionality reduction, Louvain clustering, and per-cluster marker-gene testing with Wilcoxon/MAST/DESeq2.

## When NOT to use

- Input is scATAC-seq data — SCALA uses a separate scATAC-seq pipeline (e.g., LSI dimensionality reduction, ArchR-based BMMC workflow), not the PBMC3k scRNA-seq processing route
- Input is a bulk RNA-seq or precomputed feature table rather than per-cell scRNA-seq counts; cluster-level one-vs-all Wilcoxon/MAST/DESeq2 testing does not apply
- Dataset exceeds 50,000 cells on the online SCALA version — the article recommends the desktop version as some steps cannot be completed online

## Inputs

- QC-filtered, normalized, scaled PBMC3k scRNA-seq expression matrix (e.g., Seurat object) with variable features and PCA coordinates
- Cell barcode metadata for the analyzed PBMC3k cells

## Outputs

- cluster_assignments.csv: cell barcode-to-Louvain-cluster assignment table
- marker_genes_per_cluster.csv: per-cluster marker genes ranked by Log2FC, top 10 markers per cluster (Fig. 2E style)

## How to apply

Load the QC-filtered, normalized, and scaled PBMC3k scRNA-seq object with its variable features (default 2000; 1535 were identified in the paper's run) and PCA coordinates determined by 10-fold SVD cross-validation (which selected 25 PCs). Build a shared-nearest-neighbour (SNN) graph on Euclidean distances in PCA space using Seurat, then partition it with the Louvain algorithm to assign a cluster label to every cell. For markers, run cluster-specific differential expression in Seurat, testing each cluster's cells against all remaining cells, with the Wilcoxon rank sum test on normalized expression values as SCALA's primary test, with MAST and DESeq2 available as alternatives. Export the barcode-to-cluster table as cluster_assignments.csv and the per-cluster marker table ranked by Log2FC (top 10 markers per cluster, as in Fig. 2E) as marker_genes_per_cluster.csv.

## Related tools

- **SCALA** (End-to-end R/Shiny/JavaScript pipeline in which QC, normalization, clustering, and marker identification are orchestrated for the PBMC3k use-case)
- **Seurat** (Builds the SNN graph from PCA-space Euclidean distances, runs Louvain partitioning, and executes one-vs-all marker tests (Wilcoxon rank sum, MAST, DESeq2))
- **presto** (Fast Wilcoxon rank-sum differential expression / marker discovery engine referenced by the article for single-cell marker testing) — https://github.com/immunogenomics/presto
- **R** (Implementation language of SCALA and the surrounding Seurat-based marker-identification workflow)

## Examples

```
pbmc <- FindClusters(pbmc); markers <- FindAllMarkers(pbmc, test.use = "wilcox")
```

## Evaluation signals

- cluster_assignments.csv exists with non-zero rows and every barcode carries a Louvain cluster label
- marker_genes_per_cluster.csv contains per-cluster entries with log2FC-ranked genes, top 10 markers per cluster, matching the paper's Fig. 2E depiction
- Variable-feature detection yields a comparable number of variable genes (the paper's run identified 1535 from a default of 2000 features)
- SVD k-fold cross-validation resolves a defined number of informative principal components (25 PCs in the paper)
- Marker testing is performed one cluster vs. all remaining cells on normalized expression values, not between arbitrary cluster pairs

## Limitations

- In both modalities, only human and mouse datasets are currently supported
- Online SCALA version restricts dataset size to < 2 GB and allocates no more than 2 CPU threads per session
- For datasets > 50,000 cells the desktop version is recommended, as some steps cannot be completed in the online version

## Evidence

- [methods] Run cluster-specific differential expression in Seurat, testing each cluster's cells against all remaining cells, with the Wilcoxon rank sum test, MAST, and DESeq2.: [evidence span withheld — anchor status: not_found]
- [results] SCALA identifies cluster marker genes by comparing each cluster's transcriptomes against the rest of the cells' transcriptomes through the Wilcoxon rank sum test on the normalized expression values.: [evidence span withheld — anchor status: not_found]
