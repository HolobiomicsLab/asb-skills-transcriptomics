---
name: single-cell-rnaseq-subclustering
description: Use when a broad Seurat cluster is suspected to contain rare subpopulations
  (e.g. low median silhouette score, known rare cell types like CAF subtypes absent
  from original annotations) and standard Louvain clustering failed to resolve them.
when_to_use_negative:
- The dataset has already been resolved into fine-grained, homogeneous cell type labels
  with no low-silhouette heterogeneous clusters remaining.
- The goal is to determine the optimal number of robust top-level clusters rather
  than recover extremely low-abundance subgroups within an existing cluster (existing
  methods focusing on optimal cluster count are the intended alternative).
- Input is not single-cell expression data (e.g. bulk RNA-seq or a precomputed feature
  table without per-cell resolution).
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCISSORS
  role: 'Core framework: silhouette scoring for heterogeneity estimation plus multi-step
    semi-supervised iterative reclustering of target clusters'
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Underlying single-cell pipeline SCISSORS wraps around (NormalizeData, FindVariableFeatures,
    RunPCA, FindNeighbors, DimPlot)
- name: cluster
  role: R clustering tools used for silhouette computation within the reclustering
    workflow
- name: ggplot2
  role: Plotting silhouette distributions and subcluster visualizations; ggsave for
    figure export
- name: dplyr
  role: Tidy data manipulation within the R workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:9e466cd1aab10eb69592e1514cf85071fc8bbabb43cf6f60267fbeec43d56758
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SCISSORS
  - Seurat
  - cluster
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-rnaseq-subclustering

## Summary

Iterative reclustering of heterogeneous scRNA-seq clusters using SCISSORS, which combines per-cell silhouette scoring with multi-step semi-supervised reclustering wrapped around Seurat to recover rare cell subpopulations missed by standard unsupervised clustering (e.g. the 23-cell apCAF population in the Elyada human PDAC dataset).

## When to use

Use when a broad Seurat cluster is suspected to contain rare subpopulations (e.g. low median silhouette score, known rare cell types like CAF subtypes absent from original annotations) and standard Louvain clustering failed to resolve them.

## When NOT to use

- The dataset has already been resolved into fine-grained, homogeneous cell type labels with no low-silhouette heterogeneous clusters remaining.
- The goal is to determine the optimal number of robust top-level clusters rather than recover extremely low-abundance subgroups within an existing cluster (existing methods focusing on optimal cluster count are the intended alternative).
- Input is not single-cell expression data (e.g. bulk RNA-seq or a precomputed feature table without per-cell resolution).

## Inputs

- Seurat scRNA-seq object with raw counts (e.g. Elyada human PDAC dataset, PBMC3k from SeuratData)
- Pre-defined marker gene sets for subcluster validation (e.g. apCAF markers for VAM enrichment)
- Target cluster identifier or subset of cells suspected to be heterogeneous (e.g. Cluster 8)

## Outputs

- Subcluster assignments per cell within the reclustered broad cluster
- Per-cell silhouette score table (ComputeSilhouetteScores output)
- Subcluster-specific marker genes for annotation
- Visualization of subclusters on UMAP/t-SNE embedding (e.g. DimPlot saved as SCISSORS_Clusters.pdf via ggsave)

## How to apply

First preprocess the Seurat object with NormalizeData, FindVariableFeatures (vst, 2000 features), ScaleData, and RunPCA, then run an initial clustering (e.g. Louvain at resolution ~0.4 via PrepareData) to obtain broad clusters. Estimate heterogeneity with ComputeSilhouetteScores on each cell and compare median silhouette distributions per cluster: clusters with lower median scores are reclustering targets. Apply the SCISSORS iterative semi-supervised reclustering to split the target cluster into subclusters, then annotate subclusters by examining subcluster-specific marker genes (e.g. CAF subtypes within the stromal Cluster 8 of the Elyada PDAC data). Validate rare subclusters with an independent enrichment test such as VAM against a pre-defined marker gene set, and confirm expected cell counts (e.g. the apCAF subcluster of 23 cells, 0.092% of the dataset).

## Related tools

- **SCISSORS** (Core framework: silhouette scoring for heterogeneity estimation plus multi-step semi-supervised iterative reclustering of target clusters) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Underlying single-cell pipeline SCISSORS wraps around (NormalizeData, FindVariableFeatures, RunPCA, FindNeighbors, DimPlot))
- **cluster** (R clustering tools used for silhouette computation within the reclustering workflow)
- **ggplot2** (Plotting silhouette distributions and subcluster visualizations; ggsave for figure export)
- **dplyr** (Tidy data manipulation within the R workflow)

## Examples

```
seu_sub <- SCISSORS::ReclusterClusters(seu, group.by = "seurat_clusters", clusters = 8); ComputeSilhouetteScores(seu_sub, avg = FALSE)
```

## Evaluation signals

- Rare subcluster is recovered at the reported size and proportion (e.g. apCAF: 23 cells, 0.092% of the Elyada PDAC dataset).
- Reclustered rare subcluster shows significant VAM enrichment of the pre-defined marker gene set (apCAF markers).
- Silhouette score distributions identify target clusters correctly: target clusters (e.g. PBMC clusters 0, 1, 3) have lower median silhouette scores than homogeneous ones.
- Subclusters are annotated consistently with subcluster-specific marker genes (e.g. three CAF subtypes plus endothelial and perivascular cells from Cluster 8).
- Subcluster assignments visualize coherently on the t-SNE/UMAP embedding without scattering across unrelated regions.

## Limitations

- Small differences in analysis and parameter choice can greatly alter scRNA-seq clustering results, influencing which cell types are identified.
- Extremely low-abundance cells contribute subtly toward global clustering metrics, which is why standard optimal-cluster-number methods fail; SCISSORS is designed for this regime but parameter sensitivity remains.
- SCISSORS is an R package wrapped around Seurat; it requires a Seurat object and does not natively operate on other single-cell formats (e.g. scanpy/AnnData).

## Evidence

- [abstract] silhouette + semi-supervised reclustering rationale: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [abstract] Seurat integration: "SCISSORS is wrapped around the popular Seurat R package and can be easily integrated into existing Seurat pipelines"
- [intro] CAF subclustering motivation: "use SCISSORS to identify the CAF subtypes within the larger stroma population"
- [methods] preprocessing parameters: "FindVariableFeatures(selection.method = "vst", nfeatures = 2000, verbose = FALSE)"
- [readme] silhouette computation call: [evidence span withheld — anchor status: external_doc]
- [abstract] parameter sensitivity caveat: "even small differences in analysis and parameter choosing can greatly alter clustering results and thus impose great influence on which cell types are identified"
