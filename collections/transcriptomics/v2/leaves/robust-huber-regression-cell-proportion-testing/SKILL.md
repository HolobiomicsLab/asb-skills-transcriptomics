---
name: robust-huber-regression-cell-proportion-testing
description: Use when a sample-level trajectory (pseudotime from Wasserstein distances
  + diffusion maps + EIPLGraph) has been estimated on single-cell data (AnnData .h5ad)
  and you must rank which cell populations and genes change significantly along that
  trajectory.
when_to_use_negative:
- Input is already an aggregated feature/marker table — this skill requires raw cell-level
  AnnData with sample and cluster labels to recompute proportions and fits
- The goal is comparing two samples pairwise or differential expression between discrete
  disease groups rather than testing features along an ordered progression trajectory
- Data are pathomics/structure-level without cell-cluster proportion annotations,
  so cell_importance's per-cluster proportion fits are undefined without adaptation
edam_operation: http://edamontology.org/operation_3659
tools:
- name: PILOT
  role: provides cell_importance (Huber regression per cluster), genes_importance,
    and gene_cluster_differentiation (Wald tests)
  repo: github.com/CostaLab/PILOT
- name: POT library
  role: emd2-based Wasserstein distances feed the trajectory whose pseudotime the
    regressions are fit against
  repo: https://github.com/CostaLab/PILOT
- name: scanpy
  role: loads/preprocesses the .h5ad AnnData before PILOT trajectory estimation
  repo: https://github.com/CostaLab/PILOT
- name: limma
  role: PILOT reuses limma's empirical Bayes approach for gene-expression marker contrasts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:ae33a2cacf5e32bc3c4cfa9ab4e80fbc90dbb5ec89ba5183ca2f40d7ce84e9ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PILOT
  - POT library
  - limma
  - scanpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# robust-huber-regression-cell-proportion-testing

## Summary

PILOT's statistical layer for interpreting disease-progression trajectories: step-wise Huber regression (linear, quadratic, linear-quadratic terms) fits each cell cluster's proportion (and each gene's expression) against EIPLGraph pseudotime, and Wald statistics test whether fitted curves differ between a target cluster and all other clusters.

## When to use

Use when a sample-level trajectory (pseudotime from Wasserstein distances + diffusion maps + EIPLGraph) has been estimated on single-cell data (AnnData .h5ad) and you must rank which cell populations and genes change significantly along that trajectory.

## When NOT to use

- Input is already an aggregated feature/marker table — this skill requires raw cell-level AnnData with sample and cluster labels to recompute proportions and fits
- The goal is comparing two samples pairwise or differential expression between discrete disease groups rather than testing features along an ordered progression trajectory
- Data are pathomics/structure-level without cell-cluster proportion annotations, so cell_importance's per-cluster proportion fits are undefined without adaptation

## Inputs

- AnnData (.h5ad) single-cell matrix with sampleID, Status, and cell_subtype/cell-cluster annotations
- Sample-by-sample Wasserstein distance matrix computed by PILOT (POT emd2, cosine-distance cost matrix in PCA space)
- Pseudotime per sample from diffusion maps + EIPLGraph principal graph (requires a root/source node)
- Per-sample cell-cluster proportion table (Dirichlet-multinomial smoothed with pseudo cell counts)

## Outputs

- Ranked cell-cluster importance table (clusters whose proportions change linearly, quadratically, or linear-quadratically along pseudotime)
- Per-gene fit statistics per cell type (genes_importance output)
- 'gene_clusters_stats_extend.csv' with Wald statistic, p-value, fold change (FC), and expression-pattern columns per gene per cluster

## How to apply

After embedding the sample Wasserstein-distance matrix with diffusion maps and fitting the EIPLGraph principal graph (e.g., source_node=7, a control sample, in the myocardial infarction study), run cell_importance, which fits robust regression models with linear, quadratic, and linear-quadratic terms for each cell cluster's proportion and uses statistical tests to judge goodness of fit. Then run genes_importance per cell type to fit gene expression along pseudotime, and gene_cluster_differentiation (e.g., cellnames=['healthy_CM','Myofib'], number_genes=70) which applies Wald statistics via linear combinations of coefficients to test differences between pair points of fitted curves. Export 'gene_clusters_stats_extend.csv', filter to genes with fold change > 0.5, and sort by Wald test p-value. Robust (Huber) regression is chosen to limit the influence of outlier patients on proportion/expression fits along pseudotime.

## Related tools

- **PILOT** (provides cell_importance (Huber regression per cluster), genes_importance, and gene_cluster_differentiation (Wald tests)) — github.com/CostaLab/PILOT
- **POT library** (emd2-based Wasserstein distances feed the trajectory whose pseudotime the regressions are fit against) — https://github.com/CostaLab/PILOT
- **scanpy** (loads/preprocesses the .h5ad AnnData before PILOT trajectory estimation) — https://github.com/CostaLab/PILOT
- **limma** (PILOT reuses limma's empirical Bayes approach for gene-expression marker contrasts)

## Examples

```
pl.tl.gene_cluster_differentiation(cellnames=['healthy_CM','Myofib'], number_genes=70, sample_col='sampleID', col_cell='cell_subtype')
```

## Evaluation signals

- Output table 'gene_clusters_stats_extend.csv' exists and contains Wald statistic, p-value, FC, and expression-pattern columns
- Filtering at FC > 0.5 reproduces the reported MI hallmarks: decrease of healthy cardiomyocytes, increase of fibroblasts/myofibroblasts and myeloid cells (e.g., MYBPC3/MYOM1/FHOD3 in healthy_CM)
- Top hits include the quadratic increase of SPP1-positive macrophages and myofibroblasts and quadratic decrease of healthy cardiomyocytes, matching expected early MI remodeling
- Robust fits are stable: linear, quadratic, and linear-quadratic models are each evaluated and the best-fitting term per feature is selected via the goodness-of-fit tests

## Limitations

- PILOT does not explicitly correct for technical artefacts; it only provides statistical tests to evaluate their influence on predictions
- The EIPLGraph root sample was visually selected rather than chosen by an objective rule, which limits reproducibility of the pseudotime the regressions depend on
- Robust regression reduces but does not eliminate sensitivity to trajectory outliers (e.g., PILOT excluded four pancreas E12-E15 samples as trajectory outliers)

## Evidence

- [discussion] step-wise huber regression: "it fits regression models with linear, quadratic, and linear-quadratic terms for each feature and uses statistical tests to determine the goodness of fit."
- [discussion] wald test: "We utilized Wald statistics, using the linear combination of coefficients, to test whether there are differences between pair points of fitted curves"
- [abstract] purpose: "PILOT provides a statistical approach to find changes in cell populations, gene expression, and tissue structures related to the trajectories or clusters"
- [discussion] technical artifacts: "While PILOT does not explicitly correct for such potential technical artefacts, it provides statistical tests to evaluate if technical factors might influence the predictions."
- [discussion] root selection gap: "EIPLGraph also requires a root sample, which was visually selected to reflect parts of the trajectory with control samples."
