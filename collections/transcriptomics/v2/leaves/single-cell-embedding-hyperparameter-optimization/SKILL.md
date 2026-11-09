---
name: single-cell-embedding-hyperparameter-optimization
description: Use when a Seurat scRNA-seq object needs a t-SNE or UMAP 2D visualization
  and the hyperparameters (perplexity, min.dist, n_neighbors) are unvalidated, or
  when dubious cell embeddings must be detected before interpreting 2D cluster layout.
when_to_use_negative:
- When the embedding method is neither t-SNE nor UMAP and you are not prepared to
  adapt scDEED's Distances/reduction hooks to another dimension reduction method
- When the number of dubious embeddings shows a complex, non-monotonic trend as the
  hyperparameter increases — the article warns cells may have complex topology and
  2D visualization interpretation needs caution
- When the goal is embedding quality assessment only for stability (DynamicViz-style)
  rather than fidelity to pre-embedding cell distances
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: scDEED
  role: 'Core method: computes reliability scores per cell embedding and minimizes
    dubious embeddings to optimize t-SNE/UMAP hyperparameters on Seurat objects'
  repo: https://github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Provides the scRNA-seq pipeline (NormalizeData, ScaleData, FindVariableFeatures,
    RunPCA, ElbowPlot) and the RunTSNE/RunUMAP embeddings that scDEED optimizes
- name: devtools
  role: Used to install scDEED from GitHub (devtools::install_github('JSB-UCLA/scDEED'))
- name: EMBEDR
  role: Competing method for embedding-quality assessment; the article reports scDEED
    outperforms it in dubious embedding detection, hyperparameter optimization, and
    computational time
  repo: https://github.com/ejohnson643/EMBEDR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:dcb609a05cb3bf4f0a1d4b9f4f006998c19325b0ef500bbfc61f932bb03e11c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scDEED
  tools:
  - EMBEDR
  - R
  - Seurat
  - devtools
  - scDEED
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# single-cell-embedding-hyperparameter-optimization

## Summary

Optimize t-SNE perplexity and UMAP min.dist/n_neighbors for single-cell 2D embeddings using scDEED, which classifies each cell embedding as dubious or trustworthy by comparing pre-embedding (PCA) neighbors with 2D-embedding neighbors, and selects hyperparameters minimizing the number of dubious embeddings.

## When to use

Use when a Seurat scRNA-seq object needs a t-SNE or UMAP 2D visualization and the hyperparameters (perplexity, min.dist, n_neighbors) are unvalidated, or when dubious cell embeddings must be detected before interpreting 2D cluster layout.

## When NOT to use

- When the embedding method is neither t-SNE nor UMAP and you are not prepared to adapt scDEED's Distances/reduction hooks to another dimension reduction method
- When the number of dubious embeddings shows a complex, non-monotonic trend as the hyperparameter increases — the article warns cells may have complex topology and 2D visualization interpretation needs caution
- When the goal is embedding quality assessment only for stability (DynamicViz-style) rather than fidelity to pre-embedding cell distances

## Inputs

- Seurat object with raw count matrix (e.g., input_counts from PBMC GSE132044 Dropseq/inDrops/SeqWell or the Hydra dataset)
- Number of PCs K chosen from Seurat::ElbowPlot
- Hyperparameter grids: perplexity (t-SNE) or min.dist/n_neighbors (UMAP)

## Outputs

- num_dubious data.frame of hyperparameter settings and counts of dubious cells
- full_results data.frame with per-cell dubious/trustworthy/intermediate classifications per setting
- Optimized t-SNE or UMAP embedding regenerated at selected hyperparameters with seed.use = 100
- ARI table comparing spectral clusters in original vs. optimized embeddings against annotated cell types

## How to apply

