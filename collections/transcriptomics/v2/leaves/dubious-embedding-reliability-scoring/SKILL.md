---
name: dubious-embedding-reliability-scoring
description: Use when a Seurat-processed scRNA-seq dataset has 2D t-SNE or UMAP embeddings
  and you need to detect which cell embeddings are unreliable or choose embedding
  hyperparameters (perplexity, min.dist, n_neighbors) that minimize dubious cells.
when_to_use_negative:
- When the embedding method is not t-SNE or UMAP and you do not want to adapt scDEED's
  pipeline to another dimension reduction method
- When you only need a global embedding-quality summary (e.g. EMBEDR-style p-values
  or DynamicViz stability scores) rather than per-cell dubious/trustworthy classification
  and hyperparameter selection
- When the number of dubious embeddings shows a complex, non-monotonic trend as a
  hyperparameter value increases — the article advises caution since cells may have
  complex topology and optimization guidance becomes ambiguous
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: scDEED R package
  role: 'Core method: computes reliability scores, classifies dubious/trustworthy
    embeddings, and sweeps hyperparameters via scDEED(), Permuted(), Distances.pre_embedding(),
    Cell.Similarity(), Cell.Classify(), optimize()'
  repo: github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Preprocessing (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures,
    RunPCA) and running t-SNE/UMAP embeddings on Seurat objects
  repo: satijalab/seurat-wrappers
- name: devtools
  role: Installs scDEED from GitHub via devtools::install_github
  repo: github.com/JSB-UCLA/scDEED
- name: doParallel
  role: Parallel backend to speed up the per-hyperparameter embedding/distance computations
- name: EMBEDR
  role: Competing method for embedding evaluation used as a benchmark; scDEED was
    reported to outperform it in dubious detection, optimization, and runtime
  repo: ejohnson643/EMBEDR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:04e5d1fa79ebbd168b7b8ac72d8975631148c90cba979a938b8508fbaa48b629
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMBEDR
  - R
  - Seurat
  - devtools
  - doParallel
  - scDEED R package
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# dubious-embedding-reliability-scoring

## Summary

Score the reliability of every cell's 2D embedding (t-SNE or UMAP) by comparing post-embedding neighbors to pre-embedding (PCA) neighbors using Pearson correlation, classify embeddings as dubious/trustworthy/intermediate against null-distribution percentile cutoffs, and minimize dubious counts to optimize hyperparameters.

## When to use

Use when a Seurat-processed scRNA-seq dataset has 2D t-SNE or UMAP embeddings and you need to detect which cell embeddings are unreliable or choose embedding hyperparameters (perplexity, min.dist, n_neighbors) that minimize dubious cells.

## When NOT to use

- When the embedding method is not t-SNE or UMAP and you do not want to adapt scDEED's pipeline to another dimension reduction method
- When you only need a global embedding-quality summary (e.g. EMBEDR-style p-values or DynamicViz stability scores) rather than per-cell dubious/trustworthy classification and hyperparameter selection
- When the number of dubious embeddings shows a complex, non-monotonic trend as a hyperparameter value increases — the article advises caution since cells may have complex topology and optimization guidance becomes ambiguous

## Inputs

- Seurat object with normalized, scaled counts, variable features, and PCA (input_counts matrix or .rda)
- Choice of input dimension K (number of PCs, e.g. via ElbowPlot)
- Hyperparameter grids: perplexity values for t-SNE (default 20-410 step 30, 450-800 step 50); min.dist (default 0.1, 0.4) and n_neighbors (default 5, 20, 30, 40, 50) for UMAP

## Outputs

- num_dubious data.frame: hyperparameter settings with counts of dubious cell embeddings per setting
- full_results data.frame: hyperparameter settings plus dubious/trustworthy/intermediate cell classifications (comma-separated in last 3 columns)
- Per-cell similarity scores (rho_original vs rho_permuted) and classified cell labels

## How to apply

Preprocess counts in Seurat (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA) and choose the input dimension K, e.g. via Seurat::ElbowPlot. Compute pre-embedding and post-embedding neighbor distances (Distances.pre_embedding / Distances.tSNE or Distances.UMAP) and build a null distribution by permuting the data with Permuted(). Score each cell's embedding with Cell.Similarity (Pearson rho of ordered neighbor distances, similarity_percent=0.5 default) and classify with Cell.Classify using dubious_cutoff=0.05 and trustworthy_cutoff=0.95 percentiles of the null distribution. Optimize hyperparameters by minimizing the number of dubious embeddings — one-at-a-time (marginal) sweeps followed by a joint grid search over (min.dist, n_neighbors) for UMAP or over perplexity for t-SNE. Outputs are a num_dubious data.frame per hyperparameter setting and a full_results data.frame with dubious/trustworthy/intermediate cell lists per setting.

