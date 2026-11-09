---
name: seurat-dimension-reduction-clustering
description: Use when the input is a c-CSN-transformed conditional network degree
  matrix (CNDM, m genes x n cells) and the goal is standard scRNA-seq downstream analysis
  — t-SNE visualization, clustering, and ARI scoring against known cell labels.
when_to_use_negative:
- The input is a raw gene expression matrix (GEM) that has not been transformed by
  c-CSN — run the c-CSN/CNDM construction skill first, since this skill operates on
  network degree data
- The goal is constructing or interpreting the gene-gene association network itself
  (e.g., direct vs. indirect associations or NFE-based differentiation potency) rather
  than cell-level dimension reduction and clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: c-CSN
  role: Produces the conditional network degree matrix (CNDM) that is the direct input
    for Seurat dimension reduction and clustering
  repo: https://github.com/LinLi-0909/c-CSN
- name: Seurat
  role: Performs normalization-compatible dimension reduction (PCA, t-SNE) and clustering
    on the CNDM to yield cell clusters for ARI evaluation
- name: t-SNE
  role: Nonlinear dimension-reduction step used to visualize the clustered CNDM cells
    in 2D
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:9c21947e576a68672e3e4c669278726403667e26bc311554476ba6cf67874f1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - STtools
  - Seurat
  - c-CSN
  - t-SNE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- seurat-dimension-reduction-and-clustering
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# seurat-dimension-reduction-clustering

## Summary

Run Seurat-based dimension reduction (PCA/t-SNE) and clustering directly on a c-CSN-derived conditional network degree matrix (CNDM), treating gene-gene association degrees like an expression matrix to identify cell types (e.g., reproducing ARI = 0.73 on Tabula Muris droplet1).

## When to use

Use when the input is a c-CSN-transformed conditional network degree matrix (CNDM, m genes x n cells) and the goal is standard scRNA-seq downstream analysis — t-SNE visualization, clustering, and ARI scoring against known cell labels.

## When NOT to use

- The input is a raw gene expression matrix (GEM) that has not been transformed by c-CSN — run the c-CSN/CNDM construction skill first, since this skill operates on network degree data
- The goal is constructing or interpreting the gene-gene association network itself (e.g., direct vs. indirect associations or NFE-based differentiation potency) rather than cell-level dimension reduction and clustering

## Inputs

- c-CSN-derived conditional network degree matrix (CNDM), m genes x n cells
- Cell-origin annotation labels (e.g., bladder, trachea, spleen from Tabula Muris droplet1, GSE109774)
- Raw scRNA-seq count matrix (if the CNDM must first be generated via c-CSN)

## Outputs

- t-SNE map of cells from the dimension-reduced CNDM
- Seurat cluster assignments per cell
- Adjusted Rand index (ARI) of clusters vs. ground-truth cell origins

## How to apply

First normalize the CNDM (m genes x n cells) with the c-CSN normalization so it is compatible with standard scRNA-seq pipelines. Load the normalized CNDM into Seurat and perform dimension reduction with PCA and t-SNE — the article uses PCA and t-SNE as the linear and nonlinear representatives. Cluster the dimension-reduced CNDM with Seurat's clustering (alternatives evaluated in the paper include K-means, hierarchical clustering, K-medoids, SC3, and SIMLR) and compare clusters against known cell-origin annotations. Validate by computing the adjusted Rand index (ARI): on the Tabula Muris droplet1 dataset (GSE109774, >20,000 cells from bladder, trachea, and spleen), the expected result is ARI = 0.73 with three dominant clusters matching tissue origins on the t-SNE map. The rationale is that most scRNA-seq methods designed for gene expression matrices apply unchanged to c-CSN-transformed degree matrices, which capture direct gene-gene associations.

## Related tools

- **c-CSN** (Produces the conditional network degree matrix (CNDM) that is the direct input for Seurat dimension reduction and clustering) — https://github.com/LinLi-0909/c-CSN
- **Seurat** (Performs normalization-compatible dimension reduction (PCA, t-SNE) and clustering on the CNDM to yield cell clusters for ARI evaluation)
- **t-SNE** (Nonlinear dimension-reduction step used to visualize the clustered CNDM cells in 2D)

## Examples

```
library(Seurat); seu <- CreateSeuratObject(cndm); seu <- NormalizeData(seu); seu <- RunPCA(seu); seu <- RunTSNE(seu); seu <- FindClusters(seu)
```

## Evaluation signals

- ARI of Seurat clusters against known cell origins reproduces the reported ARI = 0.73 on Tabula Muris droplet1 (acceptable tolerance ~0.65-0.80)
- t-SNE map shows clear separation into three dominant groups corresponding to bladder, trachea, and spleen (>20,000 cells)
- CNDM clustering ARI is higher than clustering the original gene expression matrix (GEM) on the same dataset, as CNDM was reported to outperform GEM on all datasets
- CNDM dimensions match the expected m genes x n cells structure and have been normalized per the c-CSN methods before loading into Seurat
- Cluster count and composition are stable across Seurat clustering, consistent with the alternative methods (K-means, hierarchical, K-medoids, SC3, SIMLR) applied in the paper

## Limitations

- The G-fold conditional tests make c-CSN (and hence CNDM generation) computationally expensive; the authors note no parallel computation scheme is provided, stating 'a parallel computation scheme is desired to reduce the computation time'
- Reported ARI = 0.73 was demonstrated on the Tabula Muris droplet1 three-tissue dataset; performance on datasets with finer cell-type granularity is not established by this evidence
- c-CSN/CNDM captures gene-gene association degrees, not causal direction — the authors state c-CSN is not designed to construct causal gene association networks

## Evidence

- [abstract] Most existing scRNA-seq methods designed for gene expression matrices are also applicable to c-CSN-transformed degree matrices.: "Most existing scRNA-seq methods designed for gene expression matrices are also applicable to c-CSN-transformed degree matrices."
- [methods] Dimension reduction on CNDM with PCA and t-SNE: "We used PCA [35] and t-SNE [36] which respectively represent linear and nonlinear methods, to perform dimension reduction"
- [methods] CNDM definition as m x n matrix: "an m × n matrix conditional network degree matrix (CNDM) is obtained"
- [methods] Traditional clustering methods applied to the CNDM: "several traditional clustering methods such as K-means, Hierarchical clustering analysis, and K-medoids were applied to clustering analysis"
- [other] Seurat clustering validation against tissue origins on Tabula Muris: [evidence span withheld — anchor status: not_found]
- [methods] CNDM outperforms GEM in clustering: "CNDM performs obviously better than GEM on all datasets"