Process count matrices in Seurat with CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, and RunPCA, choosing the number of PCs (K) via Seurat::ElbowPlot. Run scDEED per embedding method: for t-SNE sweep perplexity 20 to 410 by 30 and 450 to 800 by 50; for UMAP sweep min.dist 0.1/0.4 with n_neighbors 5, 20, 30, 40, 50 — using similarity_percent 0.5, dubious_cutoff 0.05, and trustworthy_cutoff 0.95. scDEED permutes the data (Permuted on the scale.data slot) to build a null distribution of similarity scores, and a cell is dubious if its score falls below the 5th percentile (trustworthy if above the 95th). Select the hyperparameter setting minimizing the number of dubious cell embeddings, then regenerate the embedding with seed.use = 100 (e.g., RunTSNE(data, perplexity = optimized, seed.use = 100) or RunUMAP(data, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100)). Correctness can be judged by whether optimized embeddings improve downstream agreement, e.g., higher adjusted Rand index of spectral clusters versus annotated cell types (as in the Human PBMC Dropseq/inDrops/SeqWell evaluation, where five of six technology-method pairs improved ARI).

## Related tools

- **scDEED** (Core method: computes reliability scores per cell embedding and minimizes dubious embeddings to optimize t-SNE/UMAP hyperparameters on Seurat objects) — https://github.com/JSB-UCLA/scDEED
- **Seurat** (Provides the scRNA-seq pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, ElbowPlot) and the RunTSNE/RunUMAP embeddings that scDEED optimizes)
- **devtools** (Used to install scDEED from GitHub (devtools::install_github('JSB-UCLA/scDEED')))
- **EMBEDR** (Competing method for embedding-quality assessment; the article reports scDEED outperforms it in dubious embedding detection, hyperparameter optimization, and computational time) — https://github.com/ejohnson643/EMBEDR

## Examples

```
library(devtools); devtools::install_github("JSB-UCLA/scDEED"); result = scDEED(data, K = K, reduction.method = 'tsne', perplexity = 40, rerun = F)
```

## Evaluation signals

- The selected hyperparameter setting is the grid point minimizing the count in the num_dubious data.frame returned by scDEED
- Per-cell classifications are consistent with the percentile cutoffs: dubious cells below the 5th percentile and trustworthy cells above the 95th percentile of the permuted null similarity-score distribution
- Regenerated optimized embeddings use seed.use = 100 so runs are reproducible
- On the PBMC evaluation, spectral clustering of optimized embeddings should yield higher adjusted Rand index with annotated cell types than default embeddings in 5 of 6 technology-method pairs (3 technologies × t-SNE/UMAP)

## Limitations

- Differences between dubious and trustworthy embeddings in terms of gene expression profiles are not understood; the article notes the difference may be 'more due to randomness instead of biological signals under the optimized hyperparameter(s)'
- How the optimized hyperparameter setting relates to the internal algorithms of t-SNE and UMAP remains an open question
- If the number of dubious cell embeddings shows a complex trend with hyperparameter value, cells may have complex topology and 2D visualization interpretation warrants caution
- The UMAP two-hyperparameter grid search can be computationally expensive; Seurat version differences (5.2.1 vs 5.1.0) affected installation

## Evidence

- [abstract] scDEED identifies the cell embeddings with low reliability scores as dubious and those with high reliability scores as trustworthy: "By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors, scDEED identifies the cell embeddings with"
- [abstract] Minimizing dubious embeddings guides hyperparameter optimization: "by minimizing the number of dubious cell embeddings, scDEED provides intuitive guidance for optimizing the hyperparameters of an embedding method"
- [readme] dubious_cutoff definition: "`dubious_cutoff`: (default = 0.05) This sets the cutoff for classifying cells. For an embedding to be considered dubious, the cell's similarity score must be lower than the 5th percentile"
- [readme] Default hyperparameter grids for t-SNE and UMAP: "The default perplexity values are: 20 to 410, increasing by 30 and 450 to 800, increasing by 50"
- [methods] scDEED uses Seurat objects: "scDEED uses Seurat objects."
- [methods] Optimized UMAP invocation with seed: "data_opt = RunUMAP(data_opt, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100)"
- [discussion] Randomness caveat under optimized hyperparameters: "This result might indicate that the difference between dubious embeddings and trustworthy embeddings is more due to randomness instead of biological signals under the optimized hyperparameter(s)."
- [discussion] scDEED outperforms EMBEDR: "Our results indicate that scDEED outperforms a competing method, EMBEDR, in terms of dubious embedding detection, hyperparameter optimization, and computational time."
