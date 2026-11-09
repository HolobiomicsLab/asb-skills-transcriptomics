---
name: rare-cell-type-embedding-separation
description: Use when scRNA-seq data contain dominant/common cell types plus rare
  populations and the goal is a 2D embedding that visibly separates the rare clusters,
  evaluated via silhouette index against t-SNE and UMAP baselines.
when_to_use_negative:
- When the goal is global/hierarchical structure recovery rather than rare-cluster
  separation — t-SNE loses the global geometry pattern, but this skill is scoped to
  rare-type separation
- When no ground-truth cell labels are available, since silhouette index per rare
  cell type and label-colored scatter plots cannot be computed
- When input is simulated data with a minor group far larger than ~2% (e.g., the README's
  15/500-cell minor group) — EDGE's rare-type advantage is specific to highly imbalanced
  populations
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: EDGE
  role: Primary ensemble method producing the 2D embedding that separates rare cell
    types
  repo: github.com/shawnstat/EDGE
- name: UMAP
  role: Baseline embedding with random_state=2020 and 15 nearest neighbors matching
    EDGE's n_neigs
- name: t-SNE
  role: Baseline embedding via Rtsne(dat_log) with seed 2020
- name: RSpectra
  role: Performs the large-scale eigenvalue decomposition needed for EDGE's spectral
    embedding
  repo: github.com/shawnstat/EDGE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:e0cdec23fd1a1326d2f1de6e3c3beb3438fa1058d1bbb953f8c437a216d7f950
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - RSpectra
  - UMAP
  - t-SNE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# rare-cell-type-embedding-separation

## Summary

Benchmark EDGE against UMAP and t-SNE for separating rare cell types (e.g., Jurkat cells, dendritic cells, megakaryocytes) in 2D embeddings of scRNA-seq data, quantified by the silhouette index over 100+ replications.

## When to use

Use when scRNA-seq data contain dominant/common cell types plus rare populations and the goal is a 2D embedding that visibly separates the rare clusters, evaluated via silhouette index against t-SNE and UMAP baselines.

## When NOT to use

- When the goal is global/hierarchical structure recovery rather than rare-cluster separation — t-SNE loses the global geometry pattern, but this skill is scoped to rare-type separation
- When no ground-truth cell labels are available, since silhouette index per rare cell type and label-colored scatter plots cannot be computed
- When input is simulated data with a minor group far larger than ~2% (e.g., the README's 15/500-cell minor group) — EDGE's rare-type advantage is specific to highly imbalanced populations

## Inputs

- Raw scRNA-seq count matrix (cells x genes), e.g., PBMC dataset with rare populations such as dendritic cells and megakaryocytes
- Cell type labels for coloring embeddings and computing silhouette index
- Cell library-size totals (colSums of counts) for median normalization

## Outputs

- Three side-by-side 2D scatter-plot embeddings (EDGE, UMAP, t-SNE) colored by cell labels
- Per-replication silhouette index values for the four rare PBMC cell types per method
- Averaged silhouette index table (EDGE 0.851 SD 0.045; UMAP 0.801 SD 0.012; t-SNE 0.838 SD 0.008)

## How to apply

Normalize the raw count matrix by scaling each cell's library size to the median total count and log2-transform via log2(norm+1) to obtain dat_log. Configure endr_defs with n_wl=1000 weak learners, n_dm=10 selected genes per weak learner, n_neigs=15 nearest neighbors, opt=TRUE for embedding optimization, seed=2020, and reduced hash table size H=101107, then run endr(dat_log, custom_defs). Run UMAP on the same dat_log with random_state=2020 and 15 nearest neighbors to match EDGE's neighborhood size, and t-SNE via Rtsne(dat_log) with seed 2020. Plot the three 2D embeddings side by side as scatter plots colored by cell labels (pch=16, cex=.8, col=labs). Repeat over 100+ replications and compute the silhouette index of the rare cell types for each method; on PBMC, EDGE achieved mean silhouette 0.851 (SD 0.045) versus UMAP 0.801 (SD 0.012) and t-SNE 0.838 (SD 0.008). The rationale is that UMAP may be inefficient in identifying rare cell types when dominant/common cell types exist, while EDGE's ensemble of weak learners learns cell similarities that resolve rare populations.

## Related tools

- **EDGE** (Primary ensemble method producing the 2D embedding that separates rare cell types) — github.com/shawnstat/EDGE
- **UMAP** (Baseline embedding with random_state=2020 and 15 nearest neighbors matching EDGE's n_neigs)
- **t-SNE** (Baseline embedding via Rtsne(dat_log) with seed 2020)
- **RSpectra** (Performs the large-scale eigenvalue decomposition needed for EDGE's spectral embedding) — github.com/shawnstat/EDGE

## Examples

```
custom_defs <- endr_defs; custom_defs$n_wl <- 1000; custom_defs$n_dm <- 10; custom_defs$n_neigs <- 15; custom_defs$opt <- TRUE; custom_defs$seed <- 2020; custom_defs$H <- 101107; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- Averaged silhouette indices for the four rare PBMC cell types match EDGE 0.851, UMAP 0.801, t-SNE 0.838 within tolerance
- A three-panel figure (mfrow=c(1,3)) with EDGE, UMAP, and t-SNE scatter plots colored by cell labels is produced
- EDGE's silhouette SD (0.045) is reported alongside per-method SDs (UMAP 0.012, t-SNE 0.008) across the 100+ replications
- EDGE shows a neat separation between rare and common cell types in the 2D projection, more compact than t-SNE/UMAP
- UMAP's nearest-neighbor count equals EDGE's n_neigs=15 so the baseline is matched

## Limitations

- Large weak-learner counts and hash table sizes improve results but require large memory; the README reduces H from the optimal 1,017,881 to 101,107, which works 'most of the time'
- EDGE's mean silhouette SD (0.045) is larger than UMAP's (0.012) and t-SNE's (0.008), so per-replication rare-type separation is more variable
- t-SNE may lose the global geometry/hierarchical structure of the data, so comparisons are limited to local rare-cluster separation, not global structure
- Exact per-dataset gene-panel size B is only given as a heuristic (<0.05*G); hyperparameters a and b are not reported

## Evidence

- [discussion] EDGE separation of rare types via silhouette: "When there exist rare cell types, EDGE provides a neat separation between rare and common cell types"
- [intro] UMAP weakness with rare types: "However, UMAP may be inefficient in identifying rare cell types when dominant/common cell types exist."
- [readme] README parameter configuration: "The optimal hash table size is 1,017,881. We reduce the hash table size to 101,107 in the example. Most of the time, this works well and uses less memory."
- [readme] README embedding calls: "simu.umap <- umap(dat_log,random_state=2020)"
- [readme] README normalization: "dat_log <- t(log2(dat_norm+1))"
- [discussion] Compact projections claim: "In all three real datasets, EDGE shows more compactly projected structures compared with t-SNE and UMAP"
