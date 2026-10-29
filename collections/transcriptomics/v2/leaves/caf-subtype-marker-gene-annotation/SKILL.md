---
name: caf-subtype-marker-gene-annotation
description: Use when reclustering a broad stromal/mesenchymal cluster has produced
  subclusters and each must be assigned a CAF subtype identity, especially when validating
  a rare subcluster against a reference marker set.
when_to_use_negative:
- The cells have not yet been split into subclusters — broad-cluster-level marker
  annotation should be done first with standard Seurat unsupervised clustering
- The tissue of interest has no established CAF subtype marker gene sets to compare
  against, making subtype-level annotation and VAM validation underdetermined
- The goal is de novo discovery of marker genes rather than assigning cells to known
  subpopulations
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCISSORS
  role: Provides the iterative reclustering that generates subclusters and a method
    for identification of highly specific marker genes used in annotation
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Underlying single-cell pipeline; DimPlot visualizes annotated subclusters
    on the t-SNE embedding
  repo: https://github.com/jr-leary7/SCISSORS
- name: ggplot2
  role: Plotting layer; ggsave saves the annotated subcluster figure as SCISSORS_Clusters.pdf
  repo: https://github.com/jr-leary7/SCISSORS
- name: cluster
  role: Clustering tools used within the SCISSORS workflow that produces the subclusters
    being annotated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:b3ed53041241f851157bf3a22d518b98ad103d6339a7f3a060af04365a9e4d10
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# caf-subtype-marker-gene-annotation

## Summary

Annotates SCISSORS-derived subclusters as CAF subpopulations (e.g. apCAF) plus endothelial and perivascular cells by examining subcluster-specific marker genes, and validates rare populations via VAM enrichment of a predefined marker gene set (e.g. the originally defined apCAF markers in the Elyada PDAC data).

## When to use

Use when reclustering a broad stromal/mesenchymal cluster has produced subclusters and each must be assigned a CAF subtype identity, especially when validating a rare subcluster against a reference marker set.

## When NOT to use

- The cells have not yet been split into subclusters — broad-cluster-level marker annotation should be done first with standard Seurat unsupervised clustering
- The tissue of interest has no established CAF subtype marker gene sets to compare against, making subtype-level annotation and VAM validation underdetermined
- The goal is de novo discovery of marker genes rather than assigning cells to known subpopulations

## Inputs

- SCISSORS-reclustered Seurat object with subcluster assignments (e.g. subclusters of Elyada PDAC Cluster 8)
- Reference CAF subtype marker gene sets (e.g. the originally defined apCAF marker genes from Elyada et al.)
- Normalized scRNA-seq count matrix prepared via NormalizeData / FindVariableFeatures / ScaleData / RunPCA

## Outputs

- Per-subcluster CAF subtype annotations (myCAF, iCAF, apCAF, endothelial, perivascular)
- VAM enrichment results confirming rare subcluster identity (e.g. significant apCAF marker enrichment for the 23-cell / 0.092% apCAF subcluster)
- t-SNE visualization of annotated subclusters (SCISSORS_Clusters.pdf via DimPlot + ggsave)

## How to apply

After the SCISSORS iterative reclustering (silhouette scoring via ComputeSilhouetteScores followed by multi-step semi-supervised reclustering wrapped around the Seurat pipeline) splits a broad cluster such as the Elyada PDAC Cluster 8 (activated stellate/mesenchymal stem cell cluster), examine subcluster-specific marker genes to annotate each subcluster as one of the CAF subpopulations (myCAF, iCAF, apCAF) plus endothelial and perivascular cells. SCISSORS itself provides a method for identifying highly specific marker genes for this annotation step. Confirm a suspected rare subcluster — e.g. the apCAF population (23 cells, 0.092% of the dataset), absent from the original human analysis — by running VAM enrichment of the originally defined apCAF marker genes and requiring significant enrichment. Visualize the annotated subcluster assignments on the t-SNE embedding with DimPlot and save the figure (e.g. ggsave as SCISSORS_Clusters.pdf).

## Related tools

- **SCISSORS** (Provides the iterative reclustering that generates subclusters and a method for identification of highly specific marker genes used in annotation) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Underlying single-cell pipeline; DimPlot visualizes annotated subclusters on the t-SNE embedding) — https://github.com/jr-leary7/SCISSORS
- **ggplot2** (Plotting layer; ggsave saves the annotated subcluster figure as SCISSORS_Clusters.pdf) — https://github.com/jr-leary7/SCISSORS
- **cluster** (Clustering tools used within the SCISSORS workflow that produces the subclusters being annotated)

## Examples

```
library(SCISSORS); library(Seurat); sil <- ComputeSilhouetteScores(seu_cluster8, avg = FALSE); DimPlot(seu_cluster8, group.by = "subcluster") + ggsave("SCISSORS_Clusters.pdf")
```

## Evaluation signals

- The rare apCAF subcluster is recovered at the reported size: 23 cells, 0.092% of the dataset
- VAM enrichment of the originally defined apCAF marker genes is significant in the rare subcluster
- Subcluster-specific marker genes distinguish the annotated CAF subpopulations (myCAF, iCAF, apCAF) plus endothelial and perivascular cells
- Annotated subcluster assignments are consistent with the original Elyada et al. CAF subtype definitions when projected onto the same data

## Limitations

- Small differences in analysis and parameter choice can greatly alter scRNA-seq clustering results, which influences which cell types are identified
- Existing methods focusing on the optimal number of robust clusters are problematic for extremely low-abundance cells due to their subtle contributions
- Rare subclusters may be absent from the original publication's annotations (as the human apCAF population was), so validation depends on external marker gene sets rather than published cluster labels

## Evidence

- [results] SCISSORS reanalysis of the same human PDAC scRNA-seq data identified the rare apCAF population (23 cells, 0.092% of the dataset), confirmed by significant VAM enrichment of the originally defined apCAF marker genes.: [evidence span withheld — anchor status: not_found]
- [abstract] SCISSORS provides a method for identification of highly specific marker genes.: "Additionally, SCISSORS provides a method for the identification of marker genes of high specificity to the cell type"
- [abstract] SCISSORS reveals rare cells in heterogeneous clusters by multi-step semi-supervised reclustering using silhouette scoring.: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [intro] The paper's intro explicitly cites CAF subtype identification as a target use case.: "use SCISSORS to identify the CAF subtypes within the larger stroma population"
- [readme] The README positions SCISSORS as an extension to a typical Seurat workflow with tools to preprocess, optimize reclustering, and annotate results.: "Tools are included to preprocess data, identify potential targets for subgroup analysis, optimize reclustering, and annotate results."
