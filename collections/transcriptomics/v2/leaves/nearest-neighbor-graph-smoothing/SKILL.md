---
name: nearest-neighbor-graph-smoothing
description: Use when scoring sparse, noisy scRNA-seq count data with bulk-style gene
  set methods (e.g. singscore, ssGSEA, summed_up) and per-cell signal is too weak
  — e.g. signature genes missed or AUC degraded by downsampling.
when_to_use_negative:
- 'smooth_mode=''off'' is the explicit bypass: when the analysis requires the original
  unsmoothed counts (e.g. as the no-smoothing baseline condition), do not apply the
  graph multiplication'
- Scoring bulk RNA-seq or already-aggregated pseudobulk profiles, where per-cell sparsity
  is not the limiting issue and smoothing offers no benefit
- When the analysis requires cell-level resolution of raw counts rather than neighbor-averaged
  profiles (smoothing shares gene measure information across a transcriptomic neighborhood)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: gssnng
  role: Implements nearest neighbor graph smoothing (smooth_mode, recompute_neighbors)
    and downstream gene set scoring on the smoothed matrix
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Loads AnnData/h5ad inputs, provides sc.datasets.pbmc3k_processed(), and supplies
    the louvain groupby annotations used to construct per-group NNGs
- name: AnnData
  role: Container for counts, obs group labels, and smoothed matrices in obsm['X_smooth'];
    per-group objects merged with anndata.concat
- name: Python
  role: Runtime language; scipy sparse CSR matrix multiplication performs the M =
    A·X transform
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  - task_007
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:4cdd748af7c45c206479377f6dff60075c74dccc43354672b6f70257a1c0de33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nearest-neighbor-graph-smoothing

## Summary

Builds a nearest neighbor graph of cells within user-defined groups and computes the smoothed expression matrix M = A·X, where A is a binary adjacency or weighted connectivity matrix, producing mini-pseudobulk profiles per cell that reduce single-cell sparsity and technical noise before gene set scoring.

## When to use

Use when scoring sparse, noisy scRNA-seq count data with bulk-style gene set methods (e.g. singscore, ssGSEA, summed_up) and per-cell signal is too weak — e.g. signature genes missed or AUC degraded by downsampling.

## When NOT to use

- smooth_mode='off' is the explicit bypass: when the analysis requires the original unsmoothed counts (e.g. as the no-smoothing baseline condition), do not apply the graph multiplication
- Scoring bulk RNA-seq or already-aggregated pseudobulk profiles, where per-cell sparsity is not the limiting issue and smoothing offers no benefit
- When the analysis requires cell-level resolution of raw counts rather than neighbor-averaged profiles (smoothing shares gene measure information across a transcriptomic neighborhood)

## Inputs

- Scanpy AnnData object (h5ad file or sc.datasets.pbmc3k_processed()) with expression values in adata.X
- groupby column label in adata.obs (e.g. 'louvain') or a dict specifying one group
- recompute_neighbors integer (0 = reuse existing graph; >0 = rebuild KNN within each group, e.g. 4, 8, 11, 32)
- smooth_mode choice: 'adjacency', 'connectivity', or 'off'

## Outputs

- Smoothed cell-by-gene expression matrix M = A·X as a scipy sparse CSR matrix (mini-pseudobulk profile per cell)
- Smoothed AnnData with the matrix stored in obsm['X_smooth'] and per-group objects concatenated via anndata.concat (smoothed_adata.h5ad)
- Per-cell, per-group gene set scores (columns in adata.obsm['gssnng_estimate']) computed on smoothed data

## How to apply

