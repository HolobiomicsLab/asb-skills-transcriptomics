---
name: pearson-neighborhood-similarity-scoring
description: Use when you need to decide, cell by cell, whether a 2D embedding (t-SNE
  or UMAP) faithfully preserves pre-embedding neighborhood structure, or when optimizing
  hyperparameters like t-SNE perplexity by minimizing dubious embeddings.
when_to_use_negative:
- You only need a global embedding-quality metric or cluster-level assessment — scDEED's
  score is strictly per-cell and neighborhood-based, and methods like EMBEDR or DynamicViz
  serve different purposes (EMBEDR tests against chance, DynamicViz only measures
  stability)
- The embedding hyperparameter already appears optimal or the number of dubious cells
  shows a complex trend with increasing hyperparameter value, suggesting complex cell
  topology where interpretation of the 2D plot requires caution
- Input is raw counts with no Seurat preprocessing pipeline possible — the score requires
  a PCA pre-embedding space built from scale.data
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scDEED
  role: R package providing Cell.Similarity, Cell.Classify, Permuted, and Distances.pre_embedding
    functions that implement the Pearson neighborhood-similarity scoring and null-based
    classification
  repo: github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Builds the Seurat object and PCA pre-embedding space (NormalizeData, ScaleData,
    FindVariableFeatures, RunPCA) and runs t-SNE/UMAP embeddings that are scored
  repo: satijalab/seurat-wrappers
- name: distances
  role: R package used to compute nearest-neighbor distance vectors in pre-embedding
    and 2D-embedding spaces
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:b88ba81c4ef435e57d22b20e74979d794d6f3c7e70a685ffff6c4dd5b113862e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - devtools
  - distances
  - scDEED
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# pearson-neighborhood-similarity-scoring

## Summary

Computes a per-cell reliability score as the Pearson correlation between a cell's ordered pre-embedding neighbor distances (e.g., K-dimensional PCA space) and its 2D t-SNE/UMAP neighbor distances, as implemented in scDEED's Cell.Similarity step. Comparing these scores against a gene-permutation null distribution separates dubious from trustworthy 2D embeddings.

## When to use

Use when you need to decide, cell by cell, whether a 2D embedding (t-SNE or UMAP) faithfully preserves pre-embedding neighborhood structure, or when optimizing hyperparameters like t-SNE perplexity by minimizing dubious embeddings.

## When NOT to use

- You only need a global embedding-quality metric or cluster-level assessment — scDEED's score is strictly per-cell and neighborhood-based, and methods like EMBEDR or DynamicViz serve different purposes (EMBEDR tests against chance, DynamicViz only measures stability)
- The embedding hyperparameter already appears optimal or the number of dubious cells shows a complex trend with increasing hyperparameter value, suggesting complex cell topology where interpretation of the 2D plot requires caution
- Input is raw counts with no Seurat preprocessing pipeline possible — the score requires a PCA pre-embedding space built from scale.data

## Inputs

- Seurat object with normalized, scaled data (scale.data slot) and PCA reduction computed on the Hydra dataset input_counts
- K-dimensional PCA pre-embedding cell distance matrix (Distances.pre_embedding output)
- 2D t-SNE or UMAP reduced-dimension cell distance matrix per hyperparameter setting
- Permuted-data counterparts of both distance matrices (gene-row shuffling of scale.data, fixed seed)

## Outputs

- Per-cell Pearson correlation reliability scores (rho_original) for each 2D embedding setting
- Per-cell null (rho_permuted) score distribution from gene-permuted data
- Dubious/trustworthy/intermediate cell labels at each hyperparameter setting
- Per-hyperparameter counts of dubious embeddings for optimization

## How to apply

