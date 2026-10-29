---
name: cell-embedding-reliability-classification
description: Use when a Seurat scRNA-seq/CyTOF object has been embedded with t-SNE
  or UMAP and you need to quantify which cell embeddings are dubious vs trustworthy,
  or to pick hyperparameters (perplexity, min.dist, n.neighbors) by minimizing dubious
  embeddings.
when_to_use_negative:
- You need embeddings that guarantee preservation of distances between distant cell
  clusters — scDEED flags dubious embeddings but cannot guarantee distance preservation
  across clusters
- You only need embedding stability assessment, not fidelity to pre-embedding distances
  — DynamicViz-style stability methods apply, whereas scDEED 'cannot decide if cell
  embeddings are dubious based on pre-embedding cell distances' alternative is the
  wrong frame here
- The dimensionality reduction is neither t-SNE nor UMAP and you are unwilling to
  adapt the package's reduction hooks
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scDEED R package
  role: 'Core method: computes reliability scores, classifies dubious/trustworthy
    embeddings, and sweeps t-SNE/UMAP hyperparameters'
  repo: https://github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Provides the Seurat object, preprocessing pipeline (NormalizeData/ScaleData/FindVariableFeatures/RunPCA),
    RunTSNE/RunUMAP, ElbowPlot, and DimPlot visualization
  repo: https://github.com/satijalab/seurat-wrappers
- name: devtools
  role: Used to install scDEED from GitHub via devtools::install_github
- name: t-SNE
  role: Embedding method whose perplexity is swept and whose embeddings are classified
- name: UMAP
  role: Embedding method whose min.dist × n.neighbors grid is searched and whose embeddings
    are classified
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:273ba17050837fa85a03e18209c2aaa739940d03986a3220ffa32322ca4c2357
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scDEED
  tools:
  - R
  - Seurat
  - UMAP
  - devtools
  - scDEED R package
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cell-embedding-reliability-classification

## Summary

Classify each cell's 2D embedding (t-SNE or UMAP) as dubious, trustworthy, or intermediate using scDEED, which scores reliability by comparing the cell's 2D-embedding neighbors to its pre-embedding (e.g., PCA) neighbors against a permutation null.

## When to use

Use when a Seurat scRNA-seq/CyTOF object has been embedded with t-SNE or UMAP and you need to quantify which cell embeddings are dubious vs trustworthy, or to pick hyperparameters (perplexity, min.dist, n.neighbors) by minimizing dubious embeddings.

## When NOT to use

- You need embeddings that guarantee preservation of distances between distant cell clusters — scDEED flags dubious embeddings but cannot guarantee distance preservation across clusters
- You only need embedding stability assessment, not fidelity to pre-embedding distances — DynamicViz-style stability methods apply, whereas scDEED 'cannot decide if cell embeddings are dubious based on pre-embedding cell distances' alternative is the wrong frame here
- The dimensionality reduction is neither t-SNE nor UMAP and you are unwilling to adapt the package's reduction hooks

## Inputs

- Seurat object with normalized (NormalizeData), scaled (ScaleData), variable features, and PCA reduction computed
- Integer K: number of PCs defining the pre-embedding space
- Hyperparameter grid: perplexity values (t-SNE) or min.dist × n_neighbors pairs (UMAP)
- Cutoffs: dubious_cutoff=0.05, trustworthy_cutoff=0.95, similarity_percent=0.5

## Outputs

- num_dubious data.frame: hyperparameter settings and number of dubious cells per setting
- full_results data.frame: hyperparameter settings plus comma-separated dubious_cells, trustworthy_cells, and intermediate_cells lists
- Determined optimal hyperparameter setting (e.g., perplexity=750 for CAR-T t-SNE; min.dist=0.7, n.neighbors=160 for Samusik UMAP)

## How to apply

