---
name: umap-hyperparameter-grid-search
description: Use when a Seurat scRNA-seq or CyTOF object needs UMAP visualization
  and the analyst must choose min.dist and n.neighbors objectively rather than by
  default, e.g. when 2D cluster layout trustworthiness is in question.
when_to_use_negative:
- When you need a clustering or trajectory result rather than a 2D visualization —
  scDEED optimizes the 2D embedding only and cannot guarantee that embeddings preserve
  inter-cluster distances
- When the number of dubious embeddings shows a complex non-monotonic trend as a hyperparameter
  increases — the authors advise caution, as cells may have complex topology and another
  embedding method may be preferable
- When the input is not a Seurat object or the desired dimension reduction is not
  t-SNE/UMAP without additional adaptation work
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scDEED
  role: Performs the dubious-embedding detection and marginal/joint UMAP hyperparameter
    grid search
  repo: github.com/JSB-UCLA/scDEED
- name: scDEED R package
  role: Installed from GitHub with devtools; provides scDEED(), Permuted(), Distances.pre_embedding(),
    Cell.Similarity(), Cell.Classify(), optimize()
  repo: github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Provides the Seurat object format and preprocessing (NormalizeData, ScaleData,
    FindVariableFeatures, RunPCA, RunUMAP)
- name: UMAP
  role: The 2D embedding method whose min.dist and n.neighbors hyperparameters are
    optimized
- name: R
  role: Implementation environment; devtools used for installation, doParallel for
    parallel grid search
- name: devtools
  role: Installs scDEED from GitHub
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:32c290cbc21613eea4a620045df57f138054ec20f508b138232e9373ff8964fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - UMAP
  - devtools
  - doParallel
  - scDEED
  - scDEED R package
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# umap-hyperparameter-grid-search

## Summary

Grid-search UMAP hyperparameters (min.dist, n.neighbors) using the scDEED R package, which classifies each cell embedding as dubious, trustworthy, or intermediate by comparing 2D-embedding neighbors with pre-embedding (PCA) neighbors, and selects the settings that minimize the number of dubious cell embeddings.

## When to use

Use when a Seurat scRNA-seq or CyTOF object needs UMAP visualization and the analyst must choose min.dist and n.neighbors objectively rather than by default, e.g. when 2D cluster layout trustworthiness is in question.

## When NOT to use

- When you need a clustering or trajectory result rather than a 2D visualization — scDEED optimizes the 2D embedding only and cannot guarantee that embeddings preserve inter-cluster distances
- When the number of dubious embeddings shows a complex non-monotonic trend as a hyperparameter increases — the authors advise caution, as cells may have complex topology and another embedding method may be preferable
- When the input is not a Seurat object or the desired dimension reduction is not t-SNE/UMAP without additional adaptation work

## Inputs

- Seurat object with raw counts (e.g. Alveolar scRNA-seq from GEO GSE141259, 29,297 cells; or Samusik CyTOF with 38 markers)
- Pre-embedding dimension K (number of PCs, chosen via elbow plot)
- Hyperparameter grids for min.dist and n_neighbors
- Cutoffs: dubious_cutoff=0.05, trustworthy_cutoff=0.95, similarity_percent=0.5

## Outputs

- num_dubious data.frame: hyperparameter settings and number of dubious cells at each setting
- full_results data.frame: hyperparameter settings with dubious/trustworthy/intermediate cell lists per setting
- Optimized (min.dist, n.neighbors) settings for marginal and joint searches
- CSV results table (e.g. alveolar_optimization_results.csv, samusik_umap_optimization_results.csv)

## How to apply

Preprocess the data as a Seurat object with CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, and RunPCA, choosing the pre-embedding dimension K (e.g. via ElbowPlot). Build the null distribution by shuffling each row of the scale.data slot with Permuted(), then compute pre-embedding distances with Distances.pre_embedding() on the PCA space. Run scDEED with reduction.method='umap', extending the default grids (min.dist: 0.1, 0.4; n_neighbors: 5, 20, 30, 40, 50) as needed — e.g. the Samusik dataset required n_neighbors up to 160 and min.dist down to 0.05 — using Cell.Similarity with similarity_percent=0.5 and Cell.Classify with dubious_cutoff=0.05 and trustworthy_cutoff=0.95 against the 5th/95th percentiles of the null distribution. Perform both marginal (one-hyperparameter-at-a-time) and joint (min.dist x n_neighbors) searches, selecting the settings that minimize the number of dubious cell embeddings; verified optima include Alveolar joint min.dist=0.7/n.neighbors=6 (2.04% dubious, 58.84% trustworthy) and Samusik joint min.dist=0.7/n.neighbors=160 (0.641% dubious, 99.269% trustworthy).

