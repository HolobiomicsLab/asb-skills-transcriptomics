---
name: silhouette-based-cluster-heterogeneity-evaluation
description: Use when a Seurat object has broad Louvain clusters that may mask rare
  cell types, and you need a quantitative criterion (per-cell silhouette score distributions)
  to decide which clusters are heterogeneous enough to recluster.
when_to_use_negative:
- Clusters are already known to be pure/homogeneous (high median silhouette) — reclustering
  would add no value
- Input is not a Seurat object with cluster labels (e.g. a raw feature table or already-fine-grained
  cell type annotation); silhouette evaluation presumes an existing broad clustering
- The goal is determining the optimal number of robust clusters de novo rather than
  detecting rare subclusters within heterogeneous clusters
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: SCISSORS
  role: Provides ComputeSilhouetteScores to estimate per-cell silhouette scores and
    flag heterogeneous clusters for reclustering
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Supplies the clustered Seurat object, DimPlot visualization of broad clusters,
    and the pipeline that SCISSORS is wrapped around
- name: ggplot2
  role: Visualizes silhouette score distributions per broad cluster to identify low-median-score
    clusters
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:c96cb31956b780e5510891bd1ecfa31dcc8d084f42013a643ee79d58f0365904
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# silhouette-based-cluster-heterogeneity-evaluation

## Summary

Compute per-cell silhouette scores on a clustered Seurat object to quantify cluster heterogeneity, then identify broad clusters with low median silhouette scores as candidates for subgroup (rare-cell) reanalysis via SCISSORS.

## When to use

Use when a Seurat object has broad Louvain clusters that may mask rare cell types, and you need a quantitative criterion (per-cell silhouette score distributions) to decide which clusters are heterogeneous enough to recluster.

## When NOT to use

- Clusters are already known to be pure/homogeneous (high median silhouette) — reclustering would add no value
- Input is not a Seurat object with cluster labels (e.g. a raw feature table or already-fine-grained cell type annotation); silhouette evaluation presumes an existing broad clustering
- The goal is determining the optimal number of robust clusters de novo rather than detecting rare subclusters within heterogeneous clusters

## Inputs

- Seurat object after preprocessing and initial clustering (e.g. pbmc3k from SeuratData, or Elyada PDAC Cluster 8 subset)
- Cluster assignment metadata (Louvain clusters from the Seurat pipeline)

## Outputs

- Data frame of per-cell silhouette scores with cluster labels (from ComputeSilhouetteScores, avg = FALSE)
- ggplot2 density/boxplot of silhouette score distributions per broad cluster
- Ranked list of heterogeneous clusters selected as reclustering targets

## How to apply

After preprocessing (NormalizeData, FindVariableFeatures with vst and nfeatures = 2000 or PrepareData with n.HVG = 4000 / n.PC = 15), run initial clustering (e.g. Louvain with initial.resolution = 0.4) and visualize broad clusters on UMAP. Apply SCISSORS::ComputeSilhouetteScores with avg = FALSE to obtain per-cell silhouette scores, then plot score distributions per cluster with ggplot2. Clusters whose median silhouette is 'decently lower' than others (e.g. clusters 0, 1 & 3 in the PBMC3k example) are estimated to be heterogeneous and become targets for the multi-step semi-supervised reclustering that reveals rare cells such as the 23-cell apCAF population in the Elyada PDAC Cluster 8. The rationale is that existing methods focus on optimal robust cluster numbers and are problematic for extremely low-abundance cells whose subtle silhouette contributions are otherwise missed.

## Related tools

- **SCISSORS** (Provides ComputeSilhouetteScores to estimate per-cell silhouette scores and flag heterogeneous clusters for reclustering) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Supplies the clustered Seurat object, DimPlot visualization of broad clusters, and the pipeline that SCISSORS is wrapped around)
- **ggplot2** (Visualizes silhouette score distributions per broad cluster to identify low-median-score clusters)

## Examples

```
sil_score_df <- ComputeSilhouetteScores(seu_pbmc, avg = FALSE); ggplot(sil_score_df, aes(x = silhouette_score, fill = cluster)) + geom_density()
```

## Evaluation signals

- ComputeSilhouetteScores runs with avg = FALSE and returns per-cell (not per-cluster-average) scores
- Silhouette distributions per cluster show visibly lower medians for heterogeneous clusters (e.g. clusters 0, 1 & 3 in the PBMC3k vignette)
- Selected target clusters, when reclustered, yield biologically coherent subclusters (e.g. the 23-cell / 0.092% apCAF subcluster in PDAC Cluster 8)
- Downstream reclustering of selected clusters is reproducible with the documented random.seed (629 in the vignette)
- Silhouette-score plot reproduces the separation between high- and low-median clusters as in the README figure

## Limitations

- Silhouette scores alone do not reveal the rare cells — they only nominate heterogeneous clusters; reclustering must follow
- Small differences in analysis and parameter choice can greatly alter scRNA-seq clustering results, so silhouette-derived target selection is sensitive to preprocessing parameters (n.HVG, n.PC, resolution)
- Extremely low-abundance cells make subtle contributions toward silhouette-based statistics, making threshold-free visual inspection of distributions important

## Evidence

- [abstract] Silhouette scoring identifies heterogeneous clusters: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [abstract] Problem with existing methods for low-abundance cells: "Existing methods largely focus on determining the optimal number of robust clusters, which can be problematic for identifying cells of extremely low abundance due to their subtle contributions"
- [readme] README usage of ComputeSilhouetteScores: [evidence span withheld — anchor status: external_doc]
- [readme] README interpretation of silhouette distributions: [evidence span withheld — anchor status: external_doc]
- [readme] README preprocessing parameters: [evidence span withheld — anchor status: external_doc]
- [methods] Vignette methods preprocessing: "FindVariableFeatures(selection.method = "vst", nfeatures = 2000, verbose = FALSE)"
