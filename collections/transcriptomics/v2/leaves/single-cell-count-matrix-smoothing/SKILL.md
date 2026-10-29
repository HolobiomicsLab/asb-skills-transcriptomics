---
name: single-cell-count-matrix-smoothing
description: Use when single-cell expression data (Scanpy AnnData .h5ad) is too sparse/noisy
  for per-cell gene set scoring, and a smoothed neighbor-averaged matrix is needed
  before applying methods like singscore or ssGSEA.
when_to_use_negative:
- The input is bulk RNA-seq data or already an aggregated pseudobulk matrix — smoothing
  targets single-cell sparsity and technical noise
- A control/no-smoothing baseline is required — use smooth_mode='off', which bypasses
  the graph multiplication and stores the original X unchanged
- Cell groups are not defined or the KNN graph cannot be built within meaningful groups
  (NNGs are constructed within user-defined groupby groups)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: gssnng
  role: Implements the nearest-neighbor graph smoothing of count matrices (smooth_mode
    adjacency/connectivity/off) on Scanpy AnnData objects
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Loads the .h5ad AnnData, provides datasets (sc.datasets.pbmc3k_processed)
    and neighbor graph machinery; groupby groups processed per cluster
- name: AnnData
  role: Container for the count matrix and smoothed outputs; per-group objects are
    concatenated with anndata.concat and stored as h5ad
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:d2c85a14866b6d96db4d1b6ed8e3bce45990c52aac464f4733c4fb6fc02fed72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# single-cell-count-matrix-smoothing

## Summary

Smooths a sparse, noisy single-cell count matrix by multiplying it with a nearest-neighbor graph matrix (AX = M), producing mini-pseudobulk expression profiles per cell that are robust enough for bulk-style single-sample gene set scoring.

## When to use

Use when single-cell expression data (Scanpy AnnData .h5ad) is too sparse/noisy for per-cell gene set scoring, and a smoothed neighbor-averaged matrix is needed before applying methods like singscore or ssGSEA.

## When NOT to use

- The input is bulk RNA-seq data or already an aggregated pseudobulk matrix — smoothing targets single-cell sparsity and technical noise
- A control/no-smoothing baseline is required — use smooth_mode='off', which bypasses the graph multiplication and stores the original X unchanged
- Cell groups are not defined or the KNN graph cannot be built within meaningful groups (NNGs are constructed within user-defined groupby groups)

## Inputs

- Scanpy AnnData object (.h5ad) with a raw count matrix in .X (e.g. pbmc3k_processed)
- obs column defining groups for graph construction (e.g. 'louvain' clusters)
- smoothing parameters: smooth_mode ('adjacency' | 'connectivity' | 'off') and recompute_neighbors (int, e.g. 11 or 32)

## Outputs

- Smoothed cell-by-gene matrix M = A·X as a scipy sparse (CSR) matrix of neighbor-averaged counts
- AnnData object with smoothed counts stored in obsm (e.g. obsm['X_smooth']), written as .h5ad

## How to apply

Load the AnnData (e.g. sc.datasets.pbmc3k_processed()) and build a nearest-neighbor graph within user-defined groups via groupby (e.g. 'louvain'), recomputing neighbors within each group with recompute_neighbors (e.g. 11 or 32). Choose smooth_mode: 'adjacency' weights all neighbors equally with a binary 0/1 matrix, 'connectivity' weights closer neighbors more using distance-weighted connectivities, and 'off' bypasses smoothing entirely (control condition). Compute the smoothed matrix M = A·X as a scipy sparse (CSR) matrix multiplication so each cell receives the neighbor-averaged mini-pseudobulk profile, and store it (e.g. in obsm['X_smooth']), concatenating per-group AnnData objects with anndata.concat. The rationale is that smoothing shares gene measure information within transcriptomic neighborhoods, remediating sparsity and technical noise before scoring.

## Related tools

- **gssnng** (Implements the nearest-neighbor graph smoothing of count matrices (smooth_mode adjacency/connectivity/off) on Scanpy AnnData objects) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Loads the .h5ad AnnData, provides datasets (sc.datasets.pbmc3k_processed) and neighbor graph machinery; groupby groups processed per cluster)
- **AnnData** (Container for the count matrix and smoothed outputs; per-group objects are concatenated with anndata.concat and stored as h5ad)

## Examples

```
from gssnng.smoothing import smoothed_counts; q = sc.datasets.pbmc3k_processed(); smoothed = smoothed_counts(adata=q, groupby='louvain', smooth_mode='connectivity', recompute_neighbors=11)
```

## Evaluation signals

- With K=32 connectivity smoothing, the AUC for distinguishing naive B cells from other PBMC types stays above 0.95 until the retained count fraction drops below 10% under downsampling, while the smooth_mode='off' control degrades immediately
- The smoothed matrix is a valid scipy sparse (CSR) result of AX with the same gene dimensions as X, and each row is a neighbor-weighted average over the cell's KNN
- smooth_mode='off' yields a matrix identical to the original X (bypass invariant)
- Smoothed per-cell profiles form 'mini-pseudobulk' profiles whose per-gene values lie between the cell's own counts and its neighbors' counts
- Resulting AnnData round-trips to .h5ad and smoothed scores appear as columns in adata.obsm (e.g. 'gssnng_estimate' after scoring)

## Limitations

- Smoothing is performed within user-defined groups (groupby); results depend on sensible group definitions, e.g. cluster or cluster*sample
- Neighbors may need recomputation within each group (recompute_neighbors>0); setting it to 0 reuses an existing graph that may not respect group boundaries
- Smoothing trades single-cell resolution for robustness — for unsmoothed per-cell behavior, smooth_mode='off' must be used as the control

## Evidence

- [abstract] Sparsity/technical noise motivation: [evidence span withheld — anchor status: not_found]
- [intro] Mini-pseudobulk rationale: "This essentially creates mini-pseudobulk expression profiles for each cell, which can be scored by using single sample gene set scoring methods often associated with bulk RNA-seq."
- [readme] Smoothing modes from README: "Smoothing can be performed using either the adjacency matrix (all 1s) or the weighted graph to give less weight to more distant cells."
- [readme] smooth_mode parameter semantics: "`adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
- [methods] Dataset loading from methods: "q = sc.datasets.pbmc3k_processed()"