## Related tools

- **scDEED** (Performs the dubious-embedding detection and marginal/joint UMAP hyperparameter grid search) — github.com/JSB-UCLA/scDEED
- **scDEED R package** (Installed from GitHub with devtools; provides scDEED(), Permuted(), Distances.pre_embedding(), Cell.Similarity(), Cell.Classify(), optimize()) — github.com/JSB-UCLA/scDEED
- **Seurat** (Provides the Seurat object format and preprocessing (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP))
- **UMAP** (The 2D embedding method whose min.dist and n.neighbors hyperparameters are optimized)
- **R** (Implementation environment; devtools used for installation, doParallel for parallel grid search)
- **devtools** (Installs scDEED from GitHub)

## Examples

```
result = scDEED(data, K = K, reduction.method = 'umap', similarity_percent = 0.5, dubious_cutoff = 0.05, trustworthy_cutoff = 0.95)
```

## Evaluation signals

- The argmin settings from the grid search match reported optima: Alveolar marginal n.neighbors=5 and min.dist=0.6, joint min.dist=0.7/n.neighbors=6; Samusik marginal n.neighbors=160 and min.dist=0.05, joint min.dist=0.7/n.neighbors=160
- Dubious percentages at optimized settings reproduce reported values: 2.28% (Alveolar marginal n.neighbors=5), 2.45% (marginal min.dist=0.6), 2.04% (Alveolar joint), and 0.68%/0.837%/0.641% (Samusik marginal/marginal/joint)
- Trustworthy percentages match reports: 58.84% (Alveolar joint), 99.19%/99.028%/99.269% (Samusik)
- num_dubious data.frame covers every hyperparameter setting in the grid with a non-negative dubious count
- Joint optimum yields fewer dubious embeddings than either marginal optimum in both reported datasets

## Limitations

- The joint two-hyperparameter grid search can be computationally expensive; README advises timing one setting first to estimate total runtime and extending grids only within resource constraints
- scDEED'sSeurat dependency is version-sensitive: Seurat 5.2.1 was reportedly different from 5.1.0 and can affect installation (do not update all packages during install)
- Differences between dubious and trustworthy embeddings may be due more to randomness than biological signals under optimized hyperparameters; how optimized settings relate to UMAP's algorithm remains an open question
- Default UMAP grids (min.dist 0.1/0.4; n_neighbors 5, 20, 30, 40, 50) may not contain the true optimum — candidate optima outside the grid (e.g. n_neighbors=160) require extending it

## Evidence

- [abstract] By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors: "By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors, scDEED identifies the cell embeddings with"
- [readme] For UMAP: reduction.method = 'umap' — min.dist: The default min.dist values are: 0.1 and 0.4; n_neighbors: The default n_neighbors values are: 5, 20, 30, 40, 50: "min.dist`: The default min.dist values are: 0.1 and 0.4; `n_neighbors`:The default n_neighbors values are: 5, 20, 30, 40, 50"
- [methods] data_opt = RunUMAP(data_opt, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100): "data_opt = RunUMAP(data_opt, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100)"
- [readme] dubious_cutoff: (default = 0.05) This sets the cutoff for classifying cells.: "`dubious_cutoff`: (default = 0.05) This sets the cutoff for classifying cells. For an embedding to be considered dubious, the cell's similarity score must be lower than the 5th percentile"
- [abstract] By minimizing the number of dubious cell embeddings provides guidance for optimizing hyperparameters: "by minimizing the number of dubious cell embeddings, scDEED provides intuitive guidance for optimizing the hyperparameters of an embedding method"
- [readme] The grid search (due to two hyperparameters, rather than one) can be computationally expensive.: "The grid search (due to two hyperparameters, rather than one) can be computationally expensive. Depending on the user's resources and time constraints, more hyperparameter options can be added"
