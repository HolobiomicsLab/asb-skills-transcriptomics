---
name: supervised-pca-integration-embedding-handling
description: Use when a SingCellaR object already contains Supervised PCA embeddings
  plus cell-metadata covariates (e.g. sample/batch), and multiple scRNA-seq samples
  must be batch-corrected and clustered into a single integrated embedding.
when_to_use_negative:
- Input data lacks Supervised PCA embeddings or covariate metadata, so there is nothing
  for Supervised Harmony to correct
- The goal is unsupervised clustering within a single sample rather than multi-sample
  batch-corrected integration
- The task is variable gene selection for integration (use get_variable_genes_for_integrative_data_by_fitting_GLM_model)
  rather than embedding correction itself
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SingCellaR
  role: Provides the runSupervised_Harmony function and stores the corrected embedding
    matrix and cutree clusters in its object
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Supplies stats::hclust and cutree used to build and cut the hierarchical clustering
    tree on the corrected embeddings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:50740ebd0fe79f1a803fb557777f6ec86c18fb38cce77d78aa41aaa76031501b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# supervised-pca-integration-embedding-handling

## Summary

Handles Supervised PCA integration embeddings in SingCellaR: runs Supervised Harmony (an adaptation of Harmony batch correction, Korsunsky et al. 2019) on the supervised PCA embedding matrix using covariate columns as batch variables, then derives flat cell clusters via hierarchical clustering and cutree.

## When to use

Use when a SingCellaR object already contains Supervised PCA embeddings plus cell-metadata covariates (e.g. sample/batch), and multiple scRNA-seq samples must be batch-corrected and clustered into a single integrated embedding.

## When NOT to use

- Input data lacks Supervised PCA embeddings or covariate metadata, so there is nothing for Supervised Harmony to correct
- The goal is unsupervised clustering within a single sample rather than multi-sample batch-corrected integration
- The task is variable gene selection for integration (use get_variable_genes_for_integrative_data_by_fitting_GLM_model) rather than embedding correction itself

## Inputs

- SingCellaR object (.rds) with Supervised PCA embeddings already computed
- Cell metadata covariate column(s) to be used as batch variables
- hcl.height.cutoff value for cutting the hierarchical clustering tree

## Outputs

- Supervised Harmony integration embedding matrix (one row per cell)
- Flat cluster assignment vector per cell from stats::cutree
- supervised_harmony_embeddings.rds file storing the corrected embeddings
- Updated SingCellaR object queryable via get_umap.result() and get_clusters()

## How to apply

Load the SingCellaR object (.rds) that already contains Supervised PCA embeddings and the covariate columns to be corrected. Run the runSupervised_Harmony function, which performs Harmony-style iterative soft-clustering correction on the Supervised PCA embedding matrix using the specified covariate column(s) as batch variables. Build a hierarchical clustering tree on the corrected embeddings with stats::hclust, then cut the tree with cutree at a fixed hcl.height.cutoff to assign one cluster per cell. Store both the Harmony integration embedding matrix and the cutree cluster vector back in the SingCellaR object so downstream accessors (get_umap.result, get_clusters) can retrieve them; the corrected embeddings can also be saved as supervised_harmony_embeddings.rds. The rationale is that Supervised Harmony adapts the Harmony batch-correction method to the supervised PCA space, enabling integrative analysis across samples within SingCellaR's SingleCellExperiment-based object.

## Related tools

- **SingCellaR** (Provides the runSupervised_Harmony function and stores the corrected embedding matrix and cutree clusters in its object) — https://github.com/supatt-lab/SingCellaR
- **R** (Supplies stats::hclust and cutree used to build and cut the hierarchical clustering tree on the corrected embeddings)

## Examples

```
runSupervised_Harmony(object, covariate = "sample"); hc <- stats::hclust(dist(object@supervised_harmony_embeddings)); clusters <- cutree(hc, h = hcl.height.cutoff)
```

## Evaluation signals

- The saved supervised_harmony_embeddings.rds contains a numeric embedding matrix with exactly one row per cell
- The covariate-corrected embedding columns contain no missing (NA) values
- The cutree cluster assignment vector is non-empty and covers all cells
- get_umap.result(object) and get_clusters(object) can retrieve the stored embedding matrix and cluster vector downstream
- The corrected embeddings were computed from the Supervised PCA matrix using the specified covariate column(s), not from raw expression data

## Limitations

- The hcl.height.cutoff for cutree is a fixed parameter; a poor choice can fragment or over-merge the flat clusters
- Plot rotations and kBET integration scores can vary slightly between runs due to seed number settings and software version differences
- SingCellaR lacks 'automatic object transformation' to interact with other packages such as Seurat, so the integrated embeddings are not trivially portable

## Evidence

- [methods] Harmony on supervised PCA workflow: [evidence span withheld — anchor status: not_found]
- [methods] cutree clustering step: [evidence span withheld — anchor status: not_found]
- [discussion] object interoperability caveat: "SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat."
- [readme] tool purpose: "an integrative analysis tool for analysing single-cell RNA sequencing data"
