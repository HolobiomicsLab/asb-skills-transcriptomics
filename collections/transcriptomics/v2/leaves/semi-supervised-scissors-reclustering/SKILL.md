---
name: semi-supervised-scissors-reclustering
description: Use when a scRNA-seq clustering (e.g., Seurat Louvain) yields broad clusters
  suspected of containing rare or low-abundance subtypes that standard resolution
  sweeps miss.
when_to_use_negative:
- When the goal is only to determine the optimal number of robust whole-dataset clusters
  — existing methods like MultiK or SAFE-clustering target that, while SCISSORS targets
  rare subgroups within heterogeneous clusters.
- When input data is already a normalized feature matrix or cell-type-annotated object
  with no broad clusters to subdivide; the iterative reclustering step adds no information.
- When clusters show uniformly high silhouette scores with no low-median heterogeneity
  candidates, since there is no target for subgroup analysis.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCISSORS
  role: 'Core package: PrepareData preprocessing, silhouette-score-based heterogeneity
    estimation, semi-supervised iterative reclustering, and annotation of rare subclusters'
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Underlying single-cell infrastructure SCISSORS wraps; supplies NormalizeData,
    FindVariableFeatures (vst, 2000), ScaleData, RunPCA, FindNeighbors, DimPlot
- name: SeuratData
  role: Provides the pbmc3k example dataset loaded via LoadData
- name: cluster
  role: Supplies clustering/silhouette tooling used for silhouette scoring comparisons
- name: ggplot2
  role: Renders cluster visualization and silhouette-score distribution plots
- name: dplyr
  role: Tidy data manipulation within the R workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:09973c4d0e0ef4b82aa5a6c9c7b540820e6131ec66c85d0d3fda3108bf3de6b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SAFEclustering
  - SCISSORS
  - Seurat
  - SeuratData
  - cluster
  - dplyr
  - ggplot2
  - ggsignif
  - giniclust3
  - magrittr
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# Semi-supervised SCISSORS reclustering of scRNA-seq clusters

## Summary

Apply the SCISSORS R package, wrapped around a Seurat workflow, to detect rare cell types by scoring cluster heterogeneity with per-cell silhouette scores and iteratively reclustering heterogeneous broad clusters in a semi-supervised manner.

## When to use

Use when a scRNA-seq clustering (e.g., Seurat Louvain) yields broad clusters suspected of containing rare or low-abundance subtypes that standard resolution sweeps miss.

## When NOT to use

- When the goal is only to determine the optimal number of robust whole-dataset clusters — existing methods like MultiK or SAFE-clustering target that, while SCISSORS targets rare subgroups within heterogeneous clusters.
- When input data is already a normalized feature matrix or cell-type-annotated object with no broad clusters to subdivide; the iterative reclustering step adds no information.
- When clusters show uniformly high silhouette scores with no low-median heterogeneity candidates, since there is no target for subgroup analysis.

## Inputs

- Raw scRNA-seq count matrix as a Seurat object (e.g., PBMC3k from SeuratData)
- Cell QC metadata (e.g., nFeature_RNA for the < 2500 filter)
- Reclustering parameters (n.HVG, n.PC, dim.reduc choice, initial.resolution, random.seed)

## Outputs

- Seurat object with broad Louvain clusters and SCISSORS-derived subcluster annotations
- Per-cell silhouette score dataframe from ComputeSilhouetteScores(avg = FALSE)
- DimPlot/UMAP or t-SNE visualizations of broad and subcluster structure
- Subcluster marker-gene tables (e.g., HLA-DR levels distinguishing intermediate monocytes)

## How to apply

