---
name: sparse-matrix-multiplication
description: Use when per-cell scRNA-seq count data (e.g. a Scanpy AnnData h5ad) are
  sparse and noisy and a per-cell mini-pseudobulk profile is needed before gene set
  scoring with methods like ssGSEA, singscore, or mean_z.
when_to_use_negative:
- The input is bulk RNA-seq data with no cell-to-cell neighborhood graph — smoothing
  was designed to remedy sparsity and technical noise specific to single-cell measures
- The AnnData has no precomputed KNN graph in adata.obsp and neighbors cannot be recomputed
  within the defined groups
- The analysis requires scores on raw per-cell counts (smooth_mode='off') rather than
  neighborhood-smoothed profiles
tools:
- name: gssnng
  role: Provides smooth_mode ('adjacency', 'connectivity', 'off'), groupby, and recompute_neighbors
    parameters that drive the AX=M smoothing computation
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Reads the AnnData h5ad input and provides the nearest-neighbor graph (e.g.
    from pbmc3k_processed) used as matrix A
  repo: https://github.com/IlyaLab/gssnng
- name: AnnData
  role: Container format whose obsm['X_smooth'] slot stores the smoothed matrix; per-group
    objects are merged with anndata.concat
  repo: https://github.com/IlyaLab/gssnng
- name: scipy
  role: Provides sparse (CSR) matrix multiplication used to compute M = A·X memory-efficiently
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:5688f6a1554bf15a4da0e32a042fb7075c64d5e3ecbe3d4053a68e9fc126f3dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# sparse-matrix-multiplication

## Summary

Computes the smoothed single-cell expression matrix M = A·X by multiplying a KNN graph matrix (binary adjacency or distance-weighted connectivities) against the count matrix X using scipy sparse operations, yielding neighbor-averaged mini-pseudobulk profiles that counteract sparsity and technical noise prior to gene set scoring.

## When to use

Use when per-cell scRNA-seq count data (e.g. a Scanpy AnnData h5ad) are sparse and noisy and a per-cell mini-pseudobulk profile is needed before gene set scoring with methods like ssGSEA, singscore, or mean_z.

## When NOT to use

- The input is bulk RNA-seq data with no cell-to-cell neighborhood graph — smoothing was designed to remedy sparsity and technical noise specific to single-cell measures
- The AnnData has no precomputed KNN graph in adata.obsp and neighbors cannot be recomputed within the defined groups
- The analysis requires scores on raw per-cell counts (smooth_mode='off') rather than neighborhood-smoothed profiles

## Inputs

- Scanpy AnnData object (h5ad) with a count matrix in adata.X
- KNN graph from adata.obsp (binary adjacency matrix or distance-weighted connectivities)
- groupby column in adata.obs (e.g. 'louvain') defining neighborhoods
- smooth_mode parameter ('adjacency', 'connectivity', or 'off')
- recompute_neighbors integer (0 to skip, >0 to recompute N neighbors per group)

## Outputs

- Smoothed cell-by-gene matrix M = A·X as a scipy sparse (CSR) matrix
- AnnData with smoothed counts stored in obsm['X_smooth']
- Concatenated smoothed AnnData written to smoothed_adata.h5ad

## How to apply

Load the AnnData (e.g. sc.datasets.pbmc3k_processed()) and access raw counts in adata.X. Build or reuse the nearest neighbor graph, recomputing neighbors within each group (groupby='louvain', recompute_neighbors=11 in the gssnng smoothing example) to avoid mixing cells across clusters or samples. Choose smooth_mode: 'adjacency' weights all neighbors equally (all 1s), 'connectivity' gives closer neighbors more weight, and 'off' bypasses multiplication so the smoothed matrix equals the original X. Compute M = A·X as a scipy CSR sparse matrix multiplication so each cell's row becomes the neighbor-weighted average of counts (a mini-pseudobulk profile), then store the result in obsm['X_smooth']. Finally, concatenate per-group AnnData objects with anndata.concat and write the smoothed object to smoothed_adata.h5ad.

## Related tools

- **gssnng** (Provides smooth_mode ('adjacency', 'connectivity', 'off'), groupby, and recompute_neighbors parameters that drive the AX=M smoothing computation) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Reads the AnnData h5ad input and provides the nearest-neighbor graph (e.g. from pbmc3k_processed) used as matrix A) — https://github.com/IlyaLab/gssnng
- **AnnData** (Container format whose obsm['X_smooth'] slot stores the smoothed matrix; per-group objects are merged with anndata.concat) — https://github.com/IlyaLab/gssnng
- **scipy** (Provides sparse (CSR) matrix multiplication used to compute M = A·X memory-efficiently)

## Examples

```
import scanpy as sc; q = sc.datasets.pbmc3k_processed(); M = q.obsp['connectivities'] @ q.X  # smoothed AX=M via scipy sparse CSR
```

## Evaluation signals

- Each row of M equals the neighbor-weighted average of count rows of X under A: with smooth_mode='adjacency', each smoothed cell is the unweighted mean of its neighbors' counts
- With smooth_mode='off', the stored obsm['X_smooth'] is identical to the original adata.X (bypass invariant)
- The output matrix is a scipy sparse CSR object and its shape matches (n_cells, n_genes) of the input X
- Every cell in the input AnnData appears in the smoothed output after anndata.concat across all groupby categories, with obsm['X_smooth'] populated
- Smoothed profiles should show reduced sparsity (more non-zero gene entries per cell) relative to the raw count matrix

## Limitations

- smooth_mode choice materially changes results: 'adjacency' weights all neighbors equally while 'connectivity' weights close neighbors more, so distant cells contribute differently
- Neighbors should be recomputed within each user-defined group (recompute_neighbors > 0); otherwise the graph can mix cells across clusters or samples
- Sparsity and technical noise of single-cell expression measures are the motivation for smoothing, but the transformation changes per-cell values into neighborhood-informed profiles, which should not be interpreted as raw measurements

## Evidence

- [abstract] abstract: NN-graph smoothing purpose: "we use the nearest neighbor graph of cells for matrix smoothing to produce high quality gene set scores on a per-cell, per-group, level"
- [intro] intro: mini-pseudobulk rationale: "This essentially creates mini-pseudobulk expression profiles for each cell, which can be scored by using single sample gene set scoring methods often associated with bulk RNA-seq."
- [methods] methods: smoothing on NNG: "By operating on the nearest neighbor graph, we create a mini-pseudobulk expression profile of neighbors for each cell, thus smoothing the count data."
- [readme] readme: adjacency vs connectivity weighting: "Smoothing can be performed using either the adjacency matrix (all 1s) or the weighted graph to give less weight to more distant cells."
- [readme] readme: smooth_mode semantics: "`adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
