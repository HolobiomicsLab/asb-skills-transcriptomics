---
name: single-cell-embedding-quality-assessment
description: Use when a t-SNE or UMAP visualization of single-cell data must be quality-controlled
  or its hyperparameters (perplexity, min.dist, n.neighbors) chosen objectively, e.g.
  when cluster adjacency or distances in the 2D plot need to be judged trustworthy.
when_to_use_negative:
- The goal is stability-only assessment of embeddings without reference to pre-embedding
  distances (DynamicViz handles stability but cannot decide if embeddings are dubious
  based on pre-embedding cell distances)
- Input is not a Seurat-compatible single-cell count matrix with a defined pre-embedding
  space (scDEED uses Seurat objects and defaults to the PCA space from the scale.data
  slot)
- A non-linear embedding method other than t-SNE/UMAP is used without first adapting
  the package (the package is set up for t-SNE and UMAP; other methods need custom
  adaptation)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDEED
  role: 'Core method: computes per-cell reliability scores, classifies embeddings
    as dubious/intermediate/trustworthy, and optimizes t-SNE/UMAP hyperparameters
    by minimizing dubious embeddings'
  repo: https://github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Data container and preprocessing pipeline (CreateSeuratObject, NormalizeData,
    ScaleData, FindVariableFeatures, RunPCA, RunUMAP/RunTSNE); scDEED uses Seurat
    objects
- name: R
  role: Implementation language; scDEED is installed from GitHub with devtools and
    provides Permuted, Distances.pre_embedding, Cell.Similarity, Cell.Classify functions
  repo: https://github.com/JSB-UCLA/scDEED
- name: UMAP
  role: 2D embedding method whose min.dist and n.neighbors hyperparameters are assessed/optimized
- name: t-SNE
  role: 2D embedding method whose perplexity is assessed/optimized (e.g. perplexity=40
    in the tutorial)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:6d24b94046485585bdce86724fd2f6704a40ac4e07c016d69ef91e973d0dd6cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scDEED
  tools:
  - R
  - Seurat
  - UMAP
  - scDEED
  - scDEED R package
  - t-SNE
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-embedding-quality-assessment

## Summary

Assess the reliability of 2D single-cell embeddings (t-SNE/UMAP) using scDEED, which computes a per-cell reliability score from the similarity between 2D-embedding neighbors and pre-embedding (PCA) neighbors, labeling embeddings as dubious, intermediate, or trustworthy, and optimizes hyperparameters by minimizing dubious embeddings.

## When to use

Use when a t-SNE or UMAP visualization of single-cell data must be quality-controlled or its hyperparameters (perplexity, min.dist, n.neighbors) chosen objectively, e.g. when cluster adjacency or distances in the 2D plot need to be judged trustworthy.

## When NOT to use

- The goal is stability-only assessment of embeddings without reference to pre-embedding distances (DynamicViz handles stability but cannot decide if embeddings are dubious based on pre-embedding cell distances)
- Input is not a Seurat-compatible single-cell count matrix with a defined pre-embedding space (scDEED uses Seurat objects and defaults to the PCA space from the scale.data slot)
- A non-linear embedding method other than t-SNE/UMAP is used without first adapting the package (the package is set up for t-SNE and UMAP; other methods need custom adaptation)

## Inputs

- Raw single-cell count matrix (e.g. CyTOF marker counts such as the 38-marker Samusik dataset, or scRNA-seq like the Hydra 10000-cell input_counts.rda) loadable into a Seurat object
- User-specified number of PCs K for the pre-embedding space (chosen e.g. from ElbowPlot)
- Hyperparameter grids: perplexity values for t-SNE (default 20 to 410 by 30, 450 to 800 by 50); min.dist and n_neighbors grids for UMAP

## Outputs

- num_dubious: data.frame of hyperparameter settings and the number of dubious cells per setting (used to select optima)
- full_results: data.frame of hyperparameter settings with dubious/trustworthy/intermediate cell classifications per setting
- Results table (e.g. samusik_umap_optimization_results.csv) of optimal settings and corresponding dubious/trustworthy percentages

## How to apply