Load the count matrix into a Seurat object (e.g., PBMC3k via SeuratData::LoadData('pbmc3k'), filtering cells with nFeature_RNA < 2500) and run SCISSORS::PrepareData with concrete parameters such as n.HVG = 4000, n.PC = 15, initial.resolution = 0.4, and random.seed = 629 to obtain initial broad clusters (6 unique clusters on PBMC3k). Estimate per-cell silhouette scores with ComputeSilhouetteScores(avg = FALSE) and inspect the per-cluster distributions; clusters with visibly lower median scores (e.g., clusters 0, 1, & 3 in the PBMC3k README example) are targets for subgroup analysis. Run the multi-step semi-supervised reclustering optimization on those target clusters, then annotate subclusters using cluster-specific marker genes (e.g., elevated HLA-DR identifying intermediate monocytes in monocyte reclustering). Finally, compare performance against alternatives (Seurat, SAFE-clustering ensemble, GiniClust3) by computing silhouette scores — CosineDist on PCA embeddings for non-SCISSORS methods — and testing median differences with paired wilcox.test comparisons.

## Related tools

- **SCISSORS** (Core package: PrepareData preprocessing, silhouette-score-based heterogeneity estimation, semi-supervised iterative reclustering, and annotation of rare subclusters) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Underlying single-cell infrastructure SCISSORS wraps; supplies NormalizeData, FindVariableFeatures (vst, 2000), ScaleData, RunPCA, FindNeighbors, DimPlot)
- **SeuratData** (Provides the pbmc3k example dataset loaded via LoadData)
- **cluster** (Supplies clustering/silhouette tooling used for silhouette scoring comparisons)
- **ggplot2** (Renders cluster visualization and silhouette-score distribution plots)
- **dplyr** (Tidy data manipulation within the R workflow)

## Examples

```
seu_pbmc <- SeuratData::LoadData("pbmc3k") %>% PrepareData(n.HVG = 4000, n.PC = 15, which.dim.reduc = "umap", initial.resolution = 0.4, random.seed = 629); sil_df <- ComputeSilhouetteScores(seu_pbmc, avg = FALSE)
```

## Evaluation signals

- PrepareData reports a fixed number of unique broad clusters (README: 'Found 6 unique clusters' with initial.resolution = 0.4, seed 629 on PBMC3k) — rerun reproducibility depends on fixing random.seed.
- Silhouette-score distributions per broad cluster should show clearly lower median scores for heterogeneous target clusters (0, 1, & 3 in the README example) before reclustering.
- Reclustering a known heterogeneous population (e.g., CD14+ monocytes) should surface a subcluster with significantly higher HLA-DR, annotatable as intermediate monocytes, not found by standard clustering.
- In method comparison, SCISSORS clusterings should achieve the highest median silhouette score, with paired wilcox.test significance versus Seurat, SAFE, and GiniClust3 shown via ggsignif bars.
- SCISSORS results should integrate into existing Seurat pipelines without breaking DimPlot/visualization objects.

## Limitations

- Even small differences in analysis and parameter choice can greatly alter clustering results, so seeds (random.seed = 629) and resolutions (initial.resolution = 0.4) must be recorded for reproducibility.
- Rare cells of extremely low abundance have subtle contributions toward global clustering metrics, so whole-dataset clustering alone is problematic — that motivates, but also bounds, the silhouette-guided approach.
- The workflow requires manual inspection of silhouette distributions to pick reclustering targets; it is semi-supervised, not fully automatic.

## Evidence

- [abstract] silhouette-guided semi-supervised reclustering: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [abstract] Seurat integration: "SCISSORS is wrapped around the popular Seurat R package and can be easily integrated into existing Seurat pipelines"
- [readme] parameter specificity: "PrepareData(n.HVG = 4000, 
                        n.PC = 15, 
                        which.dim.reduc = "umap", 
                        initial.resolution = 0.4,"
- [readme] target identification: [evidence span withheld — anchor status: external_doc]
- [intro] method rationale: "This package enables researchers to efficiently, accurately, and reproducibly uncover rare celltypes and other small subgroups using an iterative reclustering optimization routine"
- [methods] silhouette computation in methods: "sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)"
