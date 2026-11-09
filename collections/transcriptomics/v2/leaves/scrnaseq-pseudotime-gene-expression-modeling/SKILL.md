---
name: scrnaseq-pseudotime-gene-expression-modeling
description: Use when a sample-level trajectory (PILOT diffusion-map embedding + EIPLGraph
  pseudotime, e.g. from myocardial_infarction.h5ad) already exists and you must rank
  cell populations changing along progression and identify cluster-specific genes
  via Wald tests.
when_to_use_negative:
- You only need the patient-level distances or trajectory itself — use PILOT's wasserstein_distance/trajectory
  steps, not the regression/Wald-testing skill
- The input is a pathomics (histology structures) dataset without gene expression;
  genes_importance and gene_cluster_differentiation are not applicable to morphometric
  features
- Samples mix technologies or isolation methods (single nucleus vs. single cell, droplet
  vs. well based) — PILOT assumes samples were collected uniformly, so trajectory-fitted
  changes may reflect batch rather than biology
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PILOT
  role: Provides cell_importance (Huber regression per cluster), genes_importance,
    and gene_cluster_differentiation (Wald tests) along the EIPLGraph pseudotime
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: emd2-based Wasserstein distances between patient samples that define the trajectory
    on which genes/clusters are fitted
- name: limma
  role: Empirical Bayes approach underlying PILOT's contrast of gene expression between
    cells/clusters
- name: scanpy
  role: Reads and preprocesses the .h5ad scRNA-seq AnnData (e.g. myocardial_infarction.h5ad)
    before PILOT modeling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:77a0b2d852ab3f5864246ade23b9aada629f5de8a79b5bf5011c4630285fe39b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - limma
  - scanpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# scrnaseq-pseudotime-gene-expression-modeling

## Summary

Fit step-wise Huber regression models (linear, quadratic, linear-quadratic) of cell-cluster proportions and gene expression along a PILOT-estimated disease-trajectory pseudotime, then use Wald statistics to find genes whose fitted curves differ in a target cluster versus all others.

## When to use

Use when a sample-level trajectory (PILOT diffusion-map embedding + EIPLGraph pseudotime, e.g. from myocardial_infarction.h5ad) already exists and you must rank cell populations changing along progression and identify cluster-specific genes via Wald tests.

## When NOT to use

- You only need the patient-level distances or trajectory itself — use PILOT's wasserstein_distance/trajectory steps, not the regression/Wald-testing skill
- The input is a pathomics (histology structures) dataset without gene expression; genes_importance and gene_cluster_differentiation are not applicable to morphometric features
- Samples mix technologies or isolation methods (single nucleus vs. single cell, droplet vs. well based) — PILOT assumes samples were collected uniformly, so trajectory-fitted changes may reflect batch rather than biology

## Inputs

- AnnData (.h5ad) single-cell matrix with per-sample annotations (sampleID, Status, cell_subtype), e.g. myocardial_infarction.h5ad
- PILOT-estimated sample-level Wasserstein distance matrix and pseudotime from EIPLGraph (rooted at a control sample)
- Per-cluster cell proportions / fitted expression curves along pseudotime from pl.tl.cell_importance and pl.tl.genes_importance

## Outputs

- Ranked table of cell clusters with fitted regression terms (linear/quadratic/linear-quadratic) and goodness-of-fit statistics versus disease progression
- gene_clusters_stats_extend.csv with Wald statistic, p-value, fold change (FC), and expression-pattern columns per gene per cluster
- Filtered gene list (FC > 0.5, sorted by Wald p-value) of cluster-differentiated genes, e.g. MYBPC3/MYOM1/FHOD3 in healthy_CM

## How to apply

First define pseudotime by embedding the Wasserstein sample-distance matrix (POT emd2, PCA emb_matrix, clusters_col='cell_subtype', sample_col='sampleID', status='Status') with diffusion maps and fitting the EIPLGraph principal graph from a control-source node (e.g. source_node=7 in the MI study). Then call pl.tl.cell_importance, which fits robust Huber regression models with linear, quadratic, and linear-quadratic terms per cell cluster and uses goodness-of-fit statistical tests to rank clusters whose proportions change with progression. For genes, run pl.tl.genes_importance per cell type (plot_genes=False) to fit expression along pseudotime, then pl.tl.gene_cluster_differentiation (e.g. cellnames=['healthy_CM','Myofib'], number_genes=70), which applies Wald statistics on the linear combination of coefficients to test each gene's fit in the target cluster versus other clusters, using limma's empirical Bayes approach for expression markers. Export 'gene_clusters_stats_extend.csv', filter to genes with fold change > 0.5, and sort by Wald test p-value; confirm reported hallmarks (decrease of healthy cardiomyocytes; quadratic increase of SPP1+ macrophages and Myofib).

## Related tools

- **PILOT** (Provides cell_importance (Huber regression per cluster), genes_importance, and gene_cluster_differentiation (Wald tests) along the EIPLGraph pseudotime) — https://github.com/CostaLab/PILOT
- **POT library** (emd2-based Wasserstein distances between patient samples that define the trajectory on which genes/clusters are fitted)
- **limma** (Empirical Bayes approach underlying PILOT's contrast of gene expression between cells/clusters)
- **scanpy** (Reads and preprocesses the .h5ad scRNA-seq AnnData (e.g. myocardial_infarction.h5ad) before PILOT modeling)

## Examples

```
import pilotpy as pl; import scanpy as sc; adata = sc.read_h5ad('myocardial_infarction.h5ad'); pl.tl.cell_importance(adata); pl.tl.genes_importance(adata, plot_genes=False, sample_col='sampleID', col_cell='cell_subtype'); pl.tl.gene_cluster_differentiation(adata, cellnames=['healthy_CM','Myofib'], number_genes=70)
```

## Evaluation signals

- 'gene_clusters_stats_extend.csv' exists and contains the required columns: Wald statistic, p-value, FC, and expression-pattern
- Reproduces MI hallmarks: decrease of healthy cardiomyocytes, increase of fibroblasts and myeloid cells, with MYBPC3/MYOM1/FHOD3 among top healthy_CM genes
- Top-ranked clusters match the reported quadratic increase of SPP1+ macrophages and myofibroblasts and quadratic decrease of healthy cardiomyocytes
- Fold-change filter (FC > 0.5) applied before p-value sorting of the Wald test results
- Pseudotime root (source_node) is a control sample so the trajectory reflects control-to-disease progression

## Limitations

- The EIPLGraph root sample was visually selected to reflect control samples rather than chosen by an objective rule, affecting reproducibility of the pseudotime that genes are fitted against
- PILOT does not explicitly correct for technical artefacts; it only provides statistical tests to evaluate their influence
- Low-quality samples must be excluded beforehand (e.g. one MI sample with less than 1000 genes on average per cell), and package versions for pilotpy/scanpy are unspecified

## Evidence

- [discussion] Huber regression model terms: "it fits regression models with linear, quadratic, and linear-quadratic terms for each feature and uses statistical tests to determine the goodness of fit."
- [discussion] Wald test definition: "We utilized Wald statistics, using the linear combination of coefficients, to test whether there are differences between pair points of fitted curves"
- [discussion] limma for gene markers: [evidence span withheld — anchor status: not_found]
- [methods] MI tutorial entry point: "See the main [tutorial](https://pilot.readthedocs.io/en/latest/Myocardial_infarction.html), for details."
- [abstract] Purpose of the statistical layer: "PILOT provides a statistical approach to find changes in cell populations, gene expression, and tissue structures related to the trajectories or clusters"
