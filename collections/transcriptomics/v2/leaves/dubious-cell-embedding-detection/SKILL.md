---
name: dubious-cell-embedding-detection
description: Use when visualizing single-cell RNA-seq data with t-SNE or UMAP in Seurat
  and you need to quantify which cell embeddings are dubious vs trustworthy, or to
  optimize perplexity/min.dist/n_neighbors.
when_to_use_negative:
- The 2D embedding method is neither t-SNE nor UMAP and cannot be adapted through
  the package's 'Changing Input Space' workflow (package supports tSNE and UMAP via
  Seurat)
- The input is not a Seurat object or count matrix that can be processed through the
  NormalizeData/ScaleData/FindVariableFeatures/RunPCA pipeline
- You need to decide whether a visual cluster should be split into multiple clusters
  or whether distant clusters are truly distinct — scDEED classifies individual cell
  embeddings, not cluster relationships
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: scDEED
  role: 'Core statistical method: permutes data to build a null distribution, computes
    per-cell reliability scores comparing pre-embedding and 2D-embedding neighbor
    distances, classifies cells as dubious/trustworthy/intermediate, and optimizes
    hyperparameters by minimizing dubious embeddings'
  repo: https://github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Provides the Seurat object, preprocessing (NormalizeData, ScaleData, FindVariableFeatures,
    RunPCA), embedding generation (RunTSNE, RunUMAP), ElbowPlot for choosing K, and
    DimPlot visualization of dubious vs trustworthy cells
- name: devtools
  role: Used to install the scDEED R package from GitHub
  repo: https://github.com/JSB-UCLA/scDEED
- name: t-SNE
  role: 2D embedding method whose perplexity is swept and optimized; scDEED supports
    reduction.method = 'tsne' through Seurat RunTSNE
- name: UMAP
  role: 2D embedding method whose min.dist and n_neighbors are swept and optimized;
    scDEED supports reduction.method = 'umap' through Seurat RunUMAP
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:a8001a4da85ad8ec9c154570ca04944a64e1cee2d2b0081f0c288de3c015b165
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scDEED
  tools:
  - R
  - Seurat
  - UMAP
  - devtools
  - scDEED
  - scDEED R package
  - t-SNE
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# dubious-cell-embedding-detection

## Summary

Apply scDEED to detect dubious cell embeddings in 2D t-SNE/UMAP visualizations of scRNA-seq data by comparing each cell's 2D-embedding neighbors against its pre-embedding (PCA) neighbors, and use the resulting classifications to select hyperparameters that minimize dubious embeddings.

## When to use

Use when visualizing single-cell RNA-seq data with t-SNE or UMAP in Seurat and you need to quantify which cell embeddings are dubious vs trustworthy, or to optimize perplexity/min.dist/n_neighbors.

## When NOT to use

- The 2D embedding method is neither t-SNE nor UMAP and cannot be adapted through the package's 'Changing Input Space' workflow (package supports tSNE and UMAP via Seurat)
- The input is not a Seurat object or count matrix that can be processed through the NormalizeData/ScaleData/FindVariableFeatures/RunPCA pipeline
- You need to decide whether a visual cluster should be split into multiple clusters or whether distant clusters are truly distinct — scDEED classifies individual cell embeddings, not cluster relationships

## Inputs

- Raw scRNA-seq count matrix loadable as a Seurat object (e.g., input_counts.rda with 10000 Hydra dataset cells; GSE125881 CAR-T CD8+ cells; GSE132044 PBMC Dropseq/inDrops/SeqWell matrices)
- Number of principal components K chosen from Seurat::ElbowPlot
- Hyperparameter grid: t-SNE perplexity values or UMAP min.dist and n_neighbors values

## Outputs

- num_dubious: data.frame of hyperparameter settings and the number of dubious cells at each setting
- full_results: data.frame of hyperparameter settings with comma-separated dubious_cells, trustworthy_cells, and intermediate_cells columns
- Optimized t-SNE/UMAP embeddings regenerated at the hyperparameters minimizing dubious embeddings
- Percentage breakdown of dubious/trustworthy/intermediate cells per hyperparameter setting

## How to apply

