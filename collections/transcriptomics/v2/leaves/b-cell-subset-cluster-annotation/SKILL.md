---
name: b-cell-subset-cluster-annotation
description: Use when analyzing CITE-seq data from CD19+ B cells where surface protein
  expression must be integrated with transcriptome to resolve functionally distinct
  B cell subsets, e.g. spike-specific memory-like clusters in COVID-19 patient PBMC.
when_to_use_negative:
- Datasets with a very high number of ADT features (e.g. >50 ADTs or 228-ADT 170K-cell
  data) — LinQ-View failed there while Seurat 4 succeeded
- Datasets too large for quadratic pairwise distance computation, where LinQ-View's
  time/memory efficiency is limiting
- Purely unimodal scRNA-seq data without a surface protein (ADT) modality to fuse
  via the L∞ norm
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: LinQ-View
  role: 'Multimodal integration: computes per-modality pairwise distances (with sigmoid
    filter on ADT), fuses them via L∞ norm, and clusters CITE-seq B cell data'
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat
  role: Hosts the default Louvain community detection clustering and pre-processing
    (QC, normalization, scaling) used by LinQ-View
  repo: https://satijalab.org/seurat/
- name: R v3.6.3
  role: Runtime environment for the LinQ-View/Seurat pipeline
- name: ggplot2
  role: Generates Figure 7-style annotated cluster visualizations
  repo: https://ggplot2.tidyverse.org/
- name: umap
  role: Non-linear dimension reduction for embedding fused clusters
  repo: https://github.com/lmcinnes/umap
- name: dplyr
  role: Data manipulation for cluster annotation and export tables
  repo: https://dplyr.tidyverse.org/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:29046def3d88415461590f8c7304e2db564c6a20c06284f570e48b8f59ff4afd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - R
  - R v3.6.3
  - Seurat
  - dplyr
  - ggplot2
  - umap
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# b-cell-subset-cluster-annotation

## Summary

Identifies and annotates B cell subsets from CITE-seq data using LinQ-View multimodal clustering, distinguishing naive-like, polyreactive, and antigen-specific memory-like populations in SARS-CoV-2-infected patient samples by fusing transcriptome and ADT pairwise distances.

## When to use

Use when analyzing CITE-seq data from CD19+ B cells where surface protein expression must be integrated with transcriptome to resolve functionally distinct B cell subsets, e.g. spike-specific memory-like clusters in COVID-19 patient PBMC.

## When NOT to use

- Datasets with a very high number of ADT features (e.g. >50 ADTs or 228-ADT 170K-cell data) — LinQ-View failed there while Seurat 4 succeeded
- Datasets too large for quadratic pairwise distance computation, where LinQ-View's time/memory efficiency is limiting
- Purely unimodal scRNA-seq data without a surface protein (ADT) modality to fuse via the L∞ norm

## Inputs

- CITE-seq transcriptome count matrix (e.g. 10x mtx/h5 from Mendeley dataset 3jdywv5jrv/3, CD19+ B cells from 10 SARS-CoV-2-infected patients)
- CITE-seq ADT (surface protein) count matrix from the same cells
- Per-cell metadata (sample/patient ID, isotype/probe binding where available)

## Outputs

- Cluster assignment table with UMAP coordinates and sample fields for all cells (linqview_dataset6_bcell_clusters.csv)
- Figure 7-style annotated cluster plots (PNG) marking naive-like, polyreactive, and spike-specific memory-like B cell subsets

## How to apply

