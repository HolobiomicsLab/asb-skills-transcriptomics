---
name: pca-neighborhood-distance-computation
description: Use when running scDEED and you need the pre-embedding neighbor distances
  against which 2D t-SNE/UMAP neighbor distances will be correlated — i.e., after
  RunPCA on a Seurat object and after generating the Permuted null (row-shuffled scale.data)
  PCA space.
when_to_use_negative:
- You need post-embedding distances from t-SNE or UMAP — use Distances.tSNE or Distances.UMAP
  instead; this skill only covers the pre-embedding (PCA) side.
- The input is raw counts without the Seurat NormalizeData/ScaleData/FindVariableFeatures/RunPCA
  chain, or there is no permuted-null object to compare against.
- The 2D embedding method is not t-SNE/UMAP and the pre-embedding space has been changed
  away from PCA without adjusting the pre_embedding argument.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scDEED
  role: Provides the Distances.pre_embedding function that computes ordered neighbor
    distances in the PCA pre-embedding space
  repo: github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Builds the pre-embedding PCA space (CreateSeuratObject, NormalizeData, ScaleData,
    FindVariableFeatures, RunPCA) and hosts the scale.data slot that Permuted shuffles
- name: distances
  role: R package used by scDEED for neighbor-distance computations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:36c87155ee96e67db3d9e992daf147f7df23e7e5535258f4c4b0df4c037690cb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - devtools
  - distances
  - scDEED
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# pca-neighborhood-distance-computation

## Summary

Computes per-cell ordered nearest-neighbor distance vectors in the K-dimensional PCA pre-embedding space (for both original and gene-permuted Seurat data), which serve as the high-dimensional reference for scDEED's Pearson-correlation reliability scores.

## When to use

Use when running scDEED and you need the pre-embedding neighbor distances against which 2D t-SNE/UMAP neighbor distances will be correlated — i.e., after RunPCA on a Seurat object and after generating the Permuted null (row-shuffled scale.data) PCA space.

## When NOT to use

- You need post-embedding distances from t-SNE or UMAP — use Distances.tSNE or Distances.UMAP instead; this skill only covers the pre-embedding (PCA) side.
- The input is raw counts without the Seurat NormalizeData/ScaleData/FindVariableFeatures/RunPCA chain, or there is no permuted-null object to compare against.
- The 2D embedding method is not t-SNE/UMAP and the pre-embedding space has been changed away from PCA without adjusting the pre_embedding argument.

## Inputs

- Seurat object with a PCA reduction (post RunPCA on normalized/scaled Hydra input_counts data)
- Permuted Seurat object (gene-row-shuffled scale.data with fixed seed, PCA recomputed)
- K: number of PCs defining the pre-embedding dimension (user-specified, e.g., via ElbowPlot)

## Outputs

- results.PCA$pre_embedding_distances: per-cell ordered neighbor-distance vectors in the original PCA space
- results.PCA$pre_embedding_distances_permuted: corresponding distances in the permuted-null PCA space
- Distance matrices consumed by Cell.Similarity to produce per-cell Pearson rho scores

## How to apply

First build the PCA space by running the Seurat chain CreateSeuratObject → NormalizeData → ScaleData → FindVariableFeatures → RunPCA on the input_counts Hydra data, choosing K via ElbowPlot (e.g., K=8 in the vignette). Generate the permuted null with Permuted(data, default_assay='RNA', slot='scale.data', K), which shuffles each gene row of the scale.data slot with a fixed seed (e.g., 1000) before recomputing PCA. Then call Distances.pre_embedding(data, data.permuted, K=K, pre_embedding='pca') to obtain cell-by-cell ordered neighbor distances in the PCA space for both original and permuted data. The fraction of neighbors considered is controlled downstream by similarity_percent=0.5 (higher values give a global focus, lower values a local focus). These distance matrices feed Cell.Similarity, which correlates pre-embedding and 2D-embedding neighbor distances to produce rho_original and rho_permuted reliability scores used by Cell.Classify at the 5th/95th percentile cutoffs.

## Related tools

- **scDEED** (Provides the Distances.pre_embedding function that computes ordered neighbor distances in the PCA pre-embedding space) — github.com/JSB-UCLA/scDEED
- **Seurat** (Builds the pre-embedding PCA space (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA) and hosts the scale.data slot that Permuted shuffles)
- **distances** (R package used by scDEED for neighbor-distance computations)

## Examples

```
results.PCA = Distances.pre_embedding(data, data.permuted, K=K, pre_embedding = 'pca')
```

## Evaluation signals

- Distance vectors are cell-by-cell with one ordered neighbor-distance vector per cell in the K-dim PCA space (K=8 in the vignette).
- Permuted distances must derive from a PCA recomputed on a scale.data slot shuffled per gene row with a fixed reproducible seed (e.g., seed 1000).
- Downstream Cell.Similarity must be able to correlate these vectors with 2D-embedding distances at similarity_percent=0.5 to yield rho_original and rho_permuted.
- Verify against the package tsne_table reference: with these pre-embedding distances feeding Cell.Classify at dubious_cutoff=0.05, the minimum of 15 dubious cells should occur at the optimized perplexity of 200.
- results.PCA should contain both pre_embedding_distances and pre_embedding_distances_permuted components before Cell.Similarity is called.

## Limitations

- The pre-embedding space defaults to PCA; changing it requires replacing the scale.data slot or adding a normalized layer and specifying the slot/pre_embedding arguments.
- K must be chosen by the user (e.g., via ElbowPlot); an inappropriate K distorts the pre-embedding neighbor structure that reliability scores depend on.
- Seurat version differences (5.2.1 vs 5.1.0) can affect installation and behavior of the pipeline.
- Distinctions between dubious and trustworthy embeddings may reflect randomness rather than biological signal under optimized hyperparameters.

## Evidence

- [methods] Distances.pre_embedding call: "results.PCA = Distances.pre_embedding(data, data.permuted, K=K, pre_embedding = 'pca')"
- [methods] Permuted null construction: "data.permuted = Permuted(data,  default_assay = 'RNA', slot = 'scale.data', K)"
- [readme] Pearson correlation of ordered pre- vs post-embedding distances: "comparing the pearson correlation of ordered distances between a cell and its nearest neighbors in the pre-embedding space and its nearest neighbors in the post-embedding space"
- [readme] similarity_percent semantics: "`similarity_percent`: (default = 0.5) the percentage of neighboring cells to consider in the calculations"
- [readme] Pre-embedding space is PCA by default: "By default, the pre-embedding space is the PCA space, although this can also be changed if the user wishes"
