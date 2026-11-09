---
name: cluster-connectivity-disconnectedness-evaluation
description: Use when you must decide whether a dimension-reduced embedding (e.g.,
  UMAP, PCA, t-SNE) is suitable as input to trajectory/pseudotime inference in scRNA-seq
  data, and specifically need to detect clusters that become disconnected or small
  clusters that group unexpectedly in the embedding.
when_to_use_negative:
- The downstream trajectory inference method does not operate on two-dimensional representations
  — Escort presently only considers trajectory inference approaches that use two-dimensional
  representations, and methods outside this restriction require further evaluation.
- The goal is to evaluate RNA velocity-derived dimension reduction methods, which
  are not yet implemented in Escort (future work).
- An embedding has already passed trajectory-specific scoring and you only need pseudotime
  ordering (e.g., Slingshot/Monocle3 estimation) rather than connectivity pre-screening.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Escort
  role: Framework implementing the connectivity/disconnectedness evaluation; classifies
    embeddings with disconnected clusters as immediately not recommended for trajectory
    inference
  repo: https://github.com/xiaorudong/Escort
- name: R
  role: Language in which Escort (R package and R/Shiny application) is implemented
    and the connectivity check is executed
  repo: https://github.com/xiaorudong/Escort
- name: UMAP
  role: Dimension reduction method whose embeddings (R umap package v0.2.10.0, default
    settings on normalized data) are subjected to the connectivity check
- name: Shiny
  role: Web interface through which Escort's embedding evaluations, including disconnectedness
    checks, can be run (Escort::shinyEscort())
  repo: https://github.com/xiaorudong/Escort
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:bb64b40660d610c827aee41d3fd042bc0a1c174a258d2df7b62167a1afe9ca12
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaorudong/Escort
  repo_url: https://github.com/xiaorudong/Escort
  tools:
  - Escort
  - R
  - Shiny
  - UMAP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cluster-connectivity-disconnectedness-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates whether cell clusters in a 2D embedding (e.g., UMAP of normalized scRNA-seq data) remain connected, using per-cell within/between-cluster Manhattan distance distributions and a Jaccard index test with a 0.3 cutoff, so that embeddings exhibiting disconnected clusters are immediately excluded from trajectory inference.

## When to use

Use when you must decide whether a dimension-reduced embedding (e.g., UMAP, PCA, t-SNE) is suitable as input to trajectory/pseudotime inference in scRNA-seq data, and specifically need to detect clusters that become disconnected or small clusters that group unexpectedly in the embedding.

## When NOT to use

- The downstream trajectory inference method does not operate on two-dimensional representations — Escort presently only considers trajectory inference approaches that use two-dimensional representations, and methods outside this restriction require further evaluation.
- The goal is to evaluate RNA velocity-derived dimension reduction methods, which are not yet implemented in Escort (future work).
- An embedding has already passed trajectory-specific scoring and you only need pseudotime ordering (e.g., Slingshot/Monocle3 estimation) rather than connectivity pre-screening.

## Inputs

- normalized single-cell expression matrix (e.g., Seurat LogNormalize output from scRNA-seq data such as the Escort exampleData_linear dataset or publicly available simulated datasets)
- cluster assignments and optimal cluster number from scLCA or SC3 clustering of the cells
- candidate 2D embeddings (e.g., UMAP generated with R umap v0.2.10.0, plus PCA/t-SNE/diffusion map alternatives)

## Outputs

- per-embedding connectivity classification (connected / disconnected, recommended / not recommended)
- per-cluster Jaccard index scores from within- and between-cluster Manhattan distance distributions
- summary table such as escort_connectivity_results.csv with counts of disconnected or unexpectedly grouped small clusters

## How to apply

First cluster the normalized single-cell expression matrix (e.g., with scLCA to obtain the optimal number of clusters, or SC3 when prior cell type information is available) and compute the N x N Manhattan distance matrix between cells. For each cell derive within-cluster and between-cluster distance sets, calculate Jaccard index scores from these distributions, and recalculate the score for the top 20% highest-Jaccard cells per cluster; declare two clusters connected only when more than 10 cells per cluster meet the 0.3 Jaccard cutoff criteria, otherwise disconnected. Apply this same connectivity check first to the clusters on the original normalized data (Step 1 baseline), then to each candidate 2D embedding — for the paper's evaluation, UMAP embeddings generated with the R umap package v0.2.10.0 run directly on normalized data with default settings. Any embedding in which Step 1 disconnected clusters reappear as disconnected (newly disconnected clusters) is immediately classified as not recommended for trajectory inference, regardless of any other scores. Summarize per-embedding connectivity classifications and counts of disconnected or unexpectedly grouped small clusters into an output table (e.g., escort_connectivity_results.csv).

## Related tools

- **Escort** (Framework implementing the connectivity/disconnectedness evaluation; classifies embeddings with disconnected clusters as immediately not recommended for trajectory inference) — https://github.com/xiaorudong/Escort
- **R** (Language in which Escort (R package and R/Shiny application) is implemented and the connectivity check is executed) — https://github.com/xiaorudong/Escort
- **UMAP** (Dimension reduction method whose embeddings (R umap package v0.2.10.0, default settings on normalized data) are subjected to the connectivity check)
- **Shiny** (Web interface through which Escort's embedding evaluations, including disconnectedness checks, can be run (Escort::shinyEscort())) — https://github.com/xiaorudong/Escort

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort)
```

## Evaluation signals

- Invariant: an embedding is accepted as reliable only if the Step 1 disconnected clusters do NOT reappear as disconnected in the embedding.
- Hard rule: any embedding with newly disconnected clusters must be flagged not recommended immediately, before any other scoring is considered.
- Parameter check: the Jaccard index is recalculated for the top 20% highest-Jaccard cells per cluster, and connectivity requires more than 10 cells per cluster meeting the 0.3 Jaccard cutoff.
- Distance matrix check: the metric used is the N x N Manhattan distance between cells, within- and between-cluster sets computed per cell.
- Output schema check: results summarized as per-embedding connectivity classifications with counts of disconnected or unexpectedly grouped small clusters in escort_connectivity_results.csv.

## Limitations

- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated.
- Quality control steps may significantly impact Escort's embedding evaluations.
- Escort presently only considers trajectory inference approaches that use two-dimensional representations; support for RNA velocity-derived dimension reduction methods is future work.

## Evidence

- [methods] any embeddings found to be disconnected are classified immediately as not recommended for trajectory inference: "any embeddings found to be disconnected are classified immediately as not recommended for trajectory inference"
- [abstract] benchmark scoring range: "Escort uses a standardized benchmark scoring system ranging from negative infinity to two. Higher scores indicate better performance. Embeddings with a score greater than zero are recommended"
- [discussion] disconnected clusters as first-step evaluation: "identifying disconnected cell clusters or homogenous cells lacking a trajectory signal"
- [readme] Escort purpose: "Escort is a framework for evaluating a single-cell RNA-seq dataset’s suitability for trajectory inference and for quantifying trajectory properties influenced by analysis decisions"