First build the pre-embedding space by running the Seurat chain CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA on the input counts (e.g., the Hydra dataset input_counts with 10000 cells), then select K principal components. For each cell, compute ordered distances to its nearest neighbors in the K-dimensional PCA space and in the 2D embedding (similarity_percent=0.5 controls how many neighbors are compared; higher values give a more global focus), and take the Pearson correlation (rho) between the two ordered distance vectors as the reliability score. Repeat the entire computation on permuted data generated by shuffling each gene row of the scale.data slot (e.g., seed 1000) to obtain a null score distribution. Classify a cell as dubious when its original rho falls below the 5th percentile of the null distribution (dubious_cutoff=0.05) and trustworthy when it exceeds the 95th percentile (trustworthy_cutoff=0.95); cells between the cutoffs are intermediate. The number of dubious embeddings at each hyperparameter setting (e.g., perplexity 20–200 grid) guides hyperparameter optimization by minimizing dubious counts.

## Related tools

- **scDEED** (R package providing Cell.Similarity, Cell.Classify, Permuted, and Distances.pre_embedding functions that implement the Pearson neighborhood-similarity scoring and null-based classification) — github.com/JSB-UCLA/scDEED
- **Seurat** (Builds the Seurat object and PCA pre-embedding space (NormalizeData, ScaleData, FindVariableFeatures, RunPCA) and runs t-SNE/UMAP embeddings that are scored) — satijalab/seurat-wrappers
- **distances** (R package used to compute nearest-neighbor distance vectors in pre-embedding and 2D-embedding spaces)

## Examples

```
similarity_score <- Cell.Similarity(results.PCA$pre_embedding_distances, results.PCA$pre_embedding_distances_permuted, reduced_dim_distances, reduced_dim_distances_permuted, similarity_percent=0.5)
```

## Evaluation signals

- Percentile semantics match: dubious classification uses the 5th percentile (dubious_cutoff=0.05) and trustworthy the 95th percentile (trustworthy_cutoff=0.95) of the permuted null score distribution
- Reconstructed per-perplexity dubious counts reproduce the package tsne_table reference, with the minimum of 15 dubious cells occurring at the optimized perplexity of 200
- similarity_percent is set to 0.5 in the Cell.Similarity call, matching the package default
- Per-cell rho_original scores are valid Pearson correlations in [-1, 1] computed between equally ordered pre-embedding and 2D-embedding distance vectors
- Permuted scores are computed from PCA of a scale.data matrix with each gene row shuffled under a fixed seed (e.g., 1000), yielding a null distribution comparable across cells

## Limitations

- The difference between dubious and trustworthy embeddings may be more due to randomness than biological signals under optimized hyperparameters
- How the optimized hyperparameter setting relates to the t-SNE/UMAP algorithms remains an open question, as does whether the optimized perplexity yields more reasonable neighbor-picking probabilities
- If the number of dubious embeddings shows a complex trend as the hyperparameter increases, cells may have complex topology and 2D visualizations should be interpreted cautiously
- Seurat version differences (5.2.1 vs 5.1.0) affected installation of the scDEED package

## Evidence

- [abstract] reliability score from neighbor correlation: "By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors, scDEED identifies the cell embeddings with"
- [methods] Cell.Similarity call: "similarity_score <-Cell.Similarity(results.PCA$pre_embedding_distances,results.PCA$pre_embedding_distances_permuted,reduced_dim_distances,reduced_dim_distances_permuted, similarity_percent=0.5)"
- [methods] Cell.Classify call: "ClassifiedCells<-Cell.Classify(similarity_score$rho_original,similarity_score$rho_permuted, dubious_cutoff = 0.95, trustworthy_cutoff = 0.95)"
- [readme] README on similarity_percent and scoring focus: "similarity_percent`: (default = 0.5) the percentage of neighboring cells to consider in the calculations. A higher similarity percent will consider more cells (global focus)"
- [readme] README on dubious cutoff semantics: "For an embedding to be considered dubious, the cell's similarity score must be lower than the 5th percentile (or `dubious_cutoff` percentile) of cell embedding scores from the null distribution."
