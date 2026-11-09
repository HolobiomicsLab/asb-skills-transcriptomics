---
name: spectral-clustering-of-2d-embeddings
description: Use when you have 2D t-SNE or UMAP cell embeddings (original and hyperparameter-optimized)
  for scRNA-seq data with annotated cell type labels and need an objective cluster-vs-label
  agreement metric such as ARI.
when_to_use_negative:
- Cells lack annotated cell type labels, so no external reference exists against which
  to compute the adjusted Rand index
- The goal is embedding-quality assessment alone (dubious vs trustworthy detection),
  which scDEED's reliability scores already provide without clustering
- Using a dimensionality reduction method other than t-SNE/UMAP in Seurat without
  adapting scDEED, since the package currently supports only those two
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDEED
  role: Selects the hyperparameters (perplexity, min.dist, n_neighbors) that minimize
    dubious cell embeddings, producing the optimized embeddings whose ARI is compared
    against the default embeddings
  repo: github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Generates the Seurat objects, PCA, and RunTSNE/RunUMAP 2D embeddings that
    spectral clustering is applied to
  repo: https://satijalab.org/seurat/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:5c4420f4ef1a3dcb30e640950951176dd1d692d1333f93ad93058a365d0c7a9b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - devtools
  - scDEED
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spectral-clustering-of-2d-embeddings

## Summary

Quantitatively scores 2D t-SNE/UMAP cell embeddings by applying spectral clustering to the embedding coordinates and computing the adjusted Rand index (ARI) of the resulting clusters against annotated cell types, so that scDEED-optimized embeddings can be compared with default-hyperparameter embeddings.

## When to use

Use when you have 2D t-SNE or UMAP cell embeddings (original and hyperparameter-optimized) for scRNA-seq data with annotated cell type labels and need an objective cluster-vs-label agreement metric such as ARI.

## When NOT to use

- Cells lack annotated cell type labels, so no external reference exists against which to compute the adjusted Rand index
- The goal is embedding-quality assessment alone (dubious vs trustworthy detection), which scDEED's reliability scores already provide without clustering
- Using a dimensionality reduction method other than t-SNE/UMAP in Seurat without adapting scDEED, since the package currently supports only those two

## Inputs

- Seurat object with normalized/scaled counts, PCA (K PCs chosen via ElbowPlot), and 2D t-SNE/UMAP embeddings (default and scDEED-optimized)
- Vector of annotated cell type labels for each cell (e.g. for the Human PBMC dataset GSE132044)
- scDEED output selecting optimal perplexity (t-SNE) or min.dist/n_neighbors (UMAP) per technology

## Outputs

- Spectral cluster assignments for each cell under each embedding (original and optimized)
- ARI table comparing spectral clusters vs annotated cell types across all technology-method pairs (six pairs for three technologies x t-SNE/UMAP)

## How to apply

For each technology-method pair (e.g. Dropseq, inDrops, SeqWell each embedded with t-SNE and UMAP), generate the default embedding (RunTSNE with perplexity 40, RunUMAP defaults) and the scDEED-optimized embedding (RunTSNE with the perplexity minimizing dubious cells, RunUMAP with the selected min.dist and n.neighbors, seed.use = 100). Apply spectral clustering to the 2D coordinates of both embeddings independently, using the same clustering settings across the comparison so the only variable is the embedding. Compute the adjusted Rand index between the spectral clusters and the annotated cell types for all six pairs and assemble an ARI table. Rationale: an embedding that better aligns 2D clusters with known biology should yield a higher ARI; in the PBMC benchmark this held in five of six technology-method cases after scDEED optimization, validating the hyperparameter selection.

## Related tools

- **scDEED** (Selects the hyperparameters (perplexity, min.dist, n_neighbors) that minimize dubious cell embeddings, producing the optimized embeddings whose ARI is compared against the default embeddings) — github.com/JSB-UCLA/scDEED
- **Seurat** (Generates the Seurat objects, PCA, and RunTSNE/RunUMAP 2D embeddings that spectral clustering is applied to) — https://satijalab.org/seurat/

## Examples

```
data_opt = RunUMAP(data_opt, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100)
```

## Evaluation signals

- ARI of spectral clusters vs annotated cell types is higher for the scDEED-optimized embedding than the default embedding in 5 of 6 technology-method pairs (as in the PBMC benchmark)
- The same spectral clustering configuration is applied to both original and optimized embeddings so the ARI difference isolates the effect of hyperparameter optimization
- The embedding used for clustering matches the scDEED-selected settings exactly (e.g. RunTSNE with optimized perplexity, RunUMAP with optimized min.dist/n.neighbors and seed.use = 100)
- Number of spectral clusters is comparable to the number of annotated cell types, avoiding degenerate clusterings that inflate or deflate ARI
- Reproducing with seed.use = 100 yields stable cluster assignments and ARI values

## Limitations

- The ARI improvement held in five of six cases, so one technology-method pair showed no gain, meaning higher ARI is a strong but not universal signal of better embeddings
- If the number of dubious cell embeddings shows a complex trend as hyperparameter value increases, cells may have complex topology and 2D visualizations should be interpreted cautiously
- A Seurat version difference (5.2.1 vs 5.1.0) was reported to affect scDEED installation, which can block the upstream embedding generation step

## Evidence

- [results] five out of six technology-method cases (three scRNA-seq technologies with t-SNE and UMAP embeddings) showed a higher adjusted Rand index (ARI) with the annotated cell types: [evidence span withheld — anchor status: not_found]
- [readme] An intuitive way to select optimal hyperparameters is by minimizing the number of dubious cell embeddings: "An intuitive way to select optimal hyperparameters is by minimizing the number of dubious cell embeddings"
- [methods] data_opt = RunUMAP(data_opt, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100): "data_opt = RunUMAP(data_opt, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100)"
- [abstract] By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors: "By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors"
