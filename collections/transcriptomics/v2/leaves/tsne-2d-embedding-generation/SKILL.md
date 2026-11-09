---
name: tsne-2d-embedding-generation
description: Use when a Seurat object has been preprocessed (NormalizeData, ScaleData,
  FindVariableFeatures, RunPCA with K PCs chosen via ElbowPlot) and a 2D t-SNE visualization
  is needed, e.g., as input to scDEED dubious-embedding detection or perplexity optimization.
when_to_use_negative:
- When the goal is a UMAP embedding — use the analogous RunUMAP path with min.dist
  (0.1/0.4) and n_neighbors (5, 20, 30, 40, 50) grids instead
- When the object lacks a PCA pre-embedding space (no RunPCA or K not yet chosen)
  — build the PCA space first via ElbowPlot before embedding
- When 2D visualization is not needed and the goal is clustering on high-dimensional
  PCs directly
edam_operation: http://edamontology.org/operation_3893
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: Seurat
  role: Provides the preprocessing chain (NormalizeData, ScaleData, FindVariableFeatures,
    RunPCA) and RunTSNE for generating the 2D t-SNE embedding; scDEED operates on
    Seurat objects
- name: scDEED
  role: Evaluates the generated t-SNE embeddings per cell for dubiousness and sweeps
    perplexity to select the optimal setting
  repo: github.com/JSB-UCLA/scDEED
- name: scDEED R package
  role: Installed via devtools from GitHub; wraps Seurat-based t-SNE generation and
    per-cell classification
  repo: github.com/JSB-UCLA/scDEED
- name: devtools
  role: Used to install the scDEED R package from GitHub before running the t-SNE/scDEED
    workflow
  repo: github.com/JSB-UCLA/scDEED
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:5205701a79a120b0eaeb6ecdc99cee648c03a4c4e8a7a75c1d6842dab7b573db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - devtools
  - distances
  - scDEED
  - scDEED R package
  edam_operation: http://edamontology.org/operation_3893
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# tsne-2d-embedding-generation

## Summary

Generate 2D t-SNE embeddings of scRNA-seq cells from a Seurat PCA pre-embedding space using Seurat's RunTSNE, with explicit control of perplexity and seed; this is the embedding step that scDEED then evaluates for dubious cell embeddings and uses to optimize the perplexity hyperparameter.

## When to use

Use when a Seurat object has been preprocessed (NormalizeData, ScaleData, FindVariableFeatures, RunPCA with K PCs chosen via ElbowPlot) and a 2D t-SNE visualization is needed, e.g., as input to scDEED dubious-embedding detection or perplexity optimization.

## When NOT to use

- When the goal is a UMAP embedding — use the analogous RunUMAP path with min.dist (0.1/0.4) and n_neighbors (5, 20, 30, 40, 50) grids instead
- When the object lacks a PCA pre-embedding space (no RunPCA or K not yet chosen) — build the PCA space first via ElbowPlot before embedding
- When 2D visualization is not needed and the goal is clustering on high-dimensional PCs directly

## Inputs

- Seurat object with count matrix (e.g., Hydra input_counts.rda, 10000 randomly selected cells)
- Normalized, scaled data with variable features identified (scale.data slot)
- PCA reduction on the first K principal components (K chosen via Seurat::ElbowPlot)
- Perplexity value(s) to use (e.g., default 40, or values on the grid 20–410 by 30 and 450–800 by 50)

## Outputs

- 2D t-SNE reduction stored in the Seurat object
- Per-cell 2D-embedding neighbor-distance vectors for scDEED scoring
- Optimized t-SNE embedding at the selected perplexity with seed.use = 100
- Per-perplexity dubious-cell counts table (e.g., tsne_table reference)

## How to apply