Download the CITE-seq dataset (e.g. Mendeley deposit 3jdywv5jrv/3, Dataset 6: CD19+ B cells from 10 COVID-19 patients) and load transcriptome and ADT count matrices in R v3.6.3 with LinQ-View/Seurat. Pre-process by excluding cells with fewer than 200 or more than 2500 detected genes, applying a mitochondrial soft threshold set to the 95th percentile of the dataset distribution with a 10% sealing point, then normalizing and scaling both modalities. Compute cell-cell pairwise distances per modality separately, applying the sigmoid filter to the ADT modality so a single surface marker cannot dominate, and fuse the matrices with the L∞ norm joint distance model. Cluster with the default Louvain community detection in Seurat and embed with UMAP/t-SNE. Annotate clusters by cross-referencing ADT binding and isotype patterns: naive-like RNA clusters 0/1 split into LinQ-View clusters 0/3/4/5 (non-responder, polyreactive, HLA-DR+, CD62L+), clusters 3 and 8 bind all ADTs and COVID-19 probes, and spike-specific B cells enrich in memory-like clusters 6, 7, and 9 with elevated somatic hypermutation and IgA/G class switching. Export cluster assignments with UMAP coordinates to a CSV for validation against Figure 7 assignments.

## Related tools

- **LinQ-View** (Multimodal integration: computes per-modality pairwise distances (with sigmoid filter on ADT), fuses them via L∞ norm, and clusters CITE-seq B cell data) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat** (Hosts the default Louvain community detection clustering and pre-processing (QC, normalization, scaling) used by LinQ-View) — https://satijalab.org/seurat/
- **R v3.6.3** (Runtime environment for the LinQ-View/Seurat pipeline)
- **ggplot2** (Generates Figure 7-style annotated cluster visualizations) — https://ggplot2.tidyverse.org/
- **umap** (Non-linear dimension reduction for embedding fused clusters) — https://github.com/lmcinnes/umap
- **dplyr** (Data manipulation for cluster annotation and export tables) — https://dplyr.tidyverse.org/

## Examples

```
devtools::install_github("WilsonImmunologyLab/LinQView"); q <- CreateLinQViewObject(rna, adt); q <- RunLinQView(q); write.csv(q@meta.data, "linqview_dataset6_bcell_clusters.csv")
```

## Evaluation signals

- Derived clusters match reported Figure 7 assignments: RNA clusters 0/1 partition into LinQ-View clusters 0, 3, 4, 5 and antigen-specific cells enrich in memory-like clusters 6, 7, 9
- Output CSV exists with cluster, UMAP, and sample fields populated for all cells
- Clusters 3 and 8 bind all ADTs and COVID-19 probes, consistent with the polyreactive/plasmablast-like annotation
- Spike-specific memory-like clusters (6, 7, 9) show increased somatic hypermutation and IgA/G class switching relative to naive-like clusters
- Cluster purity is quantitatively assessable via the LinQ-View purity score / ROGUE

## Limitations

- Pairwise distance computation scales quadratically with cell number, limiting time and memory efficiency on large datasets
- LinQ-View failed on a massive dataset (170K cells, 228 ADT features) where Seurat 4 succeeded
- LinQ-View is best suited to large datasets with a limited number of ADT features (e.g. <50)

## Evidence

- [results] Spike-specific B cells were enriched in three memory-like B cell subsets: [evidence span withheld — anchor status: not_found]
- [intro] L∞ norm integration: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [discussion] QC thresholds: "all cells that have less than 200 genes detected or more than 2500 genes detected have been excluded"
- [discussion] Mitochondrial soft threshold: "A soft threshold was set to the 95th percentile of the current dataset distribution, and the soft threshold was subject to a sealing point as the maximum threshold"
- [discussion] Sigmoid filter: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [discussion] Dataset 6 source: "is available from Mendeley data: https://data.mendeley.com/datasets/3jdywv5jrv/3"
- [discussion] ADT feature scope: "LinQ-View is the best option on large datasets with a limited number of ADT features (e.g., <50)"
- [discussion] Default clustering: "The default cell population clustering method in LinQ-View is a community detection algorithm implemented in Seurat"
- [readme] ROGUE purity concept: [evidence span withheld — anchor status: not_found]
- [readme] LinQ-View modality rationale: "calculate the dissimilarity representations for each single cell profiling modality (i.e. mRNA and surface protein expression) and then combine the dissimilarities from each modality"
