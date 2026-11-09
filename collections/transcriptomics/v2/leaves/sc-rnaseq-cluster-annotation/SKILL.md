---
name: sc-rnaseq-cluster-annotation
description: Use when a processed Seurat object carries broad clusters (or SCISSORS-revealed
  subclusters) that need cell-type labels, and you want markers specific to the target
  cluster rather than generic upregulated genes.
when_to_use_negative:
- Clusters already have trusted labels and the goal is only to visualize known canonical
  markers rather than re-derive a specificity-filtered marker set.
- The Seurat object is unprocessed (no NormalizeData / ScaleData / RunPCA) or has
  no cluster annotations yet — preprocessing and reclustering must come first.
- The goal is to discover rare subclusters within a heterogeneous cluster — use the
  SCISSORS silhouette-scoring and semi-supervised reclustering module, not the marker-gene
  module.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SCISSORS
  role: Provides the marker gene identification module of high specificity, layered
    on the Seurat workflow; source at github.com/jr-leary7/SCISSORS
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Hosts the single-cell object and executes the candidate-list step via FindAllMarkers
    with test.use = 'wilcox' and only.pos = TRUE; SCISSORS is wrapped around Seurat
    and integrates into existing Seurat pipelines
- name: dplyr
  role: Tidy-data manipulation used to build and filter the candidate gene table (e.g.
    percentile comparisons against non-target clusters)
- name: magrittr
  role: Piping used to chain the preprocessing-to-annotation workflow steps
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:d2cb5ff2023edf5d89eee1c4927407efdecd6b838be8f5ca13a89642004886e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SCISSORS
  - Seurat
  - dplyr
  - magrittr
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# sc-rnaseq-cluster-annotation

## Summary

Annotate clusters or subclusters in a Seurat scRNA-seq analysis by identifying marker genes of high specificity to the cell type: build a candidate list with Wilcoxon rank-sum testing against background clusters, then filter out any gene highly expressed in non-target clusters.

## When to use

Use when a processed Seurat object carries broad clusters (or SCISSORS-revealed subclusters) that need cell-type labels, and you want markers specific to the target cluster rather than generic upregulated genes.

## When NOT to use

- Clusters already have trusted labels and the goal is only to visualize known canonical markers rather than re-derive a specificity-filtered marker set.
- The Seurat object is unprocessed (no NormalizeData / ScaleData / RunPCA) or has no cluster annotations yet — preprocessing and reclustering must come first.
- The goal is to discover rare subclusters within a heterogeneous cluster — use the SCISSORS silhouette-scoring and semi-supervised reclustering module, not the marker-gene module.

## Inputs

- Processed scRNA-seq Seurat object (e.g. PBMC3k loaded via SeuratData and prepared with NormalizeData, FindVariableFeatures with vst selection of 2000 features, ScaleData, RunPCA)
- Cluster or subcluster annotations on the Seurat object (broad Louvain clusters or SCISSORS semi-supervised reclustering output)

## Outputs

- Candidate marker gene list per cluster/subcluster (positive markers only, Wilcoxon rank-sum)
- Specificity-filtered marker gene table written to markers_filtered.csv
- Cell-type labels assigned to each cluster/subcluster from the filtered markers

## How to apply

For each target cluster or subcluster, run Wilcoxon rank-sum differential expression against background clusters — the rest of the clusters for broad clustering, or closely related sibling subclusters for subclusters — using Seurat's FindAllMarkers with test.use = 'wilcox' and only.pos = TRUE to build the candidate list of positive markers. For every candidate gene, quantify its expression percentile across the non-target clusters. Remove any gene that falls within the top 10% expressed percentile (user-adjustable) of any non-target cluster, yielding the specificity-filtered marker set, and write the result to markers_filtered.csv. Assign the cell-type label from the filtered markers, consistent with the common practice of determining cell type labels through examination of cluster-specific genes. The comparison-to-background plus percentile-exclusion design is what makes the reported markers specific to the target cell type rather than merely enriched.

## Related tools

- **SCISSORS** (Provides the marker gene identification module of high specificity, layered on the Seurat workflow; source at github.com/jr-leary7/SCISSORS) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Hosts the single-cell object and executes the candidate-list step via FindAllMarkers with test.use = 'wilcox' and only.pos = TRUE; SCISSORS is wrapped around Seurat and integrates into existing Seurat pipelines)
- **dplyr** (Tidy-data manipulation used to build and filter the candidate gene table (e.g. percentile comparisons against non-target clusters))
- **magrittr** (Piping used to chain the preprocessing-to-annotation workflow steps)

## Examples

```
markers <- FindAllMarkers(seu_pbmc, test.use = "wilcox", only.pos = TRUE)
```

## Evaluation signals

- Every gene retained in markers_filtered.csv ranks below the top 10% (user-adjustable) expression percentile of every non-target cluster — the filtered set must contain no gene within the top expressed percentile of any non-target cluster.
- The candidate list contains only positive markers (FindAllMarkers run with test.use = 'wilcox' and only.pos = TRUE); no negative-fold-change genes appear.
- markers_filtered.csv exists and carries one row per surviving candidate gene with its expression percentiles across non-target clusters.
- The filtered markers are sufficient to assign distinct labels to each cluster/subcluster, i.e. markers of high specificity to the target cell type as reported in the paper.
- Preprocessing invariants hold before annotation: vst variable-feature selection with nfeatures = 2000 and PCA on the scaled data (the article's PBMC3k example).

## Limitations

- The exclusion threshold is user-adjustable: a lenient top-percentile cutoff can leave genes that are strongly expressed in non-target clusters, weakening label specificity.
- Background definition matters — broad clusters are compared to the rest of the clusters, whereas subclusters must be compared against other closely related subclusters, or shared lineage markers may be mislabeled as specific.
- Marker specificity is conditional on clustering quality; per the paper, even small differences in analysis and parameter choice can greatly alter clustering results and thus which cell types are identified.

## Evidence

- [abstract] marker genes of high specificity: "Additionally, SCISSORS provides a method for the identification of marker genes of high specificity to the cell type"
- [abstract] cell type labels from cluster-specific genes: "the common practice in scRNA-seq is to determine cell type labels through unsupervised clustering and the examination of cluster-specific genes"
- [readme] annotation is an explicit step of the SCISSORS toolset: "Tools are included to preprocess data, identify potential targets for subgroup analysis, optimize reclustering, and annotate results."
