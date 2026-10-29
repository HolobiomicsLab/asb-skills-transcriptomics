---
name: umap-dimension-reduction-embedding
description: Use when constructing low-dimensional embeddings of single-cell RNA-seq
  data with UMAP as an input choice for trajectory inference and you need to verify
  the embedding is not disconnected before fitting pseudotime with Slingshot or Monocle3.
when_to_use_negative:
- When the target trajectory method does not operate on a two-dimensional representation
  — Escort only considers trajectory inference approaches using two-dimensional representations
- When you want to pre-select a single universally best dimension reduction method
  — the benchmark shows no dimension reduction algorithm or HVG percentage shows a
  clear performance advantage across datasets
- When prior cell type information is available and clustering should be done with
  SC3 instead of scLCA as the clustering backbone
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Escort
  role: Framework that evaluates the UMAP embedding's suitability for trajectory inference
    via disconnected-cluster detection and benchmark scoring (range -inf to 2, scores
    > 0 recommended)
  repo: https://github.com/xiaorudong/Escort
- name: umap
  role: R package (v0.2.10.0) used to generate UMAP embeddings directly on normalized
    data with default settings
- name: Slingshot
  role: Trajectory/pseudotime inference method fitted on the UMAP embedding to measure
    accuracy (Kendall correlation, MSE)
- name: Monocle3
  role: Alternative trajectory inference method fitted on the UMAP embedding for the
    benchmark comparison
- name: Rtsne
  role: Generates the comparison t-SNE embedding (initial PCA retaining top 50 dimensions)
    against which UMAP is benchmarked
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:3379c751c1f24cbe2ea75ccb30ce8f3db931b11b6fcd9f8be0cb88a30570f18b
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
  - Monocle3
  - R
  - Rtsne
  - SCORPIUS
  - Seurat
  - Slingshot
  - mclust
  - scran
  - umap
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# umap-dimension-reduction-embedding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate two-dimensional UMAP embeddings from normalized scRNA-seq data and evaluate their suitability for trajectory inference using Escort's connectivity checks, since UMAP frequently produces disconnected or unexpectedly grouped clusters that disqualify it for pseudotime analysis.

## When to use

Use when constructing low-dimensional embeddings of single-cell RNA-seq data with UMAP as an input choice for trajectory inference and you need to verify the embedding is not disconnected before fitting pseudotime with Slingshot or Monocle3.

## When NOT to use

- When the target trajectory method does not operate on a two-dimensional representation — Escort only considers trajectory inference approaches using two-dimensional representations
- When you want to pre-select a single universally best dimension reduction method — the benchmark shows no dimension reduction algorithm or HVG percentage shows a clear performance advantage across datasets
- When prior cell type information is available and clustering should be done with SC3 instead of scLCA as the clustering backbone

## Inputs

- Normalized single-cell expression matrix (Seurat LogNormalize-normalized, e.g. SingleCellExperiment) for datasets such as Escort's exampleData_linear or the six publicly available simulated datasets
- Cell cluster assignments and optimal cluster number from scLCA or SC3
- Highly variable gene subsets (top 20%, 40%, or 100% of genes selected via scran modelGeneVar/getTopHVGs)

## Outputs

- Two-dimensional UMAP embedding (R umap package output)
- Per-embedding connectivity classification (recommended / not recommended) written to escort_connectivity_results.csv
- Trajectory accuracy metrics: Kendall rank correlation and mean squared error of inferred versus ground-truth pseudotime (if trajectories are fitted)

## How to apply

Run UMAP on the Seurat LogNormalize-normalized expression matrix (using the R umap package, v0.2.10.0, directly on normalized data with default settings, no PCA pre-step). First cluster cells — with scLCA, or SC3 when prior cell type information exists — and use Escort to compute the N x N Manhattan distance matrix and Jaccard index scores from within- versus between-cluster distance distributions, recalculating for the top 20% highest-Jaccard cells per cluster; declare two clusters disconnected unless more than 10 cells per cluster meet the 0.3 cutoff. Re-apply the same connectivity check to the UMAP embedding: any cluster that was connected in the original space but newly disconnected in the embedding is immediately classified not-recommended (score <= 0 on Escort's benchmark scale). Then optionally fit trajectories with Slingshot and Monocle3 and assess accuracy with Kendall rank correlation (threshold > 0.85) and MSE (< 0.01). Because performance is dataset-specific, do not assume UMAP outperforms t-SNE or MDS — evaluate it per dataset alongside varying HVG proportions (20/40/100%).

## Related tools

- **Escort** (Framework that evaluates the UMAP embedding's suitability for trajectory inference via disconnected-cluster detection and benchmark scoring (range -inf to 2, scores > 0 recommended)) — https://github.com/xiaorudong/Escort
- **umap** (R package (v0.2.10.0) used to generate UMAP embeddings directly on normalized data with default settings)
- **Slingshot** (Trajectory/pseudotime inference method fitted on the UMAP embedding to measure accuracy (Kendall correlation, MSE))
- **Monocle3** (Alternative trajectory inference method fitted on the UMAP embedding for the benchmark comparison)
- **Rtsne** (Generates the comparison t-SNE embedding (initial PCA retaining top 50 dimensions) against which UMAP is benchmarked)

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort); umap_emb <- umap::umap(norm_expr)$layout
```

## Evaluation signals

- An embedding is accepted as reliable only if disconnected clusters identified in the original high-dimensional space (Jaccard 0.3 cutoff, >10 cells per cluster) do not reappear as disconnected in the UMAP embedding
- Escort benchmark score for the UMAP embedding exceeds 0 (scores range from negative infinity to 2)
- If trajectories are fitted, Kendall rank correlation > 0.85 and MSE < 0.01 flag the embedding as 'good', matching the benchmark calibration criterion
- escort_connectivity_results.csv contains per-embedding connectivity classifications with disconnected or unexpectedly grouped small UMAP clusters counted
- No cluster that was connected pre-embedding is newly disconnected post-embedding

## Limitations

- UMAP frequently generated embeddings that exhibited disconnected clusters or grouped small clusters together in unexpected ways, disqualifying them as non-recommended embeddings
- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated
- Escort only considers trajectory inference approaches that use two-dimensional representations
- No single dimension reduction technique or HVG percentage consistently yields more accurate trajectories — performance is highly dataset-specific, so UMAP cannot be pre-selected as universally best

## Evidence

- [results] In the analyzed scRNA-seq datasets, UMAP frequently generated embeddings that exhibited disconnected clusters or grouped small clusters together in unexpected ways, disqualifying them as non-recommended embeddings.: [evidence span withheld — anchor status: not_found]
- [methods] Embedding generation method: [evidence span withheld — anchor status: not_found]
