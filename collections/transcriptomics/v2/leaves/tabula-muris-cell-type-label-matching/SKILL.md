---
name: tabula-muris-cell-type-label-matching
description: Use when a c-CSN-transformed CNDM (m genes x n cells) must be shown compatible
  with standard scRNA-seq pipelines, or when Seurat cluster labels on Tabula Muris
  droplet data need to be matched to known tissue origins and quantified by ARI.
when_to_use_negative:
- Input is a raw gene expression matrix (GEM) rather than a c-CSN-derived CNDM — the
  skill validates compatibility of the transformed degree matrix, not raw counts
- The dataset has fewer than the three tissues or lacks cell-origin ground-truth labels,
  making ARI against known origins impossible
- The goal is de novo cell-type annotation without reference labels rather than validation
  of CNDM compatibility against known origins
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: c-CSN
  role: Constructs the CCSNs and the conditional network degree matrix (CNDM) that
    serves as input to Seurat clustering
  repo: https://github.com/LinLi-0909/c-CSN
- name: Seurat
  role: Performs dimension reduction (t-SNE) and clustering on the normalized CNDM;
    clusters are matched to cell origins via ARI
- name: t-SNE
  role: Nonlinear dimension reduction used to visualize cells in a t-SNE map showing
    separation of the three tissue groups
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:e1deaa76a7cd6c2d10870158dc1df14ef63c758b1b33ef99329534fbeeeaf54b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - Seurat
  - c-CSN
  - t-SNE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# tabula-muris-cell-type-label-matching

## Summary

Validates that a c-CSN-derived conditional network degree matrix (CNDM) supports standard scRNA-seq clustering by running Seurat dimension reduction and clustering on the Tabula Muris droplet1 dataset and matching clusters to tissue/cell-type labels via adjusted Rand index.

## When to use

Use when a c-CSN-transformed CNDM (m genes x n cells) must be shown compatible with standard scRNA-seq pipelines, or when Seurat cluster labels on Tabula Muris droplet data need to be matched to known tissue origins and quantified by ARI.

## When NOT to use

- Input is a raw gene expression matrix (GEM) rather than a c-CSN-derived CNDM — the skill validates compatibility of the transformed degree matrix, not raw counts
- The dataset has fewer than the three tissues or lacks cell-origin ground-truth labels, making ARI against known origins impossible
- The goal is de novo cell-type annotation without reference labels rather than validation of CNDM compatibility against known origins

## Inputs

- c-CSN-derived CNDM (m genes x n cells matrix) for Tabula Muris droplet1 (GSE109774)
- Cell-origin annotations for bladder, trachea, and spleen (ground-truth labels)

## Outputs

- t-SNE visualization of the dimension-reduced CNDM
- Seurat cluster assignments for each cell
- Adjusted Rand index (ARI) of Seurat clusters vs. known tissue origins (expected ~0.73)

## How to apply

Load the c-CSN-derived CNDM for the Tabula Muris droplet1 dataset (GSE109774; >20,000 cells from bladder, trachea, and spleen) together with cell-origin annotations, and normalize it as described in the c-CSN methods so it enters standard scRNA-seq downstream pipelines. Run Seurat to perform dimension reduction, visualizing cells on a t-SNE map, then perform Seurat clustering on the dimension-reduced CNDM and check that cells segregate into three dominant groups corresponding to their tissue origins. Compute the adjusted Rand index (ARI) of Seurat clusters against the ground-truth cell origins. The workflow reproduces the reported ARI = 0.73 on the Tabula Muris droplet dataset, demonstrating that most existing scRNA-seq methods designed for gene expression matrices apply to c-CSN-transformed degree matrices.

## Related tools

- **c-CSN** (Constructs the CCSNs and the conditional network degree matrix (CNDM) that serves as input to Seurat clustering) — https://github.com/LinLi-0909/c-CSN
- **Seurat** (Performs dimension reduction (t-SNE) and clustering on the normalized CNDM; clusters are matched to cell origins via ARI)
- **t-SNE** (Nonlinear dimension reduction used to visualize cells in a t-SNE map showing separation of the three tissue groups)

## Examples

```
R: library(Seurat); cndm <- Seurat::CreateSeuratObject(read.csv('tabula_muris_cndm.csv')); cndm <- RunTSNE(cndm); cndm <- FindClusters(cndm); mclust::adjustedRandIndex(cndm$seurat_clusters, cell_origins)
```

## Evaluation signals

- Computed ARI falls within the reported tolerance band 0.65-0.80, with the reported value ARI = 0.73 as the target
- t-SNE map shows clear separation of the three tissue groups (bladder, trachea, spleen)
- Seurat clusters resolve into three dominant groups corresponding to the three tissue origins
- CNDM clustering ARI is at least competitive with, or better than, clustering the raw gene expression matrix (article reports CNDM performs obviously better than GEM on all datasets)
- Number of cells analyzed exceeds 20,000 as in the Tabula Muris droplet1 dataset

## Limitations

- Validation is specific to the Tabula Muris droplet1 configuration (bladder, trachea, spleen); ARI values on other tissues or datasets may differ
- c-CSN incurs a G-fold computational cost increase (one network per conditional gene) and no parallel computation scheme is provided despite this cost
- CNDM-derived clusters capture tissue origin; finer sub-type resolution beyond the labeled groups is not demonstrated by this ARI benchmark

## Evidence

- [methods] CNDM compatibility with Seurat clustering reproducing ARI: "The Tabula Muris droplet1 dataset [47] comprising more than 20,000 cells from three tissues (bladder, trachea, and spleen)"
- [methods] CNDM better than GEM: "CNDM performs obviously better than GEM on all datasets"
- [intro] Compatibility with traditional analyses: "we first demonstrate that the transformed gene–gene association data not only are fully compatible with traditional analyses such as dimension reduction and clustering"
- [abstract] Existing scRNA-seq methods applicable to CNDM: "2) Most existing scRNA-seq methods designed for gene expression matrices are also applicable to c-CSN-transformed degree matrices."
- [methods] Seurat among clustering methods applied to CNDM: "several traditional clustering methods such as K-means, Hierarchical clustering analysis, and K-medoids were applied to clustering analysis"