Preprocess the count matrix into a Seurat object with NormalizeData, ScaleData, FindVariableFeatures, and RunPCA, then pick the pre-embedding dimension K (e.g. via Seurat::ElbowPlot). Build the null distribution by shuffling each row of the scale.data slot with Permuted() and compute pre-embedding distances with Distances.pre_embedding() on the PCA space. For each hyperparameter setting, run the embedding (e.g. RunUMAP with dims=1:K, seed.use=100), compute cell similarity scores with Cell.Similarity (similarity_percent=0.5, Pearson correlation of ordered distances), and classify cells with Cell.Classify using dubious_cutoff=0.05 and trustworthy_cutoff=0.95 against the null percentiles. Select the setting minimizing the number of dubious embeddings; in practice extend the default UMAP grids (min.dist 0.1/0.4; n_neighbors 5,20,30,40,50) since optima can lie outside (e.g. Samusik joint optimum min.dist=0.7, n.neighbors=160). Extract dubious/trustworthy cell lists from full_results and validate that the argmin settings and dubious/trustworthy percentages (e.g. 0.641% dubious, 99.269% trustworthy for the Samusik joint optimum) match expectations.

## Related tools

- **scDEED** (Core method: computes per-cell reliability scores, classifies embeddings as dubious/intermediate/trustworthy, and optimizes t-SNE/UMAP hyperparameters by minimizing dubious embeddings) — https://github.com/JSB-UCLA/scDEED
- **Seurat** (Data container and preprocessing pipeline (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP/RunTSNE); scDEED uses Seurat objects)
- **R** (Implementation language; scDEED is installed from GitHub with devtools and provides Permuted, Distances.pre_embedding, Cell.Similarity, Cell.Classify functions) — https://github.com/JSB-UCLA/scDEED
- **UMAP** (2D embedding method whose min.dist and n.neighbors hyperparameters are assessed/optimized)
- **t-SNE** (2D embedding method whose perplexity is assessed/optimized (e.g. perplexity=40 in the tutorial))

## Examples

```
result = scDEED(data, K = K, reduction.method = 'umap', min.dist = c(0.05, 0.1, 0.4, 0.7), n.neighbors = c(5, 20, 30, 40, 50, 160))
```

## Evaluation signals

- Argmin hyperparameter settings reproduce reported optima: Samusik marginal n.neighbors=160 and min.dist=0.05; joint min.dist=0.7 with n.neighbors=160
- Resulting dubious percentages match reported values: 0.68% (marginal n.neighbors=160), 0.837% (marginal min.dist=0.05), 0.641% (joint)
- Resulting trustworthy percentages match reported values: 99.19%, 99.028%, 99.269% respectively
- Dubious cells are those with similarity scores below the 5th percentile of the Permuted() null distribution; trustworthy cells above the 95th percentile
- Timing one hyperparameter setting first gives a runtime estimate consistent with completing the full joint min.dist x n_neighbors grid search

## Limitations

- The joint UMAP grid search over min.dist x n_neighbors pairs is computationally expensive; users may need to extend default grids carefully and time one setting first
- Difference between dubious and trustworthy embeddings may be more due to randomness than biological signals under optimized hyperparameters
- If the number of dubious embeddings shows a complex trend as hyperparameter value increases, cells may have complex topology and 2D visualizations should be interpreted cautiously
- How the optimized hyperparameter setting relates to the t-SNE/UMAP algorithms remains an open question; Seurat version differences (5.2.1 vs 5.1.0) can affect installation

## Evidence

- [abstract] By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors, scDEED identifies the cell embeddings with: "By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors, scDEED identifies the cell embeddings with"
- [abstract] by minimizing the number of dubious cell embeddings, scDEED provides intuitive guidance for optimizing the hyperparameters of an embedding method: "by minimizing the number of dubious cell embeddings, scDEED provides intuitive guidance for optimizing the hyperparameters of an embedding method"
- [readme] For an embedding to be considered dubious, the cell's similarity score must be lower than the 5th percentile (or dubious_cutoff percentile) of cell embedding scores from the null distribution: "For an embedding to be considered dubious, the cell's similarity score must be lower than the 5th percentile (or `dubious_cutoff` percentile) of cell embedding scores from the null distribution"
- [methods] data_opt = RunUMAP(data_opt, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100): "data_opt = RunUMAP(data_opt, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100)"
- [readme] For UMAP: reduction.method = 'umap' - min.dist: The default min.dist values are: 0.1 and 0.4; n_neighbors: The default n_neighbors values are: 5, 20, 30, 40, 50: "For UMAP: reduction.method = 'umap'
    - `min.dist`: The default min.dist values are: 0.1 and 0.4
    - `n_neighbors`:The default n_neighbors values are: 5, 20, 30, 40, 50"
