---
name: single-cell-embedding-reliability-scoring
description: Use when you have a scRNA-seq count matrix (Seurat object) and want to
  detect dubious 2D cell embeddings from t-SNE/UMAP, or to grid-search perplexity
  / min.dist / n_neighbors by minimizing the number of dubious cell embeddings.
when_to_use_negative:
- You only have a finished 2D visualization without the underlying count matrix or
  the pre-embedding PCA distances — scDEED's reliability score requires comparing
  2D-embedding neighbors with pre-embedding neighbors
- Your goal is purely embedding-stability assessment without reference to pre-embedding
  cell distances — the article notes DynamicViz covers the stability-only perspective
  and cannot decide dubiousness from pre-embedding distances
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scDEED
  role: Computes the reliability score per cell embedding, classifies cells as dubious/trustworthy/intermediate
    against the Permuted null distribution, and grid-searches t-SNE perplexity or
    UMAP min.dist/n_neighbors
  repo: github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Provides the Seurat object workflow (CreateSeuratObject, NormalizeData, ScaleData,
    FindVariableFeatures, RunPCA, RunTSNE, RunUMAP) whose PCA space is the pre-embedding
    input to scDEED
  repo: github.com/satijalab/seurat-wrappers
- name: EMBEDR
  role: Competing embedding-reliability method used for head-to-head benchmarking;
    unlike scDEED it tends to prefer high (maximal) perplexity values such as 1000
  repo: github.com/ejohnson643/EMBEDR
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:e1b09b84456917111a9a1579e4498bce95e55f818be64f0162eb71081bf91dc4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMBEDR
  - R
  - Seurat
  - scDEED
  - scDEED R package
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-embedding-reliability-scoring

## Summary

Apply scDEED (single-cell dubious embedding detector) to score the reliability of every cell's 2D t-SNE/UMAP embedding by comparing post-embedding neighbors to pre-embedding PCA neighbors, labeling low-scoring embeddings as dubious and high-scoring ones as trustworthy, and use the minimized count of dubious embeddings to optimize t-SNE perplexity or UMAP min.dist/n_neighbors.

## When to use

Use when you have a scRNA-seq count matrix (Seurat object) and want to detect dubious 2D cell embeddings from t-SNE/UMAP, or to grid-search perplexity / min.dist / n_neighbors by minimizing the number of dubious cell embeddings.

## When NOT to use

- You only have a finished 2D visualization without the underlying count matrix or the pre-embedding PCA distances — scDEED's reliability score requires comparing 2D-embedding neighbors with pre-embedding neighbors
- Your goal is purely embedding-stability assessment without reference to pre-embedding cell distances — the article notes DynamicViz covers the stability-only perspective and cannot decide dubiousness from pre-embedding distances

## Inputs

- scRNA-seq count matrix (input_counts.rda; e.g. 10000 randomly selected Hydra cells) loadable via CreateSeuratObject
- Seurat object processed with NormalizeData, ScaleData, FindVariableFeatures, RunPCA
- Chosen pre-embedding dimension K (number of PCs, e.g. K=31 or K=8 from ElbowPlot)
- Candidate t-SNE perplexity grid (20 to 410 by 30; 450 to 800 by 50) or UMAP min.dist/n_neighbors values

## Outputs

- result$num_dubious data.frame of hyperparameter settings and number of dubious cells at each
- result$full_results data.frame with dubious_cells / trustworthy_cells / intermediate cells per hyperparameter setting
- Perplexity or UMAP hyperparameter optimization curve minimizing dubious embeddings
- Percentage of dubious vs trustworthy cell embeddings per setting (e.g. Hydra: 4.77%/92.84% at perplexity 40, 0.51%/99.1% at 230, out of 24985 cells)

## How to apply

