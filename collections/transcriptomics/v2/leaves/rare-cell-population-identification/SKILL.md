---
name: rare-cell-population-identification
description: Use when a broadly-defined scRNA-seq cluster is suspected heterogeneous
  (e.g. a stromal/mesenchymal cluster that may hide CAF subtypes, or immune clusters
  hiding fine-grained subtypes) and rare, low-abundance subpopulations must be recovered
  from Seurat-processed data.
when_to_use_negative:
- Broad clusters already have high median silhouette scores (homogeneous) — reclustering
  there adds noise, since SCISSORS targets clusters with low median silhouette scores.
- Input is not single-cell RNA-seq data requiring subclustering (e.g. an already fine-grained
  cell-type-annotated dataset or a bulk/feature table) — SCISSORS is designed for
  subcluster discovery within broadly-defined scRNA-seq clusters.
- The question is de novo clustering of an unclustered dataset rather than recovering
  rare subpopulations hidden inside existing broad clusters — existing methods focused
  on the optimal number of robust clusters are the wrong framing here, but so is treating
  SCISSORS as a general first-pass clustering tool.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SCISSORS
  role: 'Core framework: silhouette scoring for heterogeneity estimation plus multi-step
    semi-supervised iterative reclustering to reveal rare cells'
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Underlying single-cell pipeline (NormalizeData, FindVariableFeatures, ScaleData,
    RunPCA, FindNeighbors, DimPlot) that SCISSORS wraps and extends
- name: cluster
  role: Clustering tools used for silhouette computation in the R workflow
- name: ggplot2
  role: Plotting library for silhouette distributions and final subcluster visualizations
    saved via ggsave
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:2e68f049b294e7cbb20b20ec23294e1203c1372a7bfd9ddd9877478f9640e45a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SCISSORS
  - Seurat
  - cluster
  - ggplot2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# rare-cell-population-identification

## Summary

SCISSORS-based skill for recovering extremely low-abundance cell populations by splitting heterogeneous broad clusters via silhouette-scored, semi-supervised iterative reclustering wrapped around Seurat. It enables identification of rare subtypes (e.g. 23-cell apCAF populations) that standard unsupervised clustering misses.

## When to use

Use when a broadly-defined scRNA-seq cluster is suspected heterogeneous (e.g. a stromal/mesenchymal cluster that may hide CAF subtypes, or immune clusters hiding fine-grained subtypes) and rare, low-abundance subpopulations must be recovered from Seurat-processed data.

## When NOT to use

- Broad clusters already have high median silhouette scores (homogeneous) — reclustering there adds noise, since SCISSORS targets clusters with low median silhouette scores.
- Input is not single-cell RNA-seq data requiring subclustering (e.g. an already fine-grained cell-type-annotated dataset or a bulk/feature table) — SCISSORS is designed for subcluster discovery within broadly-defined scRNA-seq clusters.
- The question is de novo clustering of an unclustered dataset rather than recovering rare subpopulations hidden inside existing broad clusters — existing methods focused on the optimal number of robust clusters are the wrong framing here, but so is treating SCISSORS as a general first-pass clustering tool.

## Inputs

- Seurat object with raw UMI counts (e.g. the Elyada human PDAC scRNA-seq Seurat object)
- Broad cluster assignments from an initial Louvain/Seurat clustering
- Dimensionality-reduction embeddings (PCA, UMAP/t-SNE) from the Seurat object
- Known marker gene sets for candidate rare subpopulations (e.g. apCAF markers for VAM validation)

## Outputs

- Per-cell silhouette score distribution per broad cluster (silhouette-score data frame)
- Subcluster assignments splitting the heterogeneous cluster (e.g. CAF subtypes, endothelial, perivascular cells)
- Subcluster-specific marker genes used for annotation
- VAM enrichment results confirming rare population identity
- t-SNE/UMAP visualization with subcluster labels (e.g. SCISSORS_Clusters.pdf via ggsave)

## How to apply

First preprocess the scRNA-seq data through the Seurat pipeline (NormalizeData, FindVariableFeatures with vst and 2000 features, ScaleData, RunPCA, FindNeighbors) and obtain broad cluster labels, or run SCISSORS::PrepareData with chosen n.HVG, n.PC, dimensionality reduction (e.g. umap) and initial Louvain resolution. Subset the target cluster (e.g. Cluster 8, the activated stellate/mesenchymal cluster in the Elyada PDAC dataset) and compute per-cell silhouette scores with ComputeSilhouetteScores; clusters with low median silhouette scores are the reclustering targets, since SCISSORS uses silhouette scoring to estimate cluster heterogeneity. Apply the multi-step semi-supervised iterative reclustering optimization to split the heterogeneous cluster into subclusters. Annotate subclusters by examining subcluster-specific marker genes (e.g. the three CAF subpopulations plus endothelial and perivascular cells), and validate any rare recovered population by VAM enrichment of its originally defined marker gene set. Finally visualize subcluster assignments on the t-SNE/UMAP embedding with DimPlot and export (e.g. ggsave to SCISSORS_Clusters.pdf).

## Related tools

- **SCISSORS** (Core framework: silhouette scoring for heterogeneity estimation plus multi-step semi-supervised iterative reclustering to reveal rare cells) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Underlying single-cell pipeline (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, DimPlot) that SCISSORS wraps and extends)
- **cluster** (Clustering tools used for silhouette computation in the R workflow)
- **ggplot2** (Plotting library for silhouette distributions and final subcluster visualizations saved via ggsave)

## Examples

```
library(SCISSORS); seu <- PrepareData(seu, n.HVG = 4000, n.PC = 15, which.dim.reduc = "umap", initial.resolution = 0.4); sils <- ComputeSilhouetteScores(seu, avg = FALSE)
```

## Evaluation signals

- Rare subcluster is recovered at the reported abundance — e.g. the apCAF population at 23 cells, 0.092% of the Elyada PDAC dataset.
- Recovered rare population shows significant VAM enrichment of the originally defined marker gene set (e.g. apCAF markers).
- Silhouette-score distributions identify low-median-score clusters as reclustering targets; skipped clusters should not be heterogeneous.
- Subclusters are annotatable from subcluster-specific marker genes (three CAF subpopulations plus endothelial and perivascular cells in the PDAC case).
- Reclustering is reproducible and integrated into the existing Seurat pipeline (source code and vignettes available at the SCISSORS GitHub repo).

## Limitations

- Even small differences in analysis and parameter choice can greatly alter scRNA-seq clustering results and thus influence which cell types are identified.
- Existing methods focusing on the optimal number of robust clusters are problematic for extremely low-abundance cells due to their subtle contributions — rare populations recovered are inherently small (e.g. 0.092% of the dataset).
- No discussion section was present in the extracted document, so limitations/reproducibility gaps could not be extracted from that section.

## Evidence

- [abstract] SCISSORS framework for rare cell identification: "we present a carefully designed framework, SCISSORS, which accurately profiles subclusters within broad cluster(s) for the identification of rare cell types in scRNA-seq data"
- [abstract] Silhouette scoring and semi-supervised reclustering: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [intro] CAF subtype identification use case: "use SCISSORS to identify the CAF subtypes within the larger stroma population"
- [methods] Silhouette computation call: "sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)"
- [readme] Iterative reclustering as Seurat extension: "uncover rare celltypes and other small subgroups using an iterative reclustering optimization routine built as an extension to a typical `Seurat` workflow"
