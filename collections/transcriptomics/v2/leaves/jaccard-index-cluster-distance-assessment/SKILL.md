---
name: jaccard-index-cluster-distance-assessment
description: Decision support for judging whether a 2D embedding (e.g. PCA, UMAP,
  t-SNE of normalized scRNA-seq data) is suitable for trajectory inference, by detecting
  disconnected cell clusters or homogeneous cells lacking a trajectory signal before
  fitting a trajectory.
when_to_use_negative:
- The trajectory inference method of interest does not rely on a two-dimensional representation
  — Escort only considers trajectory inference approaches using two-dimensional representations
- The dimension reduction is derived from RNA velocity — Escort has not yet implemented
  support for velocity-derived dimension reduction methods
- You need a guaranteed recommendation — Escort is not guaranteed to identify a recommended
  embedding among the evaluated choices
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Escort
  role: R package / Shiny app implementing the Manhattan-distance Jaccard connectivity
    assessment and the benchmark scoring system (scores from negative infinity to
    two; scores > 0 are recommended)
  repo: https://github.com/xiaorudong/Escort
- name: R
  role: Implementation environment; Escort is an accessible R package and R/Shiny
    application
  repo: https://github.com/xiaorudong/Escort
- name: SC3
  role: Bioconductor clustering used to obtain cell cluster assignments when prior
    cell type information is available (scLCA otherwise)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:59bc49ef9453c5f9aa8a4e66e1698a4df0e61dc1579c9962295d839e8ca4f9df
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
  - SC3
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Jaccard-index cluster distance assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Escort's connectivity check for single-cell embeddings: computes Manhattan distance-based within- vs between-cluster distance distributions, derives Jaccard index scores per cell, and declares clusters disconnected (immediately disqualifying the embedding for trajectory inference) when connectivity fails the 0.3 cutoff in >10 cells per cluster.

## When to use

Decision support for judging whether a 2D embedding (e.g. PCA, UMAP, t-SNE of normalized scRNA-seq data) is suitable for trajectory inference, by detecting disconnected cell clusters or homogeneous cells lacking a trajectory signal before fitting a trajectory.

## When NOT to use

- The trajectory inference method of interest does not rely on a two-dimensional representation — Escort only considers trajectory inference approaches using two-dimensional representations
- The dimension reduction is derived from RNA velocity — Escort has not yet implemented support for velocity-derived dimension reduction methods
- You need a guaranteed recommendation — Escort is not guaranteed to identify a recommended embedding among the evaluated choices

## Inputs

- Normalized scRNA-seq expression matrix (cells x genes), e.g. Seurat LogNormalize output from datasets such as Escort's exampleData_linear
- Cell cluster assignments and optimal cluster number (from scLCA or SC3)
- Candidate 2D embeddings to evaluate (PCA, UMAP, t-SNE, diffusion maps), e.g. UMAP (R umap package v0.2.10.0) run on normalized data with default settings

## Outputs

- Per-cluster Jaccard index scores based on within- vs between-cluster Manhattan distance distributions
- Connected/disconnected classification per cluster pair (0.3 cutoff, >10 cells per cluster)
- Per-embedding recommendation status: embeddings with newly disconnected clusters flagged 'not recommended for trajectory inference'
- Summary table of connectivity classifications per embedding, e.g. escort_connectivity_results.csv

## How to apply

Start from the normalized single-cell expression matrix and cluster assignments (e.g. from scLCA, or SC3 when prior cell type information is available). Compute the N x N Manhattan distance matrix between cells (Escort uses parallelDist-type computation) and derive within-cluster and between-cluster distance sets for each cell. Calculate Jaccard index scores comparing the within- and between-cluster distance distributions, then recalculate the scores for the top 20% highest-Jaccard cells within each cluster. Declare two clusters connected when more than 10 cells per cluster meet the 0.3 cutoff criteria; otherwise they are disconnected. Apply the identical distance/Jaccard check to each candidate embedding: any embedding in which previously connected clusters appear newly disconnected is immediately classified as not recommended for trajectory inference, since disconnected structure in the embedding indicates the representation cannot support a continuous trajectory.

## Related tools

- **Escort** (R package / Shiny app implementing the Manhattan-distance Jaccard connectivity assessment and the benchmark scoring system (scores from negative infinity to two; scores > 0 are recommended)) — https://github.com/xiaorudong/Escort
- **R** (Implementation environment; Escort is an accessible R package and R/Shiny application) — https://github.com/xiaorudong/Escort
- **SC3** (Bioconductor clustering used to obtain cell cluster assignments when prior cell type information is available (scLCA otherwise))

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort); Escort::shinyEscort()
```

## Evaluation signals

- Two clusters are declared connected only when more than 10 cells per cluster meet the 0.3 Jaccard cutoff — verify per-cluster cell counts against this threshold
- Jaccard scores must be recomputed on the top 20% highest-Jaccard cells per cluster before the connectivity decision
- Validation invariant: an embedding is accepted as reliable only if Step 1 disconnected clusters do not reappear as disconnected in the embedding
- Embeddings with newly disconnected clusters must be flagged not recommended; Escort benchmark scores range from negative infinity to two, with scores > 0 recommended
- Output table (e.g. escort_connectivity_results.csv) should contain per-embedding connectivity classifications for every evaluated embedding

## Limitations

- Escort only considers trajectory inference approaches that use two-dimensional representations; methods outside this restriction require further evaluation
- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated
- Support for RNA velocity-derived dimension reduction methods is not yet implemented (future work)
- Quality control steps may significantly impact Escort's embedding evaluations

## Evidence
