---
name: wald-test-gene-cluster-differentiation
description: Use when you have PILOT pseudotime plus robust-regression fits of gene
  expression per cell cluster (from pl.tl.genes_importance) and need to rank genes
  with cluster-specific trajectories, e.g. genes dropping only in healthy cardiomyocytes.
when_to_use_negative:
- You need static cluster-vs-cluster marker analysis without a pseudotime trajectory
  — this test compares fitted curves along PILOT's disease progression, not two fixed
  cell states.
- The gene expression fits along pseudotime (pl.tl.genes_importance) have not been
  computed yet; gene_cluster_differentiation depends on those robust regression models.
- Sample-level distances/trajectory have not been established (no Wasserstein distance
  matrix, diffusion-map embedding, or EIPLGraph principal graph with a root sample).
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PILOT
  role: provides pl.tl.genes_importance (robust gene fits along pseudotime) and pl.tl.gene_cluster_differentiation
    (Wald test of cluster-specific gene curves)
  repo: https://github.com/CostaLab/PILOT
- name: limma
  role: empirical Bayes approach used by PILOT to contrast gene expression of cells
- name: scanpy
  role: loads and prepares the myocardial infarction .h5ad AnnData before testing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:d95eb15f6b53e3f7d54dfcd233124eb2d9a98c2b4d22e244af9ccdd0769a9378
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - limma
  - scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# wald-test-gene-cluster-differentiation

## Summary

PILOT's statistical test for genes whose fitted expression curve along a disease-progression pseudotime differs significantly in one cell cluster versus all other clusters, using Wald statistics on Huber regression coefficients.

## When to use

Use when you have PILOT pseudotime plus robust-regression fits of gene expression per cell cluster (from pl.tl.genes_importance) and need to rank genes with cluster-specific trajectories, e.g. genes dropping only in healthy cardiomyocytes.

## When NOT to use

- You need static cluster-vs-cluster marker analysis without a pseudotime trajectory — this test compares fitted curves along PILOT's disease progression, not two fixed cell states.
- The gene expression fits along pseudotime (pl.tl.genes_importance) have not been computed yet; gene_cluster_differentiation depends on those robust regression models.
- Sample-level distances/trajectory have not been established (no Wasserstein distance matrix, diffusion-map embedding, or EIPLGraph principal graph with a root sample).

## Inputs

- AnnData object (.h5ad, e.g. myocardial_infarction.h5ad) with PILOT pseudotime, sample_col='sampleID', and clusters_col='cell_subtype'
- Per-gene robust regression fits along pseudotime produced by pl.tl.genes_importance for each cell type
- List of target cluster names (cellnames) and number of top genes to test (number_genes, e.g. 70)

## Outputs

- gene_clusters_stats_extend.csv containing Wald statistic, p-value, fold change (FC), and expression-pattern columns per gene
- Ranked gene list filtered to FC > 0.5 and sorted by Wald test p-value per target cluster

## How to apply

After running pl.tl.genes_importance for each cell type (plot_genes=False, sample_col='sampleID', col_cell='cell_subtype') to fit robust regression models with linear, quadratic, and linear-quadratic terms along pseudotime, call pl.tl.gene_cluster_differentiation with cellnames set to the target clusters (e.g. ['healthy_CM','Myofib']) and number_genes=70. The test applies Wald statistics — using the linear combination of coefficients — to compare each gene's fitted curve in the target cluster against all other clusters. Results are exported to 'gene_clusters_stats_extend.csv' with columns for Wald statistic, p-value, fold change, and expression pattern; filter to fold change > 0.5 and sort by Wald p-value. Underlying expression contrast is supported by limma's empirical Bayes approach. The myocardial infarction application should reproduce known hallmarks such as MYBPC3/MYOM1/FHOD3 in healthy_CM and increasing fibroblast/myeloid markers.

## Related tools

- **PILOT** (provides pl.tl.genes_importance (robust gene fits along pseudotime) and pl.tl.gene_cluster_differentiation (Wald test of cluster-specific gene curves)) — https://github.com/CostaLab/PILOT
- **limma** (empirical Bayes approach used by PILOT to contrast gene expression of cells)
- **scanpy** (loads and prepares the myocardial infarction .h5ad AnnData before testing)

## Examples

```
pl.tl.gene_cluster_differentiation(cellnames=['healthy_CM','Myofib'], number_genes=70)
```

## Evaluation signals

- 'gene_clusters_stats_extend.csv' exists and contains Wald statistic, p-value, FC, and expression-pattern columns.
- Filtered results (FC > 0.5, sorted by Wald p-value) recover reported MI hallmarks: MYBPC3/MYOM1/FHOD3 in healthy_CM, decrease of cardiomyocytes, increase of fibroblasts and myeloid cells.
- number_genes=70 and cellnames=['healthy_CM','Myofib'] match the reference workflow parameters.
- Significant genes show cluster-specific (not global) changes along the trajectory, e.g. quadratic decrease in healthy cardiomyocytes vs other clusters.

## Limitations

- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence.
- EIPLGraph requires a root sample, which was visually selected to reflect control samples — pseudotime (and hence the tested curves) depends on this subjective choice.
- Analysis of data from distinct technologies (single nucleus vs. single cell) or isolation methods is an open challenge, affecting comparability of tested clusters.

## Evidence

- [discussion] We utilized Wald statistics, using the linear combination of coefficients, to test whether there are differences between pair points of fitted curves: "We utilized Wald statistics, using the linear combination of coefficients, to test whether there are differences between pair points of fitted curves"
- [discussion] it fits regression models with linear, quadratic, and linear-quadratic terms for each feature and uses statistical tests to determine the goodness of fit.: "it fits regression models with linear, quadratic, and linear-quadratic terms for each feature and uses statistical tests to determine the goodness of fit."
- [discussion] Regarding gene expression markers, PILOT uses limma's empirical Bayes approach (Ritchie et al, 2015) to contrast the expression of cells: [evidence span withheld — anchor status: not_found]
- [readme] PILOT provides a statistical approach to find changes in cell populations, gene expression, and tissue structures related to the trajectories: "PILOT provides a statistical approach to find changes in cell populations, gene expression, and tissue structures related to the trajectories"
