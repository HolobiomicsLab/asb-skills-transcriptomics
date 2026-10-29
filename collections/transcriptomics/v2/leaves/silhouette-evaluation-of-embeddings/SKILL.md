---
name: silhouette-evaluation-of-embeddings
description: Use when comparing 2D embeddings of scRNA-seq data for rare cell type
  separation, where a silhouette index computed over 100+ replicated runs (e.g., EDGE
  mean 0.851 vs UMAP 0.801 vs t-SNE 0.838 on PBMC) is needed as an objective metric.
when_to_use_negative:
- Input is not a 2D cell embedding derived from a normalized count matrix (dat_log)
  — silhouette on unnormalized or raw counts does not benchmark embedding quality
- The dataset lacks labeled rare cell type annotations, since the silhouette index
  must be computed per rare cell type (e.g., Jurkat cells, dendritic cells, megakaryocytes)
- You need feature gene identification rather than embedding quality assessment —
  use EDGE's endr feature gene output instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: EDGE
  role: Produces the candidate 2D embedding (via endr) whose rare-cell-type silhouette
    index is benchmarked against UMAP and t-SNE
  repo: github.com/shawnstat/EDGE
- name: UMAP
  role: Baseline embedding method run on dat_log with random_state=2020 and 15 nearest
    neighbors, matched to EDGE's neighborhood size
- name: t-SNE
  role: Baseline embedding method run via Rtsne(dat_log) with random seed 2020 for
    comparison of silhouette indices
- name: RSpectra
  role: Dependency of EDGE used for the large-scale eigenvalue decomposition underlying
    spectral embedding before silhouette evaluation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:f21bcece16bf67d05ba9ad61fab1a9fd57dcf6c850b5a799ab6376660c43d4e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - RSpectra
  - UMAP
  - t-SNE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# silhouette-evaluation-of-embeddings

## Summary

Quantitatively benchmarks 2D embeddings (EDGE vs UMAP vs t-SNE) of scRNA-seq data by computing the silhouette index of rare cell types (e.g., Jurkat cells, dendritic cells, megakaryocytes) over many replicated runs, testing whether EDGE separates rare populations better than conventional methods.

## When to use

Use when comparing 2D embeddings of scRNA-seq data for rare cell type separation, where a silhouette index computed over 100+ replicated runs (e.g., EDGE mean 0.851 vs UMAP 0.801 vs t-SNE 0.838 on PBMC) is needed as an objective metric.

## When NOT to use

- Input is not a 2D cell embedding derived from a normalized count matrix (dat_log) — silhouette on unnormalized or raw counts does not benchmark embedding quality
- The dataset lacks labeled rare cell type annotations, since the silhouette index must be computed per rare cell type (e.g., Jurkat cells, dendritic cells, megakaryocytes)
- You need feature gene identification rather than embedding quality assessment — use EDGE's endr feature gene output instead

## Inputs

- Raw scRNA-seq count matrix (cells x genes), e.g., PBMC dataset with four rare cell types labeled
- Cell type label vector (labs) for coloring embeddings and subsetting rare types for silhouette computation
- Library-size-normalized, log2-transformed matrix dat_log (log2(norm+1))

## Outputs

- Per-replication silhouette index values for the four rare cell types under each method (EDGE, UMAP, t-SNE)
- Averaged silhouette index table across >100 replications (e.g., EDGE 0.851, UMAP 0.801, t-SNE 0.838 with SDs)
- Three-panel scatter plot of 2D embeddings (EDGE, UMAP, t-SNE) colored by cell labels

## How to apply

Normalize the raw count matrix by scaling each cell's library size to the median total count and log2-transform via log2(norm+1) to obtain dat_log. Generate each embedding under a fixed random seed (2020): run EDGE's endr(dat_log, custom_defs) with n_wl=1000 weak learners, n_dm=10 genes per weak learner, n_neigs=15, opt=TRUE, and hash table size H=101107; run UMAP on dat_log with random_state=2020 and 15 nearest neighbors to match EDGE's neighborhood size; run t-SNE via Rtsne(dat_log) after set.seed(2020). For each replication, compute the silhouette index of the four rare PBMC cell types in each 2D embedding, then average over more than 100 replications. Rationale: silhouette index directly measures how compactly and separately rare populations are embedded, which is the property EDGE claims to improve over t-SNE and UMAP. Validate by checking the averaged silhouettes match EDGE 0.851 (SD 0.045), UMAP 0.801 (SD 0.012), t-SNE 0.838 (SD 0.008) within tolerance, and that the three-panel scatter figure (colored by cell labels) is produced.

## Related tools

- **EDGE** (Produces the candidate 2D embedding (via endr) whose rare-cell-type silhouette index is benchmarked against UMAP and t-SNE) — github.com/shawnstat/EDGE
- **UMAP** (Baseline embedding method run on dat_log with random_state=2020 and 15 nearest neighbors, matched to EDGE's neighborhood size)
- **t-SNE** (Baseline embedding method run via Rtsne(dat_log) with random seed 2020 for comparison of silhouette indices)
- **RSpectra** (Dependency of EDGE used for the large-scale eigenvalue decomposition underlying spectral embedding before silhouette evaluation)

## Examples

```
custom_defs$n_wl <- 1000; custom_defs$n_neigs <- 15; custom_defs$seed <- 2020; simu.endr <- endr(dat_log,custom_defs); simu.umap <- umap(dat_log,random_state=2020); set.seed(2020); simu.tsne <- Rtsne(dat_log)
```

## Evaluation signals

- Averaged silhouette indices match reported values within tolerance: EDGE 0.851 (SD 0.045), UMAP 0.801 (SD 0.012), t-SNE 0.838 (SD 0.008) over >100 replications on PBMC
- EDGE's mean rare-cell-type silhouette exceeds both UMAP (0.801) and t-SNE (0.838) across replications
- Three-panel figure is produced with side-by-side EDGE, UMAP, and t-SNE scatter plots colored by cell labels
- All methods use consistent random seed 2020 and matched neighborhood size (15 nearest neighbors for EDGE and UMAP)
- Input matrix dat_log is correctly computed as log2 of library-size-normalized counts (each cell scaled to the median total count)

## Limitations

- EDGE's silhouette advantage is modest in magnitude (0.851 vs 0.838 for t-SNE) and silhouettes over >100 replications are needed for reliable comparison
- Large weak-learner counts and hash table sizes improve EDGE results but increase memory use; the README's reduced hash table size H=101107 (vs optimal 1,017,881) works 'most of the time'
- Stochastic dependence on seeds: both UMAP (random_state=2020) and t-SNE (set.seed(2020)) require explicit seed setting for reproducible silhouette comparison

## Evidence

- [results] Over 100 replications on the PBMC dataset, EDGE achieved a mean silhouette index of 0.851 (SD 0.045) for the four rare cell types, higher than UMAP (0.801, SD 0.012) and t-SNE (0.838, SD 0.008).: [evidence span withheld — anchor status: not_found]
- [readme] We reduce the hash table size to 101,107 in the example. Most of the time, this works well and uses less memory.: "We reduce the hash table size to 101,107 in the example. Most of the time, this works well and uses less memory."
- [discussion] In all three real datasets, EDGE shows more compactly projected structures compared with t-SNE and UMAP: "In all three real datasets, EDGE shows more compactly projected structures compared with t-SNE and UMAP"
- [discussion] When there exist rare cell types, EDGE provides a neat separation between rare and common cell types: "When there exist rare cell types, EDGE provides a neat separation between rare and common cell types"
