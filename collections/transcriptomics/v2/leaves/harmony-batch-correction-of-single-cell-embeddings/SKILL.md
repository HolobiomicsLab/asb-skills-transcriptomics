---
name: harmony-batch-correction-of-single-cell-embeddings
description: 'Use when multiple scRNA-seq samples must be integrated: the SingCellaR
  object already has Supervised PCA embeddings and cell metadata covariates (e.g.
  sample batch) that need correction before downstream clustering and UMAP visualization.'
when_to_use_negative:
- The input lacks Supervised PCA embeddings or the cell metadata covariates, so there
  is nothing to correct — run dimensionality reduction first
- Data are a single sample with no batch covariate, where batch correction is unnecessary
- The user wants interoperability with Seurat objects — SingCellaR 'still lacks automatic
  object transformation' to interact with other packages such as Seurat
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SingCellaR
  role: provides the runSupervised_Harmony function, the SingCellaR object that stores
    the corrected embeddings and cluster assignments, and downstream accessors
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: implementation environment; stats::hclust builds the hierarchical clustering
    tree on corrected embeddings and cutree assigns flat clusters
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:43936c7d8f5f13dc8c09c0eaf9cc5322479c9164988bf32f5d80c0756ac533fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# harmony-batch-correction-of-single-cell-embeddings

## Summary

Supervised Harmony batch correction of Supervised PCA embeddings from multiple scRNA-seq samples within SingCellaR, an adaptation of the Harmony method (Korsunsky et al., 2019), followed by hierarchical clustering of the corrected embeddings.

## When to use

Use when multiple scRNA-seq samples must be integrated: the SingCellaR object already has Supervised PCA embeddings and cell metadata covariates (e.g. sample batch) that need correction before downstream clustering and UMAP visualization.

## When NOT to use

- The input lacks Supervised PCA embeddings or the cell metadata covariates, so there is nothing to correct — run dimensionality reduction first
- Data are a single sample with no batch covariate, where batch correction is unnecessary
- The user wants interoperability with Seurat objects — SingCellaR 'still lacks automatic object transformation' to interact with other packages such as Seurat

## Inputs

- SingCellaR object (.rds) containing Supervised PCA embeddings
- cell metadata covariate column(s) to be corrected (batch variables)
- hcl.height.cutoff value for cutree

## Outputs

- Supervised Harmony embedding matrix (numeric, one row per cell)
- flat cluster assignment vector per cell from cutree
- updated SingCellaR object with integrative embeddings accessible via get_umap.result() and get_clusters()

## How to apply

Load the saved SingCellaR object (.rds) that already contains Supervised PCA embeddings and the cell metadata covariates to be corrected, then run Supervised Harmony via the runSupervised_Harmony function, supplying the covariate column(s) as batch variables for the Korchmaros-style iterative soft-clustering correction of the embedding matrix. Build a hierarchical clustering tree on the corrected embeddings using stats::hclust and cut the tree with cutree at a fixed hcl.height.cutoff to obtain a flat cluster assignment per cell. Store both the Harmony integration embedding matrix and the cutree cluster vector back in the SingCellaR object so downstream accessors such as get_umap.result(object) and get_clusters(object) can retrieve them. The rationale is that correcting covariate-driven variation in the Supervised PCA space before clustering prevents samples from separating by batch rather than by cell type in the hematopoietic HSPC analysis.

## Related tools

- **SingCellaR** (provides the runSupervised_Harmony function, the SingCellaR object that stores the corrected embeddings and cluster assignments, and downstream accessors) — https://github.com/supatt-lab/SingCellaR
- **R** (implementation environment; stats::hclust builds the hierarchical clustering tree on corrected embeddings and cutree assigns flat clusters)

## Examples

```
runSupervised_Harmony(object, covariates = c("sample")); tree <- hclust(dist(supervised_harmony_embeddings)); clusters <- cutree(tree, h = hcl.height.cutoff)
```

## Evaluation signals

- supervised_harmony_embeddings.rds contains a numeric embedding matrix with exactly one row per cell
- covariate-corrected embedding columns have no missing values (NA)
- the cutree cluster assignment vector is non-empty and covers all cells
- downstream accessors get_umap.result(object) and get_clusters(object) return the stored Harmony embeddings and cluster vector without error

## Limitations

- SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat
- Plot rotations from downstream UMAP/FDG visualization may differ from the protocol due to software version and seed number differences
- The protocol is demonstrated on human hematopoietic stem and progenitor cells; covariate choice and hcl.height.cutoff may need adjustment for other systems

## Evidence

- [abstract] a comprehensive analytical protocol to interrogate multiple datasets on SingCellaR, an analysis package in R: "a comprehensive analytical protocol to interrogate multiple datasets on SingCellaR, an analysis package in R"
- [methods] identifyClusters with integrative embeddings: "`identifyClusters(object, dim_reduction_method, useIntegrativeEmbeddings, integrative_method, n.dims.use, n.neighbors, knn.n_trees, knn.metric, knn.n_threads)`"
- [discussion] SingCellaR SingleCellExperiment object: "SingCellaR uses the SingleCellExperiment object, the standard object for storing single-cell experimental data in R."
- [readme] SingCellaR description from README: "an integrative analysis tool for analysing single-cell RNA sequencing data"
