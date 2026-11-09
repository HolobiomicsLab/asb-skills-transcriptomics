---
name: tsne-perplexity-grid-optimization
description: Use when a Seurat scRNA-seq object (e.g., Tabula Muris marrow, CAR-T
  GSE125881, Hydra) needs an optimized t-SNE perplexity, or when comparing hyperparameter
  optimization by scDEED vs EMBEDR via dubious-embedding counts and KNN/KNC preservation
  metrics.
when_to_use_negative:
- Input is not a scRNA-seq count matrix in a Seurat object (scDEED requires Seurat
  objects and a PCA pre-embedding space)
- You need to preserve accurate inter-cluster distances in the 2D plot rather than
  a visualization — none of these optimization strategies guarantee distance preservation
- The number of dubious embeddings shows a complex, non-unimodal trend as perplexity
  increases, indicating complex cell topology requiring cautious interpretation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDEED
  role: 'Core statistical method: grid-searches t-SNE perplexity, computes per-cell
    reliability scores from pre-embedding vs 2D-embedding neighbor similarity, and
    minimizes dubious embeddings to select the optimal perplexity'
  repo: https://github.com/JSB-UCLA/scDEED
- name: scDEED R package
  role: R implementation installed via devtools::install_github('JSB-UCLA/scDEED');
    exposes scDEED(), Permuted(), Distances.tSNE(), Cell.Similarity(), Cell.Classify(),
    optimize() functions
  repo: https://github.com/JSB-UCLA/scDEED
- name: EMBEDR
  role: Competing benchmark method that scores embedding reliability via empirical
    marginal resampling p-values over its candidate perplexity grid (max 1000); comparison
    shows preference for high perplexity
  repo: https://github.com/ejohnson643/EMBEDR
- name: Seurat
  role: Preprocessing pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA,
    ElbowPlot) and t-SNE/UMAP execution; scDEED uses Seurat objects
- name: t-SNE
  role: The 2D embedding method whose perplexity hyperparameter is optimized over
    the grid
- name: R
  role: Runtime environment for the scDEED package and downstream t-test / KNN-KNC
    analysis
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:965f0cf062e6731455bbf3d5c98861eb74fbc35bb28a920768ee34190d021298
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scDEED
  tools:
  - EMBEDR
  - R
  - Seurat
  - scDEED
  - scDEED R package
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# tsne-perplexity-grid-optimization

## Summary

Optimize t-SNE perplexity for scRNA-seq 2D embeddings by running scDEED's grid search over candidate perplexity values and selecting the setting that minimizes the number of dubious cell embeddings, as opposed to EMBEDR's reliability-based selection which tends to prefer high perplexities.

## When to use

Use when a Seurat scRNA-seq object (e.g., Tabula Muris marrow, CAR-T GSE125881, Hydra) needs an optimized t-SNE perplexity, or when comparing hyperparameter optimization by scDEED vs EMBEDR via dubious-embedding counts and KNN/KNC preservation metrics.

## When NOT to use

- Input is not a scRNA-seq count matrix in a Seurat object (scDEED requires Seurat objects and a PCA pre-embedding space)
- You need to preserve accurate inter-cluster distances in the 2D plot rather than a visualization — none of these optimization strategies guarantee distance preservation
- The number of dubious embeddings shows a complex, non-unimodal trend as perplexity increases, indicating complex cell topology requiring cautious interpretation

## Inputs

- Seurat object with raw/normalized scRNA-seq counts (e.g., Tabula Muris marrow 4821 cells, CAR-T GSE125881 CD8+ cells, Hydra input_counts with 10000 cells)
- Number of principal components K (chosen via Seurat ElbowPlot)
- Optional: scDesign3-simulated count matrices for benchmarking
- Perplexity candidate grid (defaults 20-410 step 30; 450-800 step 50)

## Outputs

- num_dubious data.frame of perplexity settings and dubious-cell counts
- full_results data.frame listing dubious/trustworthy/intermediate cells (comma-separated) per perplexity
- Optimized perplexity value (e.g., 100 for scDEED on Tabula Muris marrow)
- Optional: KNN/KNC preservation statistics and paired t-test p-values (perplexity_optimization_comparison.csv)

## How to apply

Preprocess the count matrix with Seurat (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA), choosing the input dimension K via ElbowPlot. Run scDEED with reduction.method='tsne' over the default perplexity grid (20 to 410 by 30, then 450 to 800 by 50) and optionally extend to include higher values such as 1000 to allow head-to-head comparison with EMBEDR. For each perplexity, scDEED computes a reliability score per cell by comparing the Pearson correlation of ordered pre-embedding (PCA) distances and 2D-embedding distances, classifies cells as dubious (score below the dubious_cutoff, default 0.05 percentile), trustworthy (above the trustworthy_cutoff, default 0.95 percentile), or intermediate, using similarity_percent=0.5. Select the perplexity minimizing the number of dubious cells in the num_dubious data.frame; on Tabula Muris marrow (4821 cells) this yields perplexity 100, whereas EMBEDR selects the maximal candidate 1000. Optionally validate by computing KNN and KNC preservation between PCA space and the 2D embedding at each method's chosen perplexity and running paired t-tests across 20 scDesign3-simulated datasets (expected KNN p=7.88e-15, KNC p=0.208), writing results to perplexity_optimization_comparison.csv.

