---
name: pca-dimension-reduction-of-scrna-counts
description: Use when you have a genes-by-cells scRNA-seq count matrix (e.g. GSE45719,
  268 mouse embryo cells) and need low-dimensional coordinates for downstream spectral
  clustering or comparison of imputation methods.
when_to_use_negative:
- Data are bulk RNA-seq profiles, where the goal is cell-to-cell transcriptomic variation
  rather than clustering of individual cells
- The downstream method requires more than two latent dimensions and the two-PC projection
  loses subpopulation structure needed by the analysis
- The count matrix has already been embedded (e.g. by ZIFA or other dimension-reduction
  model), making a second PCA step redundant
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scImpute
  role: Imputes dropout values in the raw count matrix (drop_thre = 0.5, Kcluster
    set to known number of embryonic stages, parallel ncores) before PCA; produced
    the best clustering on the first two PCs
  repo: https://github.com/Vivianstats/scImpute
- name: MAGIC
  role: Comparison imputation method whose output matrix is also projected onto the
    first two PCs; two-cell Pearson correlation after MAGIC reached 0.95 versus 0.82
    for scImpute
- name: SAVER
  role: Bayesian comparison imputation method whose output is likewise clustered spectrally
    on the first two PCs for benchmarking
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:297c9c717e9a7a8e238c1648153e4048b427dc0e588ed5b211411672b5c03730
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - MAGIC
  - SAVER
  - scImpute
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pca-dimension-reduction-of-scrna-counts

## Summary

Project single-cell RNA-seq count matrices (raw or imputed by scImpute/MAGIC/SAVER) onto the first two principal components so that cell subpopulations can be clustered and benchmarked. This is the standard post-imputation dimension-reduction step in the scImpute mouse embryo evaluation workflow.

## When to use

Use when you have a genes-by-cells scRNA-seq count matrix (e.g. GSE45719, 268 mouse embryo cells) and need low-dimensional coordinates for downstream spectral clustering or comparison of imputation methods.

## When NOT to use

- Data are bulk RNA-seq profiles, where the goal is cell-to-cell transcriptomic variation rather than clustering of individual cells
- The downstream method requires more than two latent dimensions and the two-PC projection loses subpopulation structure needed by the analysis
- The count matrix has already been embedded (e.g. by ZIFA or other dimension-reduction model), making a second PCA step redundant

## Inputs

- genes-as-rows, cells-as-columns raw count matrix (e.g. GSE45719 mouse preimplantation embryo data, 268 cells)
- imputed count matrix from scImpute (scimpute_count.csv), MAGIC, or SAVER

## Outputs

- cell coordinates on the first two principal components
- spectral clustering assignments at k = 6, 8, 10, 12, 14
- clustering accuracy table (ARI, Jaccard index, NMI, purity) per imputation method and k, e.g. imputation_clustering_benchmark.csv

## How to apply

Start from a raw or imputed count matrix with genes as rows and cells as columns (the scImpute output scimpute_count.csv has the same dimensions as the input). Compute principal components on the cell expression profiles and retain the first two PCs, which the scImpute paper uses as the coordinate space for spectral clustering of the 268 mouse preimplantation embryo cells. In the benchmarking setting, run spectral clustering on these two PCs at k = 6, 8, 10, 12, and 14, and score each solution with adjusted Rand index, Jaccard index, NMI, and purity against known embryonic stage labels. scImpute-imputed data attained the best clustering scores under all four measures versus no imputation, MAGIC, or SAVER, so identical PCA+spectral-clustering settings across methods are essential for a fair comparison.

## Related tools

- **scImpute** (Imputes dropout values in the raw count matrix (drop_thre = 0.5, Kcluster set to known number of embryonic stages, parallel ncores) before PCA; produced the best clustering on the first two PCs) — https://github.com/Vivianstats/scImpute
- **MAGIC** (Comparison imputation method whose output matrix is also projected onto the first two PCs; two-cell Pearson correlation after MAGIC reached 0.95 versus 0.82 for scImpute)
- **SAVER** (Bayesian comparison imputation method whose output is likewise clustered spectrally on the first two PCs for benchmarking)

## Examples

```
scimpute(count_path = "raw_count.csv", infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 6, ncores = 10)
```

## Evaluation signals

- All four measures (ARI, Jaccard, NMI, purity) computed at each k should favor scImpute-imputed data over no imputation, MAGIC, and SAVER on the mouse embryo dataset
- PCA output matrix must have 268 cell rows (one per cell) and exactly 2 retained components for the benchmarked clustering
- Spectral clustering must be run at the same k values (6, 8, 10, 12, 14) for every imputation method to keep the comparison fair
- Two-cell Pearson correlation should rise from 0.72 to 0.82 for scImpute (vs 0.95 for MAGIC), confirming imputation was applied before dimension reduction
- Outlier cell indices returned by scimpute should be excluded before projecting cells onto PCs

## Limitations

- scImpute should be applied on the whole-genome count matrix; a gene filtering step is acceptable but most genes must be present for robust dropout identification
- Imputation efficiency degrades when dropout rates in the raw data are severely high, as in droplet-based technologies
- Choice of Kcluster is left to the user without a definitive selection procedure, which affects the imputation feeding into PCA
- Computational efficiency depends on a cell filtering step based on biological knowledge, which is not specified

## Evidence

- [methods] PCA on first two PCs precedes spectral clustering benchmark: "`scImpute` is developed to accurately and efficiently impute the dropout values in scRNA-seq data."
- [readme] Raw count input format and imputation parameters: "Its only input is the raw count matrix with rows representing genes and columns representing cells."
- [methods] drop_thre controls which values are imputed: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [abstract] scImpute alters only likely dropouts, unlike MAGIC and SAVER: "scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data."
- [intro] MAGIC and SAVER modify all values, changing the PCA input: "Both MAGIC and SAVER would alter all gene expression levels including those unaffected by dropouts"
- [readme] Downstream analyses after imputation: "dimension reduction of scRNA-seq data"
