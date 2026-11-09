---
name: hierarchical-clustering-cutree-cutoff-application
description: Use when a batch-corrected (Supervised Harmony) embedding matrix with
  one row per cell is available and you need a single flat cluster assignment per
  cell to feed get_clusters() downstream in SingCellaR.
when_to_use_negative:
- The embedding matrix has not yet been batch-corrected by Supervised Harmony — clustering
  raw Supervised PCA embeddings would mix batch effects into the tree
- You need a data-driven number of clusters (e.g. Louvain/Leiden resolution search)
  rather than a fixed hcl.height.cutoff-based flat cut
- The input is already a cluster label table or feature table rather than a cell-by-dimension
  embedding matrix
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SingCellaR
  role: Provides runSupervised_Harmony (Harmony adaptation), stores the corrected
    embedding matrix and cutree cluster vector, and exposes get_umap.result() and
    get_clusters() accessors
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Runtime environment; stats::hclust builds the tree and cutree performs the
    fixed-height flat cut on the Supervised Harmony embeddings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:c6a22c70518dd611831a8bda7431e967688b258090c8d98549c7ff0638f799df
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
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# hierarchical-clustering-cutree-cutoff-application

## Summary

Builds a hierarchical clustering tree with stats::hclust on Supervised Harmony-corrected Supervised PCA embeddings of scRNA-seq samples and cuts it with cutree at a fixed hcl.height.cutoff to assign a flat cluster label per cell, as implemented in the SingCellaR protocol for integrating multiple samples.

## When to use

Use when a batch-corrected (Supervised Harmony) embedding matrix with one row per cell is available and you need a single flat cluster assignment per cell to feed get_clusters() downstream in SingCellaR.

## When NOT to use

- The embedding matrix has not yet been batch-corrected by Supervised Harmony — clustering raw Supervised PCA embeddings would mix batch effects into the tree
- You need a data-driven number of clusters (e.g. Louvain/Leiden resolution search) rather than a fixed hcl.height.cutoff-based flat cut
- The input is already a cluster label table or feature table rather than a cell-by-dimension embedding matrix

## Inputs

- SingCellaR object (.rds) containing Supervised PCA embeddings
- Cell metadata covariate column(s) used as batch variables for Supervised Harmony correction
- Supervised Harmony-corrected embedding matrix (numeric, one row per cell)
- Fixed hcl.height.cutoff value for cutree

## Outputs

- Supervised Harmony integration embedding matrix stored in the SingCellaR object
- Flat per-cell cluster assignment vector from cutree, retrievable via get_clusters(object)
- Saved supervised_harmony_embeddings.rds file

## How to apply

Load the SingCellaR object (.rds) that already contains Supervised PCA embeddings and the covariate columns to be corrected, then run Supervised Harmony via the 'runSupervised_Harmony' function (SingCellaR's adaptation of the Harmony batch-correction method of Korsunsky et al., 2019) on the Supervised PCA embedding matrix using the covariate column(s) as batch variables. Build a hierarchical clustering tree on the corrected Supervised Harmony embeddings with stats::hclust, then cut the tree with cutree at the fixed hcl.height.cutoff value to obtain one flat cluster label per cell. Store both the Harmony integration embedding matrix and the cutree cluster vector back in the SingCellaR object so that get_umap.result(object) and get_clusters(object) can retrieve them downstream. The fixed height cutoff is the key decision point: it determines cluster granularity and must be set before cutting rather than chosen per-batch.

## Related tools

- **SingCellaR** (Provides runSupervised_Harmony (Harmony adaptation), stores the corrected embedding matrix and cutree cluster vector, and exposes get_umap.result() and get_clusters() accessors) — https://github.com/supatt-lab/SingCellaR
- **R** (Runtime environment; stats::hclust builds the tree and cutree performs the fixed-height flat cut on the Supervised Harmony embeddings)

## Examples

```
h <- stats::hclust(dist(supervised_harmony_embeddings)); clusters <- cutree(h, h = hcl.height.cutoff)
```

## Evaluation signals

- supervised_harmony_embeddings.rds contains a numeric embedding matrix with exactly one row per cell
- The covariate-corrected embedding columns have no missing values
- The cutree cluster assignment vector is non-empty and covers all cells
- get_umap.result(object) and get_clusters(object) both retrieve the stored Harmony embeddings and cluster vector without error
- Every cell receives exactly one cluster label from the single fixed hcl.height.cutoff cut

## Limitations

- The flat cut depends on a fixed hcl.height.cutoff; an inappropriate cutoff under- or over-clusters without warning
- SingCellaR still lacks 'automatic object transformation' to interact with other packages such as Seurat, so the cluster vector may need manual export
- Downstream plots such as UMAP/FDG rotations can differ from the protocol due to software version and seed number differences

## Evidence

- [discussion] SingleCellExperiment storage backing the SingCellaR object: "SingCellaR uses the SingleCellExperiment object, the standard object for storing single-cell experimental data in R."
- [readme] SingCellaR purpose from repository README: "an integrative analysis tool for analysing single-cell RNA sequencing data"
- [results] Harmony method provenance (Korsunsky et al., 2019) as stated in the source card: [evidence span withheld — anchor status: not_found]
