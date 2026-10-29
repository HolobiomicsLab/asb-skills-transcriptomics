---
name: ca-biplot-visualization-with-feature-labeling
description: Use when you have a scRNA-seq count matrix (e.g., SingleCellExperiment)
  reduced with standard CA and want a geometric joint display of cells and genes to
  identify which features (e.g., top 20 by loading L2 norm) drive cell-population
  separation.
when_to_use_negative:
- Input has already been log(x + 1)-transformed to 'logcounts' and reduced with PCA
  — CA on Pearson residuals expects raw counts, and the biplot interpretation of a
  PCA solution differs
- Data come from multiple batches/samples requiring integration — use corralm (multi-table
  CA) rather than the single-table corral biplot workflow
- The goal is quantitative clustering-accuracy benchmarking (e.g., ARI) rather than
  exploratory visualization — the biplot aids cluster interpretation, not performance
  measurement
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: corral
  role: Computes the standard CA embedding by SVD of the Pearson chi-squared residual
    matrix on the raw scRNA-seq counts, interfacing with SingleCellExperiment
  repo: github.com/laurenhsu1/corral
- name: made4
  role: Provides CA biplot plotting of features and objects in the same space for
    microarray/RNA-seq style visualization
- name: gridExtra
  role: Arranges the biplot panels in the final rendered figure
- name: R
  role: Language environment in which corral, made4, and gridExtra are invoked
- name: Bioconductor
  role: Ecosystem providing SingleCellExperiment classes that corral interfaces with
    directly
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:780b54515f72a4a190a8e3b7fe1352e87058687c05a99689cff0de2809f5d957
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# CA biplot visualization with feature labeling

## Summary

Render a correspondence analysis biplot of the first two CA dimensions showing cells (as points) and high-weight genes (as labeled features) for exploratory single-cell RNA-seq interpretation, using the corral R/Bioconductor package embedding and the made4/gridExtra plotting tools.

## When to use

Use when you have a scRNA-seq count matrix (e.g., SingleCellExperiment) reduced with standard CA and want a geometric joint display of cells and genes to identify which features (e.g., top 20 by loading L2 norm) drive cell-population separation.

## When NOT to use

- Input has already been log(x + 1)-transformed to 'logcounts' and reduced with PCA — CA on Pearson residuals expects raw counts, and the biplot interpretation of a PCA solution differs
- Data come from multiple batches/samples requiring integration — use corralm (multi-table CA) rather than the single-table corral biplot workflow
- The goal is quantitative clustering-accuracy benchmarking (e.g., ARI) rather than exploratory visualization — the biplot aids cluster interpretation, not performance measurement

## Inputs

- scRNA-seq count matrix as a SingleCellExperiment object (e.g., Zhengmix8eq PBMC counts, unlogged)
- CA embedding from corral: cell coordinates and gene loadings on the first two components

## Outputs

- CA biplot PNG of the first two CA dimensions (e.g., zhengmix8ea_ca_biplot.png) with cells as points and top-20 genes by loading L2 norm labeled
- Ranked list of top genes by L2 norm of CA loadings in the first two components

## How to apply

Run standard CA on the raw counts with corral, which computes the embedding by SVD of the Pearson chi-squared residual matrix — importantly without the log(x + 1) logcounts transformation typical of PCA workflows. Rank genes by the L2 norm of their loadings in the first two CA components and select the top features (the paper's Zhengmix8eq example labels the top 20). Plot the first two CA dimensions as a biplot with cells as points and the selected genes as labels, arranging panels with gridExtra; made4 provides the CA biplot machinery for feature/object co-display. The biplot gives a geometric interpretation of features and objects in the same space, so verify that labeled high-weight genes are biologically interpretable relative to known cell populations (e.g., GNLY for NK cells, TYROBP for NK and CD14 monocytes, LYZ, S100A8, CD74, HLA-DRA for myeloid/monocyte populations).

## Related tools

- **corral** (Computes the standard CA embedding by SVD of the Pearson chi-squared residual matrix on the raw scRNA-seq counts, interfacing with SingleCellExperiment) — github.com/laurenhsu1/corral
- **made4** (Provides CA biplot plotting of features and objects in the same space for microarray/RNA-seq style visualization)
- **gridExtra** (Arranges the biplot panels in the final rendered figure)
- **R** (Language environment in which corral, made4, and gridExtra are invoked)
- **Bioconductor** (Ecosystem providing SingleCellExperiment classes that corral interfaces with directly)

## Examples

```
library(corral); sce <- corral(sce, whichmat = 'counts'); plot_embedding_sce(sce, which_embedding = 'corral', color_attr = 'clusters'); # then label top-20 genes by L2 norm of loadings and arrange with gridExtra
```

## Evaluation signals

- The rendered biplot file exists and renders correctly (e.g., zhengmix8ea_ca_biplot.png)
- The labeled top-20 features by loading L2 norm include GNLY, TYROBP, LYZ, S100A8, CD74, and HLA-DRA on Zhengmix8eq
- Gene arrow/label positions in the biplot correspond to their loadings in the first two CA components (furthest from the origin = highest weight)
- The CA embedding was computed on raw counts, not logcounts — verify no log(x + 1) transformation was applied before SVD of the Pearson residuals
- Cells plot in the same two-dimensional space as the labeled genes, confirming a true biplot (shared coordinate system)

## Limitations

- High-weight genes and biplot interpretation depend on cell identities that may not be independently validated; the paper notes its benchmarking datasets lacked independently validated cell type annotations
- CA on raw Pearson residuals does not adjust for overdispersion of scRNA-seq counts — the paper's adapted CA variants (e.g., CA-FT with Freeman-Tukey residuals) outperform standard CA in clustering accuracy in 8 of 9 datasets
- No tool or package versions are reported for corral or its dependencies in the source

## Evidence

- [intro] We show that the CA biplot provides a geometric interpretation of features and objects in the same space, which in turn facilitates efficient exploratory data analysis and cluster interpretation.: "We show that the CA biplot provides a geometric interpretation of features and objects in the same space, which in turn facilitates efficient exploratory data analysis and cluster interpretation."
- [intro] CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation: [evidence span withheld — anchor status: not_found]
- [intro] In made4, Culhane et al. implemented CA for microarray and bulk RNA-seq data: "In made4, Culhane et al. implemented CA for microarray and bulk RNA-seq data"
- [readme] It can be used on a single table for dimensionality reduction, or it can be used for integration across batches, samples, and modes.: "It can be used on a single table for dimensionality reduction, or it can be used for integration across batches, samples, and modes."
