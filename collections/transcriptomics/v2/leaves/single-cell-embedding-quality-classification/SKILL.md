---
name: single-cell-embedding-quality-classification
description: Use when a 2D scRNA-seq embedding (UMAP, t-SNE, diffusion maps) must
  be judged fit for trajectory/pseudotime inference, or when a UMAP run exhibits disconnected
  or unexpectedly grouped small clusters that would bias Slingshot/Monocle3 pseudotime.
when_to_use_negative:
- The trajectory inference method under consideration does not use two-dimensional
  representations — Escort 'only considers trajectory inference approaches that use
  two-dimensional representations'.
- You need RNA velocity-derived dimension reduction embeddings — Escort does not yet
  implement support for these.
- A downstream goal already provides its own validated ground-truth trajectory (e.g.,
  duplicated ground truth time removed in the simulation study); connectivity flags
  alone do not substitute for pseudotime accuracy benchmarking.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Escort
  role: R package / R-Shiny framework that runs the Manhattan-distance/Jaccard connectivity
    check and classifies embeddings as recommended or not-recommended for trajectory
    inference
  repo: https://github.com/xiaorudong/Escort
- name: umap
  role: R package (v0.2.10.0) used to generate the candidate UMAP embeddings being
    evaluated, run directly on normalized data with default settings
- name: scLCA
  role: Clustering method used to obtain the optimal number of clusters and cell cluster
    assignments before the connectivity check (SC3 used instead when prior cell type
    information is available)
- name: SC3
  role: Alternative Bioconductor clustering tool used in place of scLCA when prior
    cell type information is available
- name: R
  role: Implementation environment for Escort and the embedding evaluation workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:6238f3fdf251680e2fe2393e4c7ada41fa9ffdf884266226e22166eee7258655
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
  - scLCA
  - umap
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-embedding-quality-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Escort-based classification of scRNA-seq embeddings (e.g., UMAP) as recommended or not-recommended for trajectory inference, using a Manhattan-distance/Jaccard connectivity check that immediately flags any embedding with newly disconnected clusters.

## When to use

Use when a 2D scRNA-seq embedding (UMAP, t-SNE, diffusion maps) must be judged fit for trajectory/pseudotime inference, or when a UMAP run exhibits disconnected or unexpectedly grouped small clusters that would bias Slingshot/Monocle3 pseudotime.

## When NOT to use

- The trajectory inference method under consideration does not use two-dimensional representations — Escort 'only considers trajectory inference approaches that use two-dimensional representations'.
- You need RNA velocity-derived dimension reduction embeddings — Escort does not yet implement support for these.
- A downstream goal already provides its own validated ground-truth trajectory (e.g., duplicated ground truth time removed in the simulation study); connectivity flags alone do not substitute for pseudotime accuracy benchmarking.

## Inputs

- Normalized single-cell expression matrix (e.g., Escort exampleData_linear, simulated evaluation datasets, GSE190616 human pancreas scRNA-seq)
- Cell cluster assignments from scLCA or SC3 (optimal number of clusters and per-cell labels)
- Candidate 2D embeddings (e.g., UMAP output from the R umap package v0.2.10.0)

## Outputs

- escort_connectivity_results.csv with per-embedding connectivity classifications and counts of disconnected or unexpectedly grouped small clusters
- Per-embedding recommendation flags (not-recommended vs. score > 0 recommended) on Escort's negative-infinity-to-two benchmark scale

## How to apply

Start from normalized single-cell expression matrices (e.g., Escort's exampleData_linear or the publicly available simulated evaluation datasets) and cluster cells with scLCA — or SC3 when prior cell type information is available — to fix cluster assignments. Compute the N x N Manhattan distance matrix between cells and derive within-cluster and between-cluster distance sets per cell, then calculate Jaccard index scores from these distributions, recalculating for the top 20% highest-Jaccard cells per cluster. Declare two clusters connected only when more than 10 cells per cluster meet the 0.3 cutoff criteria; otherwise they are disconnected. Re-apply the identical distance/Jaccard connectivity check to each candidate embedding (e.g., UMAP from the R umap package v0.2.10.0 run on normalized data with default settings): an embedding is accepted as reliable only if clusters disconnected in Step 1 do not reappear as disconnected in the embedding, and any embedding with newly disconnected clusters is classified immediately as not recommended. Escort then scores embeddings on a benchmark scale from negative infinity to two, where scores greater than zero are recommended for trajectory construction.

## Related tools

- **Escort** (R package / R-Shiny framework that runs the Manhattan-distance/Jaccard connectivity check and classifies embeddings as recommended or not-recommended for trajectory inference) — https://github.com/xiaorudong/Escort
- **umap** (R package (v0.2.10.0) used to generate the candidate UMAP embeddings being evaluated, run directly on normalized data with default settings)
- **scLCA** (Clustering method used to obtain the optimal number of clusters and cell cluster assignments before the connectivity check (SC3 used instead when prior cell type information is available))
- **SC3** (Alternative Bioconductor clustering tool used in place of scLCA when prior cell type information is available)
- **R** (Implementation environment for Escort and the embedding evaluation workflow)

## Examples

```
if (!requireNamespace("remotes", quietly=TRUE)) install.packages("remotes"); remotes::install_github("xiaorudong/Escort"); library(Escort)
```

## Evaluation signals

- An embedding passes only if clusters found disconnected in Step 1 do not reappear as disconnected in the embedding.
- Any embedding with newly disconnected clusters must be flagged 'not recommended' — zero such embeddings should be left unflagged in escort_connectivity_results.csv.
- Connection decisions reproduce deterministically: >10 cells per cluster meeting the 0.3 Jaccard cutoff, recalculated for the top 20% highest-Jaccard cells per cluster.
- Recommended embeddings must carry an Escort benchmark score greater than zero on the negative-infinity-to-two scale.
- Output CSV is named escort_connectivity_results.csv and includes counts of disconnected or unexpectedly grouped small UMAP clusters.

## Limitations

- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated.
- Escort only considers trajectory inference approaches that use two-dimensional representations; methods outside this restriction require further evaluation.
- Support for RNA velocity-derived dimension reduction methods is not yet implemented (future work).
- Quality control steps may significantly impact Escort's embedding evaluations, and performance of processing choices is highly dataset-specific.
- No software version is specified for Escort or R in the paper.

## Evidence
