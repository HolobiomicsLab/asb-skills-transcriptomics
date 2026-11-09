---
name: umap-embedding-visualization
description: Use when a fitted GEDI model's integrated cell space or sample-specific
  manifold parameters need a 2D UMAP visualization annotated by donor, technology,
  sample, or cell type.
when_to_use_negative:
- The input is raw counts or a SingleCellExperiment without a fitted GEDI model —
  GEDI must be fit first and the integrated/manifold space recovered (svd.gedi or
  Qi) before UMAP is meaningful.
- The goal is producing the integration_benchmark_scores.csv quantitative ranking
  (Luecken-style batch-removal and cell-type-conservation metrics) — UMAP here is
  a qualitative visualization step, not the scored output.
tools:
- name: uwot
  role: Computes the 2D UMAP embedding of the GEDI integrated space or rotated sample-manifold
    coordinates (min_dist = 0.01, euclidean metric)
  repo: https://cran.r-project.org/web/packages/uwot/index.html
- name: GEDI
  role: Provides the fitted model whose latent/manifold parameters are embedded, and
    GEDI::plot_embedding for annotated visualization
  repo: https://github.com/csglab/GEDI
- name: rsvd
  role: GEDI dependency used for randomized SVD underlying latent-space recovery and
    PCA rotation of manifold parameters
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:cc574380a6caa4d6730f9a5a863656289b6437e8c008b24b446ad179b0f50832
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterR
  - DoRothEA
  - GEDI
  - HDF5Array
  - Harmony
  - LIGER
  - R
  - Seurat
  - cluster
  - clusterSim
  - fdm2id
  - ggplot2
  - rsvd
  - scater
  - scran
  - scuttle
  - stAPAminer
  - uwot
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# umap-embedding-visualization

## Summary

Embeds GEDI-derived latent spaces — either the recovered integrated cell embedding (svd_res$v %*% diag(svd_res$d)) or the top-20 most variable sample-specific manifold parameters (Qi) after PCA rotation — into 2D with uwot::umap (min_dist = 0.01, euclidean metric) and visualizes them with GEDI::plot_embedding colored by donor, technology, sample, or cell type.

## When to use

Use when a fitted GEDI model's integrated cell space or sample-specific manifold parameters need a 2D UMAP visualization annotated by donor, technology, sample, or cell type.

## When NOT to use

- The input is raw counts or a SingleCellExperiment without a fitted GEDI model — GEDI must be fit first and the integrated/manifold space recovered (svd.gedi or Qi) before UMAP is meaningful.
- The goal is producing the integration_benchmark_scores.csv quantitative ranking (Luecken-style batch-removal and cell-type-conservation metrics) — UMAP here is a qualitative visualization step, not the scored output.

## Inputs

- Fitted GEDI model object (with params$Qi and aux$cellIDs populated, obtained after model$setup and optimization)
- Result of svd.gedi on the GEDI model (svd_res) for cell-level integrated space recovery
- Sample-level metadata matrix H mapping samples to donor and technology (sample-manifold variant)

## Outputs

- 2D UMAP coordinate table annotated with donor/technology (sample level) or sample/cell type (cell level), e.g., sample_umap_coordinates.csv
- UMAP embedding plots from GEDI::plot_embedding colored by donor, technology, sample, or cell type (e.g., PAX5 activity on B cells, TCF7 on CD4+ T cells)

## How to apply

For cell-level embeddings, first recover the integrated space from the fitted GEDI model via svd.gedi, taking svd_res$v %*% diag(svd_res$d), then call uwot::umap with min_dist = 0.01 and metric = 'euclidean'. For sample-level manifold visualization (Fig. 2a/2d style), assemble the parameter matrix Q by concatenating model$params$Qi across samples (rows = genes x latent variables, columns = samples), exclude sample-specific translation vectors Δoi, regress out the covariate of interest by centering each row within covariate groups, keep the top 20 most variable parameters (order(apply(Q_subset,1,sd), decreasing=T)[1:20]), rotate with SVD as svd_Q$v %*% diag(svd_Q$d), and embed with uwot::umap(min_dist = 0.01, metric = 'euclidean', n_neighbors = 3). Annotate and visualize the embedding with GEDI::plot_embedding, coloring by donor and technology (sample level) or by sample and cell type (cell level), and write the annotated coordinate table to CSV. The rationale is that UMAP on GEDI's latent or manifold space reproduces the paper's Figure 2 sample-to-sample variability structure without re-running the model.

## Related tools

- **uwot** (Computes the 2D UMAP embedding of the GEDI integrated space or rotated sample-manifold coordinates (min_dist = 0.01, euclidean metric)) — https://cran.r-project.org/web/packages/uwot/index.html
- **GEDI** (Provides the fitted model whose latent/manifold parameters are embedded, and GEDI::plot_embedding for annotated visualization) — https://github.com/csglab/GEDI
- **rsvd** (GEDI dependency used for randomized SVD underlying latent-space recovery and PCA rotation of manifold parameters)

## Examples

```
emb <- uwot::umap(svd_res$v %*% diag(svd_res$d), min_dist = 0.01, metric = "euclidean"); GEDI::plot_embedding(emb, labels = celltypes)
```

## Evaluation signals

- After regressing out technology from the Qi manifold matrix, samples must cluster by donor in the UMAP (and by technology after regressing out donor), matching the paper's reported result.
- Cell-level UMAP colored by cell type reproduces expected cell-type-restricted patterns, e.g., PAX5 activity localized to B cells and TCF7 activity localized to CD4+ T cells.
- The written coordinate table contains one row per embedded sample (or cell) with donor/technology or sample/cell-type annotation columns.
- Hyperparameters match the paper's settings: min_dist = 0.01, metric = 'euclidean', n_neighbors = 3 for the sample-manifold embedding.

## Limitations

- The sample-manifold encoding excludes sample-specific translation vectors Δoi, so embeddings built with them will not match the paper's Figure 2 layout.
- For the sample-level embedding, only technologies with more than one sample (rowSums of the donor-by-technology table > 1) are analyzed.
- GEDI's manifold learning is linear, so the UMAP visualizes a linear (optionally hyperellipsoid-restricted) parameterization rather than a nonlinear embedding.

## Evidence

- [methods] Embed the rotated coordinates with UMAP (min_dist = 0.01, metric = euclidean, n_neighbors = 3).: [evidence span withheld — anchor status: not_found]
- [methods] Visualize the UMAP with GEDI::plot_embedding colored by donor and by technology, and write the UMAP coordinate table with donor/technology annotations to sample_umap_coordinates.csv.: [evidence span withheld — anchor status: not_found]
- [readme] A generative model that unifies integration, cluster-free differential expression, pathway and regulatory network analysis, data normalization and imputation.: [evidence span withheld — anchor status: external_doc]
- [other] embed each result in 2D with uwot::umap(min_dist = 0.01, metric = "euclidean"), visualizing with GEDI::plot_embedding colored by sample and cell type.: [evidence span withheld — anchor status: not_found]
