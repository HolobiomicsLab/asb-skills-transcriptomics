---
name: adjusted-rand-index-clustering-validation
description: Use when 2D cell embeddings (t-SNE or UMAP) have been generated at both
  original and scDEED-optimized hyperparameters and you must quantify whether optimized
  embeddings align 2D clusters with annotated cell types.
when_to_use_negative:
- No annotated cell-type labels exist to serve as the reference clustering, so ARI
  cannot be computed
- The goal is to detect dubious embeddings or optimize hyperparameters themselves
  rather than to validate a chosen setting (use scDEED's dubious-embedding minimization
  instead)
- Embeddings are not regenerated at both original and optimized hyperparameters, leaving
  no baseline for the ARI comparison
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDEED
  role: Supplies the optimized t-SNE/UMAP hyperparameters (minimizing dubious cell
    embeddings) whose embeddings are evaluated against the original ones via ARI
  repo: https://github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Processes scRNA-seq counts (NormalizeData, ScaleData, FindVariableFeatures,
    RunPCA) and generates the t-SNE/UMAP embeddings that are clustered and scored
    by ARI
- name: R
  role: Implementation environment for the Seurat/scDEED pipeline and spectral clustering/ARI
    computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:93d89ba9cf459a844e6082a58981ba60089a9d8078f51cd9c695a115d1c635a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scDEED
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# adjusted-rand-index-clustering-validation

## Summary

Validates that scDEED-optimized t-SNE/UMAP 2D embeddings better represent biology by clustering the 2D coordinates (spectral clustering) and measuring adjusted Rand index (ARI) against annotated cell types, comparing original vs. optimized hyperparameters.

## When to use

Use when 2D cell embeddings (t-SNE or UMAP) have been generated at both original and scDEED-optimized hyperparameters and you must quantify whether optimized embeddings align 2D clusters with annotated cell types.

## When NOT to use

- No annotated cell-type labels exist to serve as the reference clustering, so ARI cannot be computed
- The goal is to detect dubious embeddings or optimize hyperparameters themselves rather than to validate a chosen setting (use scDEED's dubious-embedding minimization instead)
- Embeddings are not regenerated at both original and optimized hyperparameters, leaving no baseline for the ARI comparison

## Inputs

- 2D t-SNE or UMAP embeddings generated in Seurat at original hyperparameters (e.g., RunTSNE perplexity 40, default RunUMAP)
- 2D embeddings regenerated at scDEED-optimized hyperparameters (perplexity grid 20-410 by 30 and 450-800 by 50 for t-SNE; min.dist 0.1/0.4 with n_neighbors 5,20,30,40,50 for UMAP)
- Annotated cell-type labels per cell (Human PBMC dataset, GSE132044: Dropseq, inDrops, SeqWell)
- Number of PCs K selected via Seurat::ElbowPlot

## Outputs

- Spectral cluster assignments on both original and optimized 2D embeddings for each technology-method pair
- ARI table comparing adjusted Rand index of spectral clusters versus annotated cell types across all six technology-method pairs

## How to apply

For each technology-method pair (e.g., Dropseq/inDrops/SeqWell PBMC data with t-SNE and UMAP), regenerate embeddings at original settings (e.g., RunTSNE with perplexity 40) and at scDEED-optimized settings (e.g., RunTSNE(data, perplexity = optimized, seed.use = 100) or RunUMAP(data, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100)), where K is chosen via Seurat::ElbowPlot and optimized hyperparameters minimize the number of dubious cell embeddings. Apply spectral clustering to both the original and the optimized 2D embeddings for every combination. Compute the adjusted Rand index of the spectral clusters versus the annotated cell types for all six technology-method pairs and assemble an ARI table. Judgment of success: the optimized embeddings should yield a higher ARI than the original embeddings in five of the six pairs, as reported for the Human PBMC dataset (GSE132044).

## Related tools

- **scDEED** (Supplies the optimized t-SNE/UMAP hyperparameters (minimizing dubious cell embeddings) whose embeddings are evaluated against the original ones via ARI) — https://github.com/JSB-UCLA/scDEED
- **Seurat** (Processes scRNA-seq counts (NormalizeData, ScaleData, FindVariableFeatures, RunPCA) and generates the t-SNE/UMAP embeddings that are clustered and scored by ARI)
- **R** (Implementation environment for the Seurat/scDEED pipeline and spectral clustering/ARI computation)

## Evaluation signals

- An ARI table is produced covering all six technology-method pairs (3 technologies x t-SNE/UMAP)
- Optimized-embedding ARI exceeds original-embedding ARI in five of the six pairs, matching the reported result on the Human PBMC dataset
- Spectral clustering was applied to both original and optimized 2D embeddings, not only one condition
- Optimized embeddings were regenerated with a fixed seed (seed.use = 100) for reproducibility
- Number of PCs (K) was chosen from ElbowPlot before embedding and is consistent across the comparison

## Limitations

- Validation relies on annotated cell types as ground truth, so ARI improvements may reflect annotation quality rather than embedding fidelity
- Caveat general to the scDEED pipeline: if the number of dubious cell embeddings shows a complex trend as hyperparameter value increases, cells may have complex topology and interpretation of the 2D visualization requires caution
- ARI comparison is pair-specific (five of six pairs improved), so a single pair may not improve after optimization

## Evidence

- [results] five out of six technology-method pairs showed a higher adjusted Rand index (ARI): [evidence span withheld — anchor status: not_found]
- [methods] ARI is computed on spectral clusters versus annotated cell types for all pairs: [evidence span withheld — anchor status: not_found]
- [abstract] scDEED hyperparameter optimization rationale: "by minimizing the number of dubious cell embeddings, scDEED provides intuitive guidance for optimizing the hyperparameters of an embedding method"
- [readme] Definition of dubious embeddings underlying the optimized settings being validated: "A cell embedding is considered 'dubious' if the cell's neighbors before and after the embedding differ drastically."
