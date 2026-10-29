---
name: silhouette-index-cluster-evaluation
description: Use when comparing 2D embeddings (e.g., EDGE, t-SNE, UMAP) of labeled
  single-cell RNA-seq data and you need a quantitative, replication-averaged measure
  of how compactly and separately each cell type is projected.
when_to_use_negative:
- The input is raw counts that have not been library-size normalized and log2(x+1)
  transformed — silhouette rankings are only comparable when all methods embed the
  same normalized matrix
- The cell-type labels are unavailable or unreliable, since silhouette must be computed
  against known groups
- Only a single embedding per method exists without replication, since Monte Carlo
  tolerance on the reported means (e.g., 0.621 vs 0.501) cannot be assessed
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: EDGE
  role: Primary embedding method whose silhouette scores are compared; run via endr(dat_log,
    custom_defs) with endr_defs parameters
  repo: github.com/shawnstat/EDGE
- name: t-SNE
  role: Baseline embedding (Rtsne with seed 2020) whose silhouette index (0.501) is
    compared against EDGE and UMAP
- name: UMAP
  role: Baseline embedding (umap(dat_log, random_state=2020) with 15 nearest neighbors)
    whose silhouette index (0.597) is compared against EDGE
- name: RSpectra
  role: Required dependency for EDGE's large-scale eigenvalue decomposition during
    spectral embedding
  repo: github.com/shawnstat/EDGE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:40526b945c3d10c15e917a8e2f3c817ff6185ba1d5623125a9ef7f44aa4adc48
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - RSpectra
  - Rcpp
  - UMAP
  - t-SNE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# silhouette-index-cluster-evaluation

## Summary

Evaluates how well a low-dimensional embedding preserves cell-type structure by computing the silhouette index of each cell type in the 2D embedding, averaged over many replications. Used in the EDGE paper to rank EDGE, UMAP, and t-SNE on the 3005-cell mouse cortex/hippocampus dataset.

## When to use

Use when comparing 2D embeddings (e.g., EDGE, t-SNE, UMAP) of labeled single-cell RNA-seq data and you need a quantitative, replication-averaged measure of how compactly and separately each cell type is projected.

## When NOT to use

- The input is raw counts that have not been library-size normalized and log2(x+1) transformed — silhouette rankings are only comparable when all methods embed the same normalized matrix
- The cell-type labels are unavailable or unreliable, since silhouette must be computed against known groups
- Only a single embedding per method exists without replication, since Monte Carlo tolerance on the reported means (e.g., 0.621 vs 0.501) cannot be assessed

## Inputs

- Log-normalized single-cell RNA-seq matrix (cells x genes, e.g., 3005 mouse cortex/hippocampus cells), from library-size scaling to the median plus log2(x+1)
- Per-cell type labels for silhouette computation
- 2D embeddings from EDGE (endr), UMAP (umap), and t-SNE (Rtsne) computed on the same normalized matrix
- Replication specification (100 replications, seed 2020)

## Outputs

- Per-method averaged silhouette indices per cell type (e.g., 0.621 EDGE, 0.597 UMAP, 0.501 t-SNE) saved as silhouette_indices.csv
- Side-by-side 2D subspace scatter plots of EDGE, UMAP, and t-SNE embeddings colored by cell type (fig4a_c_embeddings.png)

## How to apply

Normalize the count matrix by scaling each cell's library size to the median and applying log2(x+1), then generate the 2D embeddings to be compared (EDGE via endr() with n_wl=1000 weak learners, n_dm=10 genes per learner, n_neigs=15, seed=2020, hash table H=101107; UMAP with 15 nearest neighbors and random_state=2020; Rtsne with seed 2020). For each embedding, compute the silhouette index of every cell type against the known cell-type labels, then repeat the full embedding procedure over 100 replications and average per-method scores. In the mouse brain benchmark this produced mean silhouette indices of 0.621 (SD 0.085) for EDGE, 0.597 (0.025) for UMAP, and 0.501 (0.017) for t-SNE. Per-method averages are saved to silhouette_indices.csv and visualized as side-by-side colored 2D scatter plots (Figs. 4a-4c).

## Related tools

- **EDGE** (Primary embedding method whose silhouette scores are compared; run via endr(dat_log, custom_defs) with endr_defs parameters) — github.com/shawnstat/EDGE
- **t-SNE** (Baseline embedding (Rtsne with seed 2020) whose silhouette index (0.501) is compared against EDGE and UMAP)
- **UMAP** (Baseline embedding (umap(dat_log, random_state=2020) with 15 nearest neighbors) whose silhouette index (0.597) is compared against EDGE)
- **RSpectra** (Required dependency for EDGE's large-scale eigenvalue decomposition during spectral embedding) — github.com/shawnstat/EDGE

## Examples

```
simu.endr <- endr(dat_log,custom_defs); simu.umap <- umap(dat_log,random_state=2020); set.seed(2020); simu.tsne <- Rtsne(dat_log)
```

## Evaluation signals

- Averaged silhouette indices over 100 replications reproduce the reported values 0.621 (EDGE), 0.597 (UMAP), and 0.501 (t-SNE) within Monte Carlo tolerance
- EDGE's mean silhouette index exceeds both baselines (0.621 > 0.597 > 0.501) with standard deviations of 0.085, 0.025, and 0.017 respectively
- Per-method averages are written to silhouette_indices.csv with one row per method over the 100 replications
- Side-by-side scatter plots (fig4a_c_embeddings.png) show EDGE separating cell types, including rare populations, more compactly than t-SNE and UMAP
- All three methods embed the identical log-normalized matrix (median-scaled library sizes, log2(x+1)) so scores are directly comparable

## Limitations

- EDGE results depend on weak-learner and hash-table settings; larger n_wl and H give better results but require large memory (optimal H is 1,017,881; the example uses 101,107)
- Silhouette values carry Monte Carlo variability (EDGE SD 0.085), so single replications may not rank methods stably
- The metric rewards compact separation of labeled cell types and does not by itself verify global/hierarchical structure preservation, which the paper assesses separately for t-SNE

## Evidence

- [results] On the mouse brain dataset, EDGE achieved the highest mean silhouette index of all cell types (0.621, SD 0.085) over 100 replications, compared with 0.597 (0.025) for UMAP and 0.501 (0.017) for t-SNE.: [evidence span withheld — anchor status: not_found]
- [discussion] In all three real datasets, EDGE shows more compactly projected structures compared with t-SNE and UMAP: "In all three real datasets, EDGE shows more compactly projected structures compared with t-SNE and UMAP"
- [discussion] The final similarity matrix S is calculated by averaging the corresponding similarity scores from all voters: "The final similarity matrix S is calculated by averaging the corresponding similarity scores from all voters"
- [readme] The optimal hash table size is 1,017,881. We reduce the hash table size to 101,107 in the example.: "The optimal hash table size is 1,017,881. We reduce the hash table size to 101,107 in the example."