Preprocess the Seurat object with NormalizeData, ScaleData, FindVariableFeatures, and RunPCA, choosing the input dimension K (e.g., via ElbowPlot). Build the null distribution with Permuted(), which shuffles each row of the scale.data slot, and compute pre-embedding distances with Distances.pre_embedding() on the PCA space. For each hyperparameter setting, compute cell similarity scores with Cell.Similarity (similarity_percent=0.5) comparing original vs permuted pre-embedding and reduced-dim distances, then classify cells with Cell.Classify using dubious_cutoff=0.05 and trustworthy_cutoff=0.95 against the 5th/95th percentiles of the null distribution. For t-SNE sweep perplexity (defaults 20–410 by 30, 450–800 by 50); for UMAP grid-search min.dist (defaults 0.1, 0.4) × n_neighbors (defaults 5, 20, 30, 40, 50), extending grids to candidate optima when needed. Select the setting minimizing the number of dubious embeddings and extract dubious/trustworthy cell lists from full_results (comma-separated columns).

## Related tools

- **scDEED R package** (Core method: computes reliability scores, classifies dubious/trustworthy embeddings, and sweeps t-SNE/UMAP hyperparameters) — https://github.com/JSB-UCLA/scDEED
- **Seurat** (Provides the Seurat object, preprocessing pipeline (NormalizeData/ScaleData/FindVariableFeatures/RunPCA), RunTSNE/RunUMAP, ElbowPlot, and DimPlot visualization) — https://github.com/satijalab/seurat-wrappers
- **devtools** (Used to install scDEED from GitHub via devtools::install_github)
- **t-SNE** (Embedding method whose perplexity is swept and whose embeddings are classified)
- **UMAP** (Embedding method whose min.dist × n.neighbors grid is searched and whose embeddings are classified)

## Examples

```
result = scDEED(data, K = 15, reduction.method = 'tsne', perplexity = 30, rerun = F)
```

## Evaluation signals

- Computed dubious/trustworthy fractions match reported values within tolerance: CAR-T t-SNE ~1.81%/83.99% at perplexity 30 and ~0.09%/95.87% at perplexity 750
- Samusik UMAP argmin settings equal reported optima (marginal n.neighbors=160, marginal min.dist=0.05; joint min.dist=0.7/n.neighbors=160) with dubious percentages 0.68%, 0.837%, 0.641% and trustworthy 99.19%, 99.028%, 99.269%
- num_dubious data.frame contains one row per hyperparameter setting with a non-negative dubious cell count; argmin row is the selected setting
- full_results last three columns parse into comma-separated cell lists whose sizes are consistent with the num_dubious counts and the total cell number
- Null distribution from Permuted() yields 5th/95th percentiles such that default cutoffs (0.05/0.95) leave most cells classified trustworthy

## Limitations

- The UMAP two-hyperparameter grid search can be computationally expensive; time one setting first to estimate total runtime
- Difference between dubious and trustworthy embeddings may reflect randomness rather than biological signals under optimized hyperparameters
- If dubious counts show a complex (non-monotonic) trend with hyperparameter value, cells may have complex topology and 2D visualizations should be interpreted cautiously
- Seurat version differences (5.2.1 vs 5.1.0) can affect installation; avoid updating Seurat when installing scDEED
- How the optimized hyperparameter relates to t-SNE/UMAP algorithms remains an open question

## Evidence

- [abstract] reliability score per cell based on neighbor similarity: "By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors, scDEED identifies the cell embeddings with"
- [abstract] hyperparameter optimization by minimizing dubious embeddings: "by minimizing the number of dubious cell embeddings, scDEED provides intuitive guidance for optimizing the hyperparameters of an embedding method"
- [discussion] outperforms EMBEDR: "Our results indicate that scDEED outperforms a competing method, EMBEDR, in terms of dubious embedding detection, hyperparameter optimization, and computational time."
- [methods] Cell.Classify call with cutoffs: "ClassifiedCells<-Cell.Classify(similarity_score$rho_original,similarity_score$rho_permuted, dubious_cutoff = 0.95, trustworthy_cutoff = 0.95)"
- [readme] install and dependencies: "You can install scDEED from GitHub using devtools. It should install in 1 minute, but may take longer if you need to update dependencies."
- [readme] cutoff semantics from README: "Together, the dubious and trustworthy cutoffs determine the intermediate cells. Intermediate cells have similarity scores within the 5th and 95th percentile of the null distribution."