Build a Seurat object with CreateSeuratObject, then run NormalizeData, ScaleData, FindVariableFeatures, and RunPCA; choose the pre-embedding dimension K from an ElbowPlot (e.g. K=31 PCs for the Hydra dataset, K=8 in the tutorial). Construct the null distribution with Permuted(data, default_assay='RNA', slot='scale.data', K) and call scDEED(data, K=K, reduction.method='tsne', perplexity=40, rerun=F), keeping the defaults similarity_percent=0.5 (percentile of neighbors considered, local vs global focus), dubious_cutoff=0.05, and trustworthy_cutoff=0.95; run Cell.Similarity on pre-embedding distances (similarity_percent=0.5) and Cell.Classify to assign dubious/trustworthy/intermediate cells. Iterate scDEED across the default t-SNE perplexity grid (20 to 410 by 30, then 450 to 800 by 50) or UMAP min.dist (0.1, 0.4) and n_neighbors (5, 20, 30, 40, 50), reading num_dubious and full_results, and select the hyperparameter value minimizing the number of dubious cell embeddings (e.g. perplexity 230 for Hydra t-SNE; perplexity 100 for Tabula Muris marrow). Extract dubious_cells and trustworthy_cells columns from result$full_results to compute per-class percentages out of the total cell count.

## Related tools

- **scDEED** (Computes the reliability score per cell embedding, classifies cells as dubious/trustworthy/intermediate against the Permuted null distribution, and grid-searches t-SNE perplexity or UMAP min.dist/n_neighbors) — github.com/JSB-UCLA/scDEED
- **Seurat** (Provides the Seurat object workflow (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunTSNE, RunUMAP) whose PCA space is the pre-embedding input to scDEED) — github.com/satijalab/seurat-wrappers
- **EMBEDR** (Competing embedding-reliability method used for head-to-head benchmarking; unlike scDEED it tends to prefer high (maximal) perplexity values such as 1000) — github.com/ejohnson643/EMBEDR

## Examples

```
result = scDEED(data, K = K, reduction.method = 'tsne', perplexity = 40, rerun = F)
```

## Evaluation signals

- On the Hydra dataset (24985 cells), scDEED at perplexity 40 should classify ~4.77% dubious and ~92.84% trustworthy embeddings, shifting to ~0.51% dubious and ~99.1% trustworthy at the optimized perplexity 230
- On the Tabula Muris marrow dataset scDEED should select a non-maximal optimal t-SNE perplexity of 100, whereas EMBEDR selects the maximal candidate 1000
- The classification uses defaults similarity_percent=0.5, dubious_cutoff=0.05 (5th percentile of the null), and trustworthy_cutoff=0.95 (95th percentile of the null) unless explicitly changed
- result$full_results must contain dubious/trustworthy/intermediate cell columns at every hyperparameter setting, and dubious embeddings should be minimized at the optimized perplexity
- The null distribution must be built from Permuted(data, default_assay='RNA', slot='scale.data', K), not from the original data

## Limitations

- Seurat version differences (5.2.1 vs 5.1.0) can affect installation; do not update all packages when installing scDEED via devtools
- If the number of dubious cell embeddings shows a complex trend as a hyperparameter value increases, cells may have complex topology and users should be cautious interpreting the 2D visualization
- Differences between dubious and trustworthy embeddings under optimized hyperparameters may be due more to randomness than biological signal, and whether the optimized perplexity aligns neighbor-picking probabilities with cell topology remains an open question
- The default UMAP grid (min.dist 0.1/0.4; n_neighbors 5–50) makes the two-hyperparameter grid search computationally expensive

## Evidence

- [abstract] scDEED reliability scoring rationale: "calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors"
- [methods] Core scDEED call at Seurat-default t-SNE perplexity: "result = scDEED(data, K = K, reduction.method = 'tsne', perplexity = 40, rerun = F)"
- [readme] Definition of a dubious embedding: "A cell embedding is considered 'dubious' if the cell's neighbors before and after the embedding differ drastically."
- [readme] Cutoff semantics and intermediate cells: "Intermediate cells have similarity scores within the 5th and 95th percentile of the null distribution."
