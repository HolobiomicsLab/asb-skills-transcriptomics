---
name: correspondence-analysis-svd-dimension-reduction
description: Use when you need dimension reduction of raw scRNA-seq count matrices
  (SingleCellExperiment) and want a chi-squared-residual-based embedding (CA) instead
  of log(x+1)+PCA, e.g. for clustering or biplot visualization.
when_to_use_negative:
- When the benchmark requires methods explicitly designed for overdispersion — the
  paper shows adapted CA variants (especially CA with Freeman-Tukey residuals) outperform
  standard CA, so standard CA may underperform in cluster-recovery tasks
- When cell-type ground truth is from complex tissue with unvalidated annotations
  — ARI evaluation may penalize methods that better distinguish rare sub-populations,
  limiting interpretation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2229
tools:
- name: corral
  role: R/Bioconductor package that performs CA on single-table count data via SVD
    of the Pearson chi-squared residual matrix and interfaces with SingleCellExperiment
  repo: github.com/laurenhsu1/corral
- name: R
  role: Analysis language for the corral pipeline
- name: Bioconductor
  role: Framework providing SingleCellExperiment classes corral interfaces with
- name: made4
  role: Earlier package implementing CA for microarray/bulk RNA-seq; used for biplot
    rendering
- name: gridExtra
  role: Arranges multi-panel biplot figures
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:b557d163a9f6abdbc8aca21a402a36cfd6214fe3cbae6339d2b115949bb46b01
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  - gridExtra
  - made4
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# correspondence-analysis-svd-dimension-reduction

## Summary

Apply correspondence analysis (CA) to single-cell RNA-seq count data by computing the SVD of the Pearson chi-squared residual matrix (no log-transformation), implemented in the corral R/Bioconductor package on SingleCellExperiment objects, as an alternative to PCA on logcounts.

## When to use

Use when you need dimension reduction of raw scRNA-seq count matrices (SingleCellExperiment) and want a chi-squared-residual-based embedding (CA) instead of log(x+1)+PCA, e.g. for clustering or biplot visualization.

## When NOT to use

- When the benchmark requires methods explicitly designed for overdispersion — the paper shows adapted CA variants (especially CA with Freeman-Tukey residuals) outperform standard CA, so standard CA may underperform in cluster-recovery tasks
- When cell-type ground truth is from complex tissue with unvalidated annotations — ARI evaluation may penalize methods that better distinguish rare sub-populations, limiting interpretation

## Inputs

- scRNA-seq raw counts matrix as a SingleCellExperiment object (e.g. Zhengmix or SCMixology datasets from CellBench / DuoClustering2018)
- published ground-truth cell-type labels for clustering evaluation

## Outputs

- Cell embeddings (reduced-dimension coordinates) stored in the SingleCellExperiment reducedDim slot
- Feature (gene) loadings from the CA solution for biplot interpretation
- CA biplot of the first two dimensions showing cells and top-weight labeled genes (e.g. zhengmix8ea_ca_biplot.png)
- ARI comparison table of CA vs PCA clustering performance (e.g. ari_comparison.csv)

## How to apply

Load the scRNA-seq count matrix as a SingleCellExperiment object in R/Bioconductor and run standard CA with the corral package, which computes the embedding by SVD of the Pearson chi-squared residual matrix while avoiding distortive log-transformation of counts. corral provides two main calls: `corral` for dimensionality reduction of a single table and `corralm` for multi-table batch integration. Use the resulting cell embeddings for downstream steps such as NNGraph clustering, and evaluate with Adjusted Rand Index against annotated cell types (the paper benchmarks CA on SCMixology and Zhengmix datasets and finds CA matches or exceeds PCA on logcounts). For feature interpretation, rank genes by the L2 norm of their loadings in the first two CA components and render a CA biplot (e.g. with made4/gridExtra) labeling the top ~20 genes furthest from the origin — on Zhengmix8eq these include GNLY, TYROBP, LYZ, S100A8, CD74, and HLA-DRA.

## Related tools

- **corral** (R/Bioconductor package that performs CA on single-table count data via SVD of the Pearson chi-squared residual matrix and interfaces with SingleCellExperiment) — github.com/laurenhsu1/corral
- **R** (Analysis language for the corral pipeline)
- **Bioconductor** (Framework providing SingleCellExperiment classes corral interfaces with)
- **made4** (Earlier package implementing CA for microarray/bulk RNA-seq; used for biplot rendering)
- **gridExtra** (Arranges multi-panel biplot figures)

## Examples

```
BiocManager::install("laurenhsu1/corral", dependencies = TRUE); sce <- corral(sce, whichmat = "counts")
```

## Evaluation signals

- NNGraph clustering ARI on CA embeddings matches or exceeds PCA-on-logcounts ARI across the six benchmark datasets (three SCMixology, three Zhengmix)
- Labeled high-weight features on the Zhengmix8eq CA biplot include GNLY, TYROBP, LYZ, S100A8, CD74, and HLA-DRA
- Top 20 genes by L2 norm of loadings are furthest from the origin in the first two CA components
- No log-transformation is applied before decomposition — the pipeline operates directly on raw counts
- Output biplot file (e.g. zhengmix8ea_ca_biplot.png) exists and renders correctly

## Limitations

- No tool or package versions reported for corral or dependencies
- Benchmarking ground-truth labels come from original dataset publications and were not independently validated; ARI may penalize methods that better distinguish sub-populations or rare cell types
- High-quality ground-truth benchmarking datasets are limited to simple pseudo-cell mixtures or pools of distinct cell types and do not reflect real-world tissue complexity

## Evidence

- [abstract] CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation: "CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation"
- [readme] corral is a package to perform correspondence analysis on count data: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
- [readme] two main function calls: "The package contains two main function calls:

1. `corral` for dimensionality reduction on a single table 
2. `corralm` for alignment and batch integration of multiple tables"
- [intro] corral interfaces with Bioconductor classes: "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment)"
- [intro] CA computed by SVD of Pearson residuals: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [discussion] CA-FT most performant: "CA-FT was most performant overall in a scRNAseq cluster recovery task."