Build a Seurat object with NormalizeData, ScaleData, FindVariableFeatures, and RunPCA, choosing the number of PCs (K) via Seurat::ElbowPlot. Call scDEED on the Seurat object with K, reduction.method = 'tsne' (perplexity grid 20–410 by 30 and 450–800 by 50) or 'umap' (min.dist 0.1/0.4, n_neighbors 5, 20, 30, 40, 50), using defaults similarity_percent = 0.5, dubious_cutoff = 0.05, and trustworthy_cutoff = 0.95. scDEED permutes the data (Permuted) to build a null distribution, computes a reliability score per cell as the Pearson correlation of ordered pre-embedding vs post-embedding neighbor distances, and classifies embeddings as dubious (below the 5th null percentile), trustworthy (above the 95th), or intermediate. From full_results, parse the comma-separated dubious_cells/trustworthy_cells columns per hyperparameter row, and select the hyperparameter setting minimizing the number of dubious cell embeddings. Regenerate the optimized embedding (e.g., RunTSNE(data, perplexity = optimized, seed.use = 100)) and optionally highlight dubious vs trustworthy cells with Seurat DimPlot.

## Related tools

- **scDEED** (Core statistical method: permutes data to build a null distribution, computes per-cell reliability scores comparing pre-embedding and 2D-embedding neighbor distances, classifies cells as dubious/trustworthy/intermediate, and optimizes hyperparameters by minimizing dubious embeddings) — https://github.com/JSB-UCLA/scDEED
- **Seurat** (Provides the Seurat object, preprocessing (NormalizeData, ScaleData, FindVariableFeatures, RunPCA), embedding generation (RunTSNE, RunUMAP), ElbowPlot for choosing K, and DimPlot visualization of dubious vs trustworthy cells)
- **devtools** (Used to install the scDEED R package from GitHub) — https://github.com/JSB-UCLA/scDEED
- **t-SNE** (2D embedding method whose perplexity is swept and optimized; scDEED supports reduction.method = 'tsne' through Seurat RunTSNE)
- **UMAP** (2D embedding method whose min.dist and n_neighbors are swept and optimized; scDEED supports reduction.method = 'umap' through Seurat RunUMAP)

## Examples

```
result = scDEED(data, K = 8, reduction.method = 'tsne', perplexity = 40, rerun = F)
```

## Evaluation signals

- Reproduction check: on the CAR-T dataset (GSE125881 CD8+), dubious embeddings drop from ~1.81% to ~0.09% and trustworthy embeddings rise from ~83.99% to ~95.87% when moving from perplexity 30 to the optimized perplexity 750 (within small tolerance)
- Consistency check: on Human PBMC (GSE132044), spectral clusters from optimized embeddings should yield a higher adjusted Rand index (ARI) with annotated cell types in 5 of 6 technology-method pairs versus original hyperparameters
- The returned num_dubious data.frame must show a minimized dubious count at the selected hyperparameter over the full perplexity or (min.dist, n_neighbors) grid
- Sanity invariant: with defaults, dubious cells have similarity scores below the 5th percentile and trustworthy cells above the 95th percentile of the permuted-data null distribution, so intermediate cells fill the 5th–95th percentile band
- Classifications in full_results must be comma-separated cell lists whose counts sum to the total number of cells at each hyperparameter setting

## Limitations

- Users lack an understanding of the difference between dubious and trustworthy embeddings in terms of gene expression profiles
- It remains an open question how the optimized hyperparameter setting relates to the algorithms of t-SNE and UMAP
- Under optimized hyperparameters, differences between dubious and trustworthy embeddings may be more due to randomness than biological signals
- If the number of dubious embeddings shows a complex trend as hyperparameter value increases, cells may have complex topology and interpretation of the 2D visualization warrants caution
- Installing the package may attempt to update Seurat (version 5.2.1 vs 5.1.0 differences noted); users should avoid updating all packages if they want to keep their Seurat version
- The UMAP grid search over two hyperparameters can be computationally expensive

## Evidence

- [abstract] reliability score per cell: "By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors"
- [abstract] optimize by minimizing dubious: "by minimizing the number of dubious cell embeddings, scDEED provides intuitive guidance for optimizing the hyperparameters of an embedding method"
- [methods] install and call scDEED: "devtools::install_github("JSB-UCLA/scDEED")"
- [methods] classify cells: "ClassifiedCells<-Cell.Classify(similarity_score$rho_original,similarity_score$rho_permuted, dubious_cutoff = 0.95, trustworthy_cutoff = 0.95)"
- [readme] cutoff defaults: "`dubious_cutoff`: (default = 0.05) This sets the cutoff for classifying cells. For an embedding to be considered dubious, the cell's similarity score must be lower"
- [readme] outputs of scDEED function: "- `num_dubious`: This is a data.frame that shows the hyperparameter settings and the number of dubious cells at those hyperparameter settings"
- [readme] default perplexity grid: "- `perplexity`: The default perplexity values are: 20 to 410, increasing by 30 and 450 to 800, increasing by 50"
- [readme] optimization rationale: "An intuitive way to select optimal hyperparameters is by minimizing the number of dubious cell embeddings, thus providing a visualization that most faithfully represents"