Load the AnnData (e.g. sc.datasets.pbmc3k_processed() or an h5ad via scanpy) and access expression from adata.X. For each groupby group (e.g. 'louvain' or cluster × sample), build a KNN graph, choosing smooth_mode='adjacency' (all neighbors weighted equally, all 1s) or smooth_mode='connectivity' (closer neighbors weighted more), with recompute_neighbors set to an integer >0 (e.g. 4, 8, 11, or 32) to rebuild neighbors within each group, or 0 to reuse the existing graph. Compute the smoothed matrix as a scipy sparse (CSR) matrix multiplication M = A·X so each cell receives a neighbor-averaged mini-pseudobulk profile, optionally storing it in obsm['X_smooth'] and concatenating per-group objects with anndata.concat. Set smooth_mode='off' to bypass smoothing and keep the original X as the unsmoothed control. In gssnng's score_cells.with_gene_sets, smoothing and scoring are combined via the smooth_mode and recompute_neighbors parameters; score the smoothed and unsmoothed versions and compare (e.g. geneset_overlap fraction, AUC) to confirm smoothing improves detection.

## Related tools

- **gssnng** (Implements nearest neighbor graph smoothing (smooth_mode, recompute_neighbors) and downstream gene set scoring on the smoothed matrix) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Loads AnnData/h5ad inputs, provides sc.datasets.pbmc3k_processed(), and supplies the louvain groupby annotations used to construct per-group NNGs)
- **AnnData** (Container for counts, obs group labels, and smoothed matrices in obsm['X_smooth']; per-group objects merged with anndata.concat)
- **Python** (Runtime language; scipy sparse CSR matrix multiplication performs the M = A·X transform)

## Examples

```
from gssnng import smoothing; smoothing.smooth_adata(adata, groupby='louvain', smooth_mode='connectivity', recompute_neighbors=32); adata.write('smoothed_adata.h5ad')
```

## Evaluation signals

- geneset_overlap on pbmc3k B cells: K=8 or K=32 connectivity smoothing should roughly double the number of B.cell.naive (LM22) signature genes detected versus smooth_mode='off'
- LPS phagocyte summed_up AUC: unsmoothed ≈ 0.95; any smoothed condition (K=4/8/32) should reach ≈ 1.0 with smoothed AUC ≥ unsmoothed
- Downsampling robustness: with recompute_neighbors=32, singscore B-cell-vs-non-B AUC stays above 0.95 until retained count fraction drops below 10%, while unsmoothed AUC drops immediately
- Injury signature AUCs must improve with smoothing for each scorer (RBO 0.62→0.71; ssGSEA 0.69→0.8)
- Schema check: per-group smoothed matrices stored in obsm['X_smooth'], concatenated output writes a valid h5ad, and unsmoothed (smooth_mode='off') scores equal scores on the original X

## Limitations

- Smoothing shares gene measure information across a transcriptomic neighborhood, so per-cell raw-count resolution is intentionally lost in favor of mini-pseudobulk profiles
- The nearest neighbor graph must be rebuilt within each groupby group (recompute_neighbors > 0) or results may reflect cross-group edges; choosing K trades local fidelity against noise suppression
- smooth_mode choice matters: 'adjacency' weights all neighbors equally while 'connectivity' weights close neighbors more — an incorrect mode can blur distinct populations

## Evidence

- [methods] The smoothed cell-by-gene expression matrix is computed by matrix multiplication AX=M, where A is a binary adjacency matrix or a weighted matrix of connectivities from the KNN graph: [evidence span withheld — anchor status: not_found]
- [readme] adjacency vs connectivity: "Smoothing can be performed using either the adjacency matrix (all 1s) or the weighted graph to give less weight to more distant cells."
- [readme] recompute_neighbors semantics: [evidence span withheld — anchor status: not_found]
- [readme] smooth_mode off bypass: [evidence span withheld — anchor status: not_found]
- [abstract] rationale for smoothing: [evidence span withheld — anchor status: not_found]
- [intro] mini-pseudobulk framing: "This essentially creates mini-pseudobulk expression profiles for each cell, which can be scored by using single sample gene set scoring methods often associated with bulk RNA-seq."
- [readme] groupwise graph construction: "Nearest neighbor graphs (NNG) are constructed based on user defined groups (see the 'groupby' parameter below)."
