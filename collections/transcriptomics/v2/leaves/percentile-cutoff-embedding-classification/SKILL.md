---
name: percentile-cutoff-embedding-classification
description: Use when per-cell 2D-embedding neighbor distances (original and gene-permuted
  data) and pre-embedding (PCA) distances are available and each cell must be labeled
  dubious/trustworthy/intermediate, e.g. for t-SNE perplexity or UMAP min.dist/n_neighbors
  optimization.
when_to_use_negative:
- The input is raw counts or a feature table without a computed 2D embedding and pre-embedding
  (PCA) neighbor distances — classification requires both reduced-dimension and pre-embedding
  distance vectors
- No gene-permuted null data can be generated, since the cutoffs are defined as percentiles
  of the permuted-cell null score distribution
- The goal is embedding stability assessment only (e.g. DynamicViz), not deciding
  whether embeddings preserve pre-embedding cell distances
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_4085
- http://edamontology.org/topic_3308
tools:
- name: scDEED
  role: Provides the Cell.Classify step that applies the percentile cutoffs to original
    and permuted similarity scores, producing dubious/trustworthy/intermediate labels
  repo: https://github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Builds the Seurat object (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures,
    RunPCA, RunTSNE/RunUMAP) whose pre-embedding and 2D distances feed the classification
  repo: https://github.com/satijalab/seurat-wrappers
- name: devtools
  role: Installs the scDEED R package from GitHub
  repo: https://github.com/JSB-UCLA/scDEED
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:e2d49939708c8848d4723b47ed7ee0fb4a1c852e280e075b23cbfd2d02483790
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scDEED
  tools:
  - R
  - Seurat
  - devtools
  - distances
  - scDEED
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_4085
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# percentile-cutoff-embedding-classification

## Summary

Classifies each cell's 2D embedding (t-SNE/UMAP) as dubious, trustworthy, or intermediate by comparing its reliability score (Pearson correlation of ordered pre-embedding vs 2D-embedding neighbor distances) against a gene-permuted null score distribution using 5th/95th-percentile cutoffs. This is scDEED's core per-cell decision rule and the basis for hyperparameter optimization by minimizing dubious embeddings.

## When to use

Use when per-cell 2D-embedding neighbor distances (original and gene-permuted data) and pre-embedding (PCA) distances are available and each cell must be labeled dubious/trustworthy/intermediate, e.g. for t-SNE perplexity or UMAP min.dist/n_neighbors optimization.

## When NOT to use

- The input is raw counts or a feature table without a computed 2D embedding and pre-embedding (PCA) neighbor distances — classification requires both reduced-dimension and pre-embedding distance vectors
- No gene-permuted null data can be generated, since the cutoffs are defined as percentiles of the permuted-cell null score distribution
- The goal is embedding stability assessment only (e.g. DynamicViz), not deciding whether embeddings preserve pre-embedding cell distances

## Inputs

- Per-cell reliability scores (Pearson correlation of ordered neighbor distances) for original data, e.g. similarity_score$rho_original
- Per-cell reliability scores for gene-permuted null data, e.g. similarity_score$rho_permuted
- Seurat object with NormalizeData/ScaleData/FindVariableFeatures/RunPCA preprocessing and a 2D t-SNE or UMAP reduction
- Cutoff percentiles: dubious_cutoff (default 0.05) and trustworthy_cutoff (default 0.95)

## Outputs

- Per-cell classification as dubious, trustworthy, or intermediate
- Per-cell similarity (reliability) scores
- num_dubious data.frame of hyperparameter settings and dubious-cell counts
- full_results data.frame of hyperparameter settings with dubious/trustworthy/intermediate cell lists

## How to apply

First compute per-cell reliability scores as Pearson correlations between each cell's ordered nearest-neighbor distance vector in the K-dimensional PCA pre-embedding space and in the 2D embedding, for both the original data and the gene-permuted null data (Permuted on the 'scale.data' slot, e.g. seed 1000; similarity_percent = 0.5). Build the null distribution from the permuted cells' scores, then apply the classification rule: a cell embedding is dubious when its similarity score falls below the dubious_cutoff percentile (default 0.05) of the null distribution and trustworthy when it exceeds the trustworthy_cutoff percentile (default 0.95), leaving intermediate cells in between. The cutoffs are user-adjustable percentiles; raising dubious_cutoff or lowering trustworthy_cutoff classifies more cells into those categories. Perplexity/hyperparameter settings are then ranked by the number of dubious cells they produce, with the minimum indicating the optimized setting (e.g. minimum of 15 dubious cells at perplexity 200 in the Hydra reconstruction).

## Related tools

- **scDEED** (Provides the Cell.Classify step that applies the percentile cutoffs to original and permuted similarity scores, producing dubious/trustworthy/intermediate labels) — https://github.com/JSB-UCLA/scDEED
- **Seurat** (Builds the Seurat object (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunTSNE/RunUMAP) whose pre-embedding and 2D distances feed the classification) — https://github.com/satijalab/seurat-wrappers
- **devtools** (Installs the scDEED R package from GitHub) — https://github.com/JSB-UCLA/scDEED

## Examples

```
ClassifiedCells <- Cell.Classify(similarity_score$rho_original, similarity_score$rho_permuted, dubious_cutoff = 0.05, trustworthy_cutoff = 0.95)
```

## Evaluation signals

- Percentile semantics match: dubious cells have scores below the 5th-percentile (dubious_cutoff = 0.05) of the permuted-null distribution; trustworthy cells exceed the 95th percentile (trustworthy_cutoff = 0.95)
- Dubious + trustworthy + intermediate cell counts sum to the total number of cells at every hyperparameter setting
- Per-perplexity dubious counts reproduce the package's tsne_table reference, including a minimum of 15 dubious cells at the optimized perplexity of 200 on the Hydra input_counts dataset
- Varying the cutoffs in the documented direction changes label counts accordingly (higher dubious_cutoff → more dubious cells; higher trustworthy_cutoff → fewer trustworthy cells)
- Saved report (scdeed_reconstruction_report.csv) contains per-cell similarity scores, dubious/trustworthy labels, and per-perplexity dubious counts

## Limitations

- Users of the scDEED R package can increase or decrease the percentile threshold based on their knowledge and preference — cutoffs are not universal constants
- If the number of dubious cell embeddings exhibits a complex trend as a hyperparameter value increases, cells may have complex topology and the 2D visualization should be interpreted cautiously
- Under optimized hyperparameters, differences between dubious and trustworthy embeddings may be more due to randomness than biological signals; gene-expression differences between the classes remain an open question
- Seurat version differences (5.2.1 vs 5.1.0) can affect installation and thus reproducibility

## Evidence

- [abstract] reliability-score cutoff semantics: "scDEED identifies the cell embeddings with low reliability scores as dubious and those with high reliability scores as trustworthy"
- [readme] dubious percentile rule: "For an embedding to be considered dubious, the cell's similarity score must be lower than the 5th percentile (or `dubious_cutoff` percentile) of cell embedding scores from the null distribution."
- [readme] trustworthy percentile rule: "For an embedding to be considered trustworthy, the cell's similarity score must be higher than the 95th percentile (or `trustworthy_cutoff` percentile)"
- [methods] Cell.Classify call signature: "ClassifiedCells<-Cell.Classify(similarity_score$rho_original,similarity_score$rho_permuted, dubious_cutoff = 0.95, trustworthy_cutoff = 0.95)"
- [readme] intermediate cells definition: "Intermediate cells have similarity scores within the 5th and 95th percentile of the null distribution."