First build the pre-embedding space: create the Seurat object from the count matrix (e.g., input_counts with 10000 Hydra-dataset cells), then run NormalizeData, ScaleData, FindVariableFeatures, and RunPCA, choosing K (e.g., K = 8 in the tutorial) with Seurat::ElbowPlot. Run t-SNE on the PCs with RunTSNE, setting perplexity explicitly — the scDEED workflow sweeps perplexity from 20 to 410 by 30 and 450 to 800 by 50 to find the setting that minimizes dubious cells (e.g., perplexity 200 minimized dubious count in the Hydra reconstruction), and regenerates the optimized embedding with RunTSNE(data, perplexity = optimized, seed.use = 100) for reproducibility. Because t-SNE 2D embeddings may not reliably preserve similarities among cell clusters, pair the embedding with a fixed seed and record the perplexity so downstream per-cell reliability scoring (Pearson correlation of pre-embedding vs. 2D neighbor distances) and dubious/trustworthy classification against the permuted null are well defined. Validate by checking that the embedding is stored as a 2D reduction in the Seurat object and that the chosen perplexity reproduces the expected dubious-cell counts against a reference such as tsne_table.

## Related tools

- **Seurat** (Provides the preprocessing chain (NormalizeData, ScaleData, FindVariableFeatures, RunPCA) and RunTSNE for generating the 2D t-SNE embedding; scDEED operates on Seurat objects)
- **scDEED** (Evaluates the generated t-SNE embeddings per cell for dubiousness and sweeps perplexity to select the optimal setting) — github.com/JSB-UCLA/scDEED
- **scDEED R package** (Installed via devtools from GitHub; wraps Seurat-based t-SNE generation and per-cell classification) — github.com/JSB-UCLA/scDEED
- **devtools** (Used to install the scDEED R package from GitHub before running the t-SNE/scDEED workflow) — github.com/JSB-UCLA/scDEED

## Examples

```
data = RunTSNE(data, perplexity = 200, seed.use = 100); result = scDEED(data, K = K, reduction.method = 'tsne', perplexity = 40, rerun = F)
```

## Evaluation signals

- The Seurat object contains a 2D t-SNE reduction computed on the first K PCs with an explicit, recorded perplexity
- Embedding generation uses a fixed seed (e.g., seed.use = 100) so reruns reproduce identical 2D coordinates
- Dubious-cell counts at the chosen perplexity match a reference (e.g., tsne_table; minimum of 15 dubious cells at optimized perplexity 200 in the Hydra reconstruction)
- Perplexity falls on the documented grid (20 to 410 by 30, and 450 to 800 by 50) when used for scDEED optimization
- Downstream scDEED reliability scores (similarity_percent = 0.5, dubious_cutoff 0.05, trustworthy_cutoff 0.95) can be computed from the embedding's neighbor distances

## Limitations

- t-SNE and UMAP 2D embeddings might not reliably inform the similarities among cell clusters; distance preservation is not guaranteed at any hyperparameter setting
- Seurat version differences (5.2.1 vs 5.1.0) affected installation of the scDEED workflow — avoid blanket package updates during install
- If the number of dubious cell embeddings shows a complex trend as perplexity increases, cells may have complex topology and the 2D visualization should be interpreted cautiously
- The 2D t-SNE grid search across perplexity values can be computationally expensive, particularly for two-hyperparameter methods like UMAP

## Evidence

- [methods] RunTSNE with explicit seed and optimized perplexity: [evidence span withheld — anchor status: not_found]
- [abstract] 2D embeddings may not preserve similarities: "it is well known that t-SNE and UMAP's 2D embeddings might not reliably inform the similarities among cell clusters"
- [readme] Default Seurat t-SNE invocation and ElbowPlot for K: "data = RunTSNE(data)"
- [readme] scDEED wraps t-SNE via Seurat with documented perplexity grid: "For t-SNE: reduction.method = 'tsne'
    - `perplexity`: The default perplexity values are: 20 to 410, increasing by 30 and 450 to 800, increasing by 50"
- [discussion] Perplexity choice drives dubious-embedding counts in the Hydra dataset: "For example, in the Hydra dataset, at a low t-SNE perplexity, cells with dubious embeddings tend to exhibit higher sparsity compared to cells with trustworthy embeddings."
