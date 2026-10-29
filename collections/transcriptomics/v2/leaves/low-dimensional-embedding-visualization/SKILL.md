---
name: low-dimensional-embedding-visualization
description: Use when comparing 2D embeddings of scRNA-seq count matrices across methods
  (EDGE vs UMAP vs t-SNE), especially to assess separation of rare cell types against
  dominant populations.
when_to_use_negative:
- The goal is feature gene identification rather than embedding — EDGE's gene-selection
  output, not the 2D plot, is the deliverable in that case
- Input is already a low-dimensional embedding or a normalized/log-transformed matrix
  for which library-size normalization should not be re-applied
- Global hierarchical structure recovery is the primary goal and only t-SNE is considered
  — the article notes t-SNE 'may suffer from losing the global geometry pattern/property
  of the data'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: EDGE
  role: Ensemble dimensionality reduction producing the primary 2D embedding via weak-learner
    similarity search, spectral embedding, and stochastic gradient descent optimization
  repo: github.com/shawnstat/EDGE
- name: UMAP
  role: Baseline 2D embedding on dat_log with random_state=2020 and 15 nearest neighbors
    to match EDGE's neighborhood size
- name: t-SNE
  role: Baseline 2D embedding via Rtsne(dat_log) with random seed 2020 for side-by-side
    comparison
- name: RSpectra
  role: Performs the large-scale eigenvalue decomposition required for EDGE's spectral
    embedding step
- name: Splatter
  role: Simulates scRNA-seq test data (e.g., 500 cells, 200 genes, two groups with
    2% minor group) to exercise the embedding workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:ff14ecdc5b7600df11ab121a5a9a966a4d7f126c9b0a1873b8be6c48f25ffc52
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - RSpectra
  - Splatter
  - UMAP
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# low-dimensional-embedding-visualization

## Summary

Produce side-by-side two-dimensional scatter plot embeddings of single-cell RNA-seq data using EDGE, UMAP, and t-SNE with matched seeds and neighborhood sizes, so that rare cell type separation can be visually and quantitatively compared (e.g., via silhouette index).

## When to use

Use when comparing 2D embeddings of scRNA-seq count matrices across methods (EDGE vs UMAP vs t-SNE), especially to assess separation of rare cell types against dominant populations.

## When NOT to use

- The goal is feature gene identification rather than embedding — EDGE's gene-selection output, not the 2D plot, is the deliverable in that case
- Input is already a low-dimensional embedding or a normalized/log-transformed matrix for which library-size normalization should not be re-applied
- Global hierarchical structure recovery is the primary goal and only t-SNE is considered — the article notes t-SNE 'may suffer from losing the global geometry pattern/property of the data'

## Inputs

- Raw scRNA-seq count matrix (cells x genes), e.g. from Splatter simulation or real datasets such as Jurkat and PBMC
- Cell type labels vector (e.g. labs from colData of a splatSimulate object) used to color scatter plots
- Cell library sizes (column sums of the count matrix) for median-scaling normalization

## Outputs

- Three 2D embedding objects: simu.endr (EDGE), simu.umap$layout (UMAP), simu.tsne$Y (t-SNE)
- A 1x3 panel scatter plot figure colored by cell labels
- Per-replication silhouette index values for rare cell types (e.g., mean EDGE 0.851, UMAP 0.801, t-SNE 0.838 on PBMC)

## How to apply

Normalize the raw count matrix by scaling each cell's library size to the median total count and log2-transform via log2(norm+1) to obtain dat_log. Configure EDGE via endr_defs with n_wl=1000 weak learners, n_dm=10 genes per weak learner, n_neigs=15 nearest neighbors, opt=TRUE for low-dimensional embedding optimization, seed=2020, and reduced hash table size H=101107 (the README notes the optimal table is 1,017,881 but 101,107 usually works well with less memory), then run endr(dat_log, custom_defs). Run UMAP on the same dat_log with random_state=2020 and 15 nearest neighbors to match EDGE's neighborhood size, and t-SNE via Rtsne(dat_log) with set.seed(2020). Plot all three embeddings as scatter plots in a 1x3 panel colored by cell labels, and optionally quantify rare-cell-type separation by averaging silhouette indices over >100 replications (reported: EDGE 0.851 SD 0.045, UMAP 0.801 SD 0.012, t-SNE 0.838 SD 0.008 on PBMC rare types).

## Related tools

- **EDGE** (Ensemble dimensionality reduction producing the primary 2D embedding via weak-learner similarity search, spectral embedding, and stochastic gradient descent optimization) — github.com/shawnstat/EDGE
- **UMAP** (Baseline 2D embedding on dat_log with random_state=2020 and 15 nearest neighbors to match EDGE's neighborhood size)
- **t-SNE** (Baseline 2D embedding via Rtsne(dat_log) with random seed 2020 for side-by-side comparison)
- **RSpectra** (Performs the large-scale eigenvalue decomposition required for EDGE's spectral embedding step)
- **Splatter** (Simulates scRNA-seq test data (e.g., 500 cells, 200 genes, two groups with 2% minor group) to exercise the embedding workflow)

## Examples

```
custom_defs <- endr_defs; custom_defs$n_wl <- 1000; custom_defs$n_dm <- 10; custom_defs$n_neigs <- 15; custom_defs$opt <- TRUE; custom_defs$seed <- 2020; custom_defs$H <- 101107; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- The 1x3 panel figure renders all three embeddings (EDGE, UMAP, t-SNE) colored by cell labels without errors
- Averaged silhouette indices over >100 PBMC replications match EDGE 0.851 (SD 0.045), UMAP 0.801 (SD 0.012), t-SNE 0.838 (SD 0.008) within tolerance
- EDGE embedding shows rare cell types (e.g., Jurkat cells, dendritic cells, megakaryocytes) more neatly separated from common types than UMAP/t-SNE
- Normalization invariant: each cell's library size equals the median total count before log2(norm+1) transformation
- Reproducibility check: identical seed=2020 across EDGE, UMAP (random_state), and t-SNE (set.seed) yields stable embeddings across re-runs

## Limitations

- Large numbers of weak learners and larger hash table sizes improve results but require large memory; the reduced hash table size 101,107 (vs optimal 1,017,881) usually works well but is a trade-off
- t-SNE may lose the global geometry/hierarchical structure of the data, and UMAP may be inefficient at identifying rare cell types when dominant/common cell types exist
- Version numbers for R, Splatter, Seurat, and RSpectra are not specified; the exact negative-sample count and hyperparameters a and b are not reported

## Evidence

- [readme] Normalize then embed with matched seeds: [evidence span withheld — anchor status: not_found]
- [readme] Reduced hash table size trade-off: "The optimal hash table size is 1,017,881. We reduce the hash table size to 101,107 in the example. Most of the time, this works well and uses less memory."
- [readme] Matching UMAP neighborhood to EDGE: "# UMAP: the number of nearest neighbors is also 15
    simu.umap <- umap(dat_log,random_state=2020)"
- [discussion] Rare vs common cell type separation claim: "When there exist rare cell types, EDGE provides a neat separation between rare and common cell types"
- [results] Silhouette index advantage on PBMC rare types: [evidence span withheld — anchor status: not_found]
