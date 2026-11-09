---
name: tsne-visualization-of-cell-clusters
description: Use when an imputed or raw single-cell expression matrix must be visually
  inspected for cluster compactness and separation across cell types, e.g., alongside
  ARI/Jaccard/NMI/purity benchmarks.
when_to_use_negative:
- When the goal is a purely quantitative clustering benchmark — t-SNE embeddings are
  qualitative and the article computes ARI/Jaccard/NMI/purity numerically over 10
  random seeds instead
- When inputs are not single-cell expression matrices (e.g., bulk RNA-seq), where
  cell cluster separation is not the quantity of interest
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Bfimpute
  role: Produces the imputed count matrix whose t-SNE embedding is generated to check
    cluster compactness and separation
  repo: https://github.com/maiziezhoulab/Bfimpute
- name: splatter
  role: Simulates scRNA-seq matrices with 5-8 cell types and known ground-truth labels
    used to color t-SNE plots
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:5a380fa3660d6e1a40c5fc47176c5534a5ce636349de9209fe719e5f12fc6ad0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/maiziezhoulab/Bfimpute
  repo_url: https://github.com/maiziezhoulab/Bfimpute
  tools:
  - Bfimpute
  - DrImpute
  - MAGIC
  - PBLR
  - SAVER
  - Spectrum
  - VIPER
  - scImpute
  - splatter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# tsne-visualization-of-cell-clusters

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate t-SNE embeddings of imputed scRNA-seq matrices to visually verify that dropout imputation (e.g., by Bfimpute versus scImpute, DrImpute, MAGIC, SAVER, VIPER, PBLR) yields more compact, well-separated cell type clusters before quantitative clustering evaluation.

## When to use

Use when an imputed or raw single-cell expression matrix must be visually inspected for cluster compactness and separation across cell types, e.g., alongside ARI/Jaccard/NMI/purity benchmarks.

## When NOT to use

- When the goal is a purely quantitative clustering benchmark — t-SNE embeddings are qualitative and the article computes ARI/Jaccard/NMI/purity numerically over 10 random seeds instead
- When inputs are not single-cell expression matrices (e.g., bulk RNA-seq), where cell cluster separation is not the quantity of interest

## Inputs

- Imputed gene-by-cell expression count matrix (e.g., Bfimpute R_calculate output from a splatter-simulated 20,000-gene, 500-800-cell matrix)
- Raw scRNA-seq count matrix for baseline comparison
- Cell type labels or estimated cluster assignments for coloring cells

## Outputs

- t-SNE embedding of cells (2D coordinates) per imputed matrix
- t-SNE scatter plots colored by cell type, comparable to Figure 2 cluster-separation figures

## How to apply

First produce the imputed count matrix, e.g., by running Bfimpute on the simulated or real scRNA-seq matrix with spectral clustering (ccluster = c(7,"Spectrum"), D = 32, totalepoch = 300, burnin = 200, threshold = 0.5, ncores = 5), letting Bfimpute apply CPM and log10 transform with bias 1.01 when normalized = FALSE. Then embed the imputed matrix with t-SNE, coloring cells by the known or estimated cell type labels. Compare embeddings across methods (raw, Bfimpute, and comparison imputers) to assess which produces the most compact, well-separated clusters, mirroring the article's Figure 2 visualization workflow. Treat visual separation as qualitative corroboration of the quantitative clustering scores (adjusted Rand index, Jaccard index, normalized mutual information, purity) computed over repeated random seeds.

## Related tools

- **Bfimpute** (Produces the imputed count matrix whose t-SNE embedding is generated to check cluster compactness and separation) — https://github.com/maiziezhoulab/Bfimpute
- **splatter** (Simulates scRNA-seq matrices with 5-8 cell types and known ground-truth labels used to color t-SNE plots)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), ncores = 5)
```

## Evaluation signals

- The t-SNE embedding of Bfimpute-imputed data shows more compact, well-separated clusters than embeddings of raw or other-imputed matrices
- Cell colors in the embedding correspond to known simulated labels for 5, 6, 7, and 8 cell type configurations
- Visual cluster structure is consistent with the direction of the quantitative scores (ARI, Jaccard, NMI, purity) computed over 10 random seeds
- The same embedding pipeline is applied uniformly to raw and all imputed matrices so comparisons are apples-to-apples

## Limitations

- The article cautions that 'it is premature to conclude that Bfimpute will achieve better performance across all datasets', so t-SNE separation on one dataset does not generalize
- Prior methods were 'less effective in recovering cell-to-cell relationships resulting in less improvement in cell clustering', so t-SNE of some imputers may still show poor separation; interpret side-by-side rather than in isolation
- t-SNE is a qualitative check; it does not replace the numeric clustering metrics in Figure 2B-style tables

## Evidence