## Related tools

- **scDEED R package** (Core method: computes reliability scores, classifies dubious/trustworthy embeddings, and sweeps hyperparameters via scDEED(), Permuted(), Distances.pre_embedding(), Cell.Similarity(), Cell.Classify(), optimize()) — github.com/JSB-UCLA/scDEED
- **Seurat** (Preprocessing (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA) and running t-SNE/UMAP embeddings on Seurat objects) — satijalab/seurat-wrappers
- **devtools** (Installs scDEED from GitHub via devtools::install_github) — github.com/JSB-UCLA/scDEED
- **doParallel** (Parallel backend to speed up the per-hyperparameter embedding/distance computations)
- **EMBEDR** (Competing method for embedding evaluation used as a benchmark; scDEED was reported to outperform it in dubious detection, optimization, and runtime) — ejohnson643/EMBEDR

## Examples

```
library(devtools); devtools::install_github("JSB-UCLA/scDEED"); result = scDEED(data, K = 8, reduction.method = 'umap', dubious_cutoff = 0.05, trustworthy_cutoff = 0.95, similarity_percent = 0.5)
```

## Evaluation signals

- Argmin of the dubious-count sweep reproduces reported optima (e.g. on the Alveolar dataset: n.neighbors=5 and min.dist=0.6 marginal; min.dist=0.7, n.neighbors=6 joint)
- Dubious percentages at optima match published values (2.28%, 2.45%, 2.04%) within tolerance; most cells classified trustworthy (>~58% at the joint optimum)
- scDEED() returns both required outputs: a num_dubious data.frame and a full_results data.frame whose last 3 columns hold comma-separated dubious/trustworthy/intermediate cell lists
- Null-distribution cutoffs respected: dubious cells have similarity scores below the 5th percentile and trustworthy cells above the 95th percentile of rho_permuted scores
- Pipeline invariants hold: embeddings run with seed.use=100 for reproducibility and pre-embedding distances computed in the K-dimensional PCA space

## Limitations

- If the number of dubious cell embeddings exhibits a complex trend as a hyperparameter value increases, cells might have complex topology and users should be cautious interpreting the 2D visualization
- Under optimized hyperparameters, differences between dubious and trustworthy embeddings may reflect randomness rather than biological signals
- It remains an open question how the optimized hyperparameter setting relates to the internal algorithms of t-SNE and UMAP, and how dubious vs trustworthy embeddings differ in gene expression profiles
- Installing scDEED may trigger dependency updates (e.g. Seurat 5.2.1 vs 5.1.0 differences); users should avoid blanket package updates
- The joint UMAP grid search over two hyperparameters can be computationally expensive

## Evidence

- [abstract] By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors: "By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors"
- [abstract] scDEED identifies low/high reliability embeddings as dubious/trustworthy: "scDEED identifies the cell embeddings with low reliability scores as dubious and those with high reliability scores as trustworthy"
- [abstract] Minimizing dubious embeddings guides hyperparameter optimization: "by minimizing the number of dubious cell embeddings, scDEED provides intuitive guidance for optimizing the hyperparameters of an embedding method"
- [methods] Cell.Classify with 0.95 cutoffs: "ClassifiedCells<-Cell.Classify(similarity_score$rho_original,similarity_score$rho_permuted, dubious_cutoff = 0.95, trustworthy_cutoff = 0.95)"
- [methods] RunUMAP with seed and hyperparameters: "data_opt = RunUMAP(data_opt, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100)"
- [readme] README: dubious cutoff semantics: "dubious_cutoff`: (default = 0.05) This sets the cutoff for classifying cells. For an embedding to be considered dubious, the cell's similarity score must be lower than the 5th percentile"
- [readme] README: scDEED outputs num_dubious and full_results: "- `num_dubious`: This is a data.frame that shows the hyperparameter settings and the number of dubious cells at those hyperparameter settings"
- [readme] README: comparison of neighbors before and after embedding: "A cell embedding is considered 'dubious' if the cell's neighbors before and after the embedding differ drastically."