## Related tools

- **scDEED** (Core statistical method: grid-searches t-SNE perplexity, computes per-cell reliability scores from pre-embedding vs 2D-embedding neighbor similarity, and minimizes dubious embeddings to select the optimal perplexity) — https://github.com/JSB-UCLA/scDEED
- **scDEED R package** (R implementation installed via devtools::install_github('JSB-UCLA/scDEED'); exposes scDEED(), Permuted(), Distances.tSNE(), Cell.Similarity(), Cell.Classify(), optimize() functions) — https://github.com/JSB-UCLA/scDEED
- **EMBEDR** (Competing benchmark method that scores embedding reliability via empirical marginal resampling p-values over its candidate perplexity grid (max 1000); comparison shows preference for high perplexity) — https://github.com/ejohnson643/EMBEDR
- **Seurat** (Preprocessing pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, ElbowPlot) and t-SNE/UMAP execution; scDEED uses Seurat objects)
- **t-SNE** (The 2D embedding method whose perplexity hyperparameter is optimized over the grid)
- **R** (Runtime environment for the scDEED package and downstream t-test / KNN-KNC analysis)

## Examples

```
result = scDEED(data, K = K, reduction.method = 'tsne', perplexity = 40, rerun = F)
```

## Evaluation signals

- The num_dubious data.frame shows a minimum at the selected perplexity; on Tabula Muris marrow scDEED must select perplexity 100 (non-maximal), confirming EMBEDR's preference for high perplexity (1000) is method-specific
- On the CAR-T dataset, dubious embeddings drop from ~1.81% to ~0.09% and trustworthy embeddings rise from ~83.99% to ~95.87% when moving from perplexity 30 to the optimized 750
- Dubious + trustworthy + intermediate cell counts sum to the total number of cells at every perplexity setting (cutoffs 0.05/0.95 percentiles of the null distribution)
- Paired t-tests across 20 simulated datasets reproduce KNN p=7.88e-15 and KNC p=0.208
- The full_results data.frame contains comma-separated dubious/trustworthy cell lists for each perplexity row

## Limitations

- No guarantee that the optimized perplexity makes the 2D embedding accurately preserve pre-embedding distances between cell clusters
- Under optimized hyperparameters, the difference between dubious and trustworthy embeddings may reflect randomness rather than biological signal
- If the number of dubious embeddings shows a complex trend as perplexity increases, cells may have complex topology and 2D visualizations should be interpreted cautiously
- Seurat version differences (5.2.1 vs 5.1.0) affected installation; do not update all packages when installing scDEED
- EMBEDR tends to prefer the maximal candidate perplexity (e.g., 1000), which should be considered when setting the grid for fair comparison

## Evidence

- [abstract] By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors, scDEED identifies the cell embeddings with: "By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors"
- [abstract] By minimizing the number of dubious cell embeddings provides guidance for optimizing hyperparameters of an embedding method: "by minimizing the number of dubious cell embeddings, scDEED provides intuitive guidance for optimizing the hyperparameters of an embedding method"
- [readme] For t-SNE: reduction.method = 'tsne' perplexity default values: "For t-SNE: reduction.method = 'tsne' - `perplexity`: The default perplexity values are: 20 to 410, increasing by 30 and 450 to 800, increasing by 50"
- [readme] dubious_cutoff default percentile definition: "For an embedding to be considered dubious, the cell's similarity score must be lower than the 5th percentile (or `dubious_cutoff` percentile) of cell embedding scores from the null distribution."
- [discussion] scDEED outperforms EMBEDR in hyperparameter optimization: "Our results indicate that scDEED outperforms a competing method, EMBEDR, in terms of dubious embedding detection, hyperparameter optimization, and computational time."
- [discussion] Caution when dubious counts trend complexly with hyperparameter value: "if the number of dubious cell embeddings exhibits a complex trend as a hyperparameter value increases, then the cells might have a complex topology, and users should be cautious when interpreting the"
- [readme] Seurat preprocessing steps from the scDEED tutorial: "data = CreateSeuratObject(input_counts)
data = NormalizeData(data)
data = ScaleData(data)
data = FindVariableFeatures(data)
data = RunPCA(data)"
- [intro] No guarantee of distance preservation: "none of these optimization strategies or new methods can guarantee to what extent the resultant cell embeddings will accurately preserve the distances between cell clusters"
