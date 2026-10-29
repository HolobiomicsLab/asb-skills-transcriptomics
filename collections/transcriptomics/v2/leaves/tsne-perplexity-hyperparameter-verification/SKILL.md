---
name: tsne-perplexity-hyperparameter-verification
description: Use when a single-cell count matrix has been embedded with t-SNE and
  you must verify or report how dubious/trustworthy embedding fractions change between
  an original and a scDEED-optimized perplexity setting.
when_to_use_negative:
- The embedding method or hyperparameter under study is not t-SNE perplexity (e.g.,
  UMAP min.dist/n_neighbors grids) without adapting the scDEED call accordingly
- The input is raw imaging or already-embedded 2D coordinates with no underlying count
  matrix to build the Seurat object and PCA pre-embedding space from
- You need a stability-only visualization assessment (DynamicViz-style) rather than
  a pre-embedding-distance-based reliability test
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3366
tools:
- name: scDEED
  role: Classifies each cell's t-SNE embedding as dubious, trustworthy, or intermediate
    by comparing 2D-embedding neighbors with pre-embedding neighbors, and finds the
    perplexity minimizing dubious embeddings
  repo: github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Builds the Seurat object, runs NormalizeData/ScaleData/FindVariableFeatures/RunPCA
    to produce the pre-embedding PCA space that scDEED consumes
- name: t-SNE
  role: The 2D embedding method whose perplexity hyperparameter is being evaluated
    and optimized
- name: R
  role: Language in which the scDEED pipeline and verification script are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:42b4aa00a1f845ad29916545dc028e932e922e77882ed5aeb931cc4b6c8562c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - scDEED
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tsne-perplexity-hyperparameter-verification

## Summary

Reproduce scDEED-based verification of t-SNE perplexity hyperparameter choice by quantifying dubious and trustworthy cell embeddings at original vs. optimized perplexity values on a single-cell dataset (e.g., Hydra: 4.77%→0.51% dubious, 92.84%→99.1% trustworthy when moving from perplexity 40 to 230).

## When to use

Use when a single-cell count matrix has been embedded with t-SNE and you must verify or report how dubious/trustworthy embedding fractions change between an original and a scDEED-optimized perplexity setting.

## When NOT to use

- The embedding method or hyperparameter under study is not t-SNE perplexity (e.g., UMAP min.dist/n_neighbors grids) without adapting the scDEED call accordingly
- The input is raw imaging or already-embedded 2D coordinates with no underlying count matrix to build the Seurat object and PCA pre-embedding space from
- You need a stability-only visualization assessment (DynamicViz-style) rather than a pre-embedding-distance-based reliability test

## Inputs

- Gene-by-cell count matrix (input_counts.rda) loadable into a Seurat object
- Pre-embedding dimensionality K (number of PCs, e.g., K=31 for Hydra)
- Original t-SNE perplexity setting (e.g., 40)
- scDEED-optimized t-SNE perplexity setting (e.g., 230)
- scDEED default thresholds: similarity_percent=0.5, dubious_cutoff=0.05, trustworthy_cutoff=0.95

## Outputs

- Percentage of dubious cell embeddings at each perplexity (out of total cells, e.g., 24,985)
- Percentage of trustworthy cell embeddings at each perplexity
- scdeed_verification_results.csv with percentages for both perplexity settings
- result$full_results data.frame with dubious_cells/trustworthy_cells columns per hyperparameter setting

## How to apply

Build a Seurat object from the count matrix (e.g., input_counts.rda from the Hydra Zenodo deposit) with CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, and RunPCA, choosing K PCs (K=31 for the full Hydra data; the README tutorial uses K=8 on 10,000 cells) as the pre-embedding dimension. Run scDEED with reduction.method='tsne', perplexity=40, rerun=F, keeping defaults similarity_percent=0.5, dubious_cutoff=0.05, trustworthy_cutoff=0.95, which builds the null distribution from Permuted data. Rerun scDEED at the optimized perplexity (230, t-SNE with seed.use=100) on the same PCA space. Extract dubious_cells and trustworthy_cells from result$full_results for both perplexities, compute each class's percentage out of the total cells (24,985 for Hydra), and compare to reported values (4.77%/92.84% at perplexity 40; 0.51%/99.1% at perplexity 230) within numerical reproduction tolerance before writing results to scdeed_verification_results.csv.

## Related tools

- **scDEED** (Classifies each cell's t-SNE embedding as dubious, trustworthy, or intermediate by comparing 2D-embedding neighbors with pre-embedding neighbors, and finds the perplexity minimizing dubious embeddings) — github.com/JSB-UCLA/scDEED
- **Seurat** (Builds the Seurat object, runs NormalizeData/ScaleData/FindVariableFeatures/RunPCA to produce the pre-embedding PCA space that scDEED consumes)
- **t-SNE** (The 2D embedding method whose perplexity hyperparameter is being evaluated and optimized)
- **R** (Language in which the scDEED pipeline and verification script are executed)

## Examples

```
result = scDEED(data, K = 31, reduction.method = 'tsne', perplexity = 40, rerun = F)
```

## Evaluation signals

- Dubious fraction at perplexity 40 reproduces 4.77% (of 24,985 Hydra cells) within numerical tolerance
- Trustworthy fraction at perplexity 40 reproduces 92.84%; at perplexity 230, dubious=0.51% and trustworthy=99.1%
- Optimized perplexity (230) yields a strictly lower dubious count than the original perplexity (40), consistent with scDEED's minimize-dubious criterion
- scdeed_verification_results.csv contains percentage rows for both perplexity settings
- scDEED run used default cutoffs: dubious_cutoff=0.05 and trustworthy_cutoff=0.95 percentiles of the null distribution from Permuted data

## Limitations

- Seurat version differences (5.2.1 vs 5.1.0) reportedly affect installation, so pin the environment when reproducing
- The paper's methods example uses 10,000 randomly selected Hydra cells in input_counts, while the verification target cites 24,985 cells; cell counts and K must match the specific data load
- scDEED is limited to t-SNE and UMAP as implemented through Seurat; other methods require adaptation
- Under optimized hyperparameters, dubious/trustworthy differences may reflect randomness rather than biological signals, so percent changes should be interpreted cautiously

## Evidence

- [results] At the scDEED-optimized t-SNE perplexity of 230 on the Hydra dataset, dubious cell embeddings decreased from 4.77% to 0.51% of all cell embeddings: [evidence span withheld — anchor status: not_found]
- [abstract] scDEED identifies the cell embeddings with low reliability scores as dubious and those with high reliability scores as trustworthy: "scDEED identifies the cell embeddings with low reliability scores as dubious and those with high reliability scores as trustworthy"
- [methods] result = scDEED(data, K = K, reduction.method = 'tsne', perplexity = 40, rerun = F): "result = scDEED(data, K = K, reduction.method = 'tsne', perplexity = 40, rerun = F)"
- [readme] This input_counts contains 10000 randomly selected cells from the Hydra dataset.: "This input_counts contains 10000 randomly selected cells from the Hydra dataset."
- [readme] for an embedding to be considered dubious, the cell's similarity score must be lower than the 5th percentile (or dubious_cutoff percentile) of cell embedding scores from the null distribution: "For an embedding to be considered dubious, the cell's similarity score must be lower than the 5th percentile (or `dubious_cutoff` percentile) of cell embedding scores from the null distribution"
