---
name: single-cell-knn-graph-clustering
description: Use when quality-filtered 10X scRNA-seq data (matrix.mtx, gene.tsv, barcode.tsv)
  with selected PCs needs high-resolution T cell cluster discovery, and cluster assignments
  must be validated against a shuffled-data null to avoid spurious clusters.
when_to_use_negative:
- The input is raw, unfiltered 10X data that has not passed read.data quality filtering
  and PCA — clustering before confounder correction and PC selection invalidates the
  null comparison
- The goal is cross-replicate cluster comparison rather than within-dataset clustering
  — use assess.cluster.similarity/get.robust.cluster.similarity instead
- A fixed, externally mandated cluster resolution must be used, since this skill's
  resolution is chosen adaptively from the modularity fold-change criterion
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: robustSingleCell
  role: Provides cluster.analysis (Phenograph/KNN-ratio clustering over resolution
    range), shuffled-data null simulations, filter_cluster_data, and cluster_assignments.csv
    export
  repo: https://github.com/asmagen/robustSingleCell
- name: R
  role: Runtime environment in which the robustSingleCell clustering workflow is executed
- name: slurm
  role: Parallel batch scheduler used to run clustering and shuffled-data simulations
    on HPC (e.g. --mem=8G interactive sessions)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:b0782d6e096b4bf3566b0098104554e9437f3166ec72270856389f25ea3515e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/asmagen/robustSingleCell
  tools:
  - R
  - robustSingleCell
  - slurm
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-knn-graph-clustering

## Summary

KNN ratio-based (Phenograph) clustering over a range of resolutions within the robustSingleCell pipeline, with shuffled-data simulations used to select the resolution that controls false discovery of cell clusters in 10X scRNA-seq data.

## When to use

Use when quality-filtered 10X scRNA-seq data (matrix.mtx, gene.tsv, barcode.tsv) with selected PCs needs high-resolution T cell cluster discovery, and cluster assignments must be validated against a shuffled-data null to avoid spurious clusters.

## When NOT to use

- The input is raw, unfiltered 10X data that has not passed read.data quality filtering and PCA — clustering before confounder correction and PC selection invalidates the null comparison
- The goal is cross-replicate cluster comparison rather than within-dataset clustering — use assess.cluster.similarity/get.robust.cluster.similarity instead
- A fixed, externally mandated cluster resolution must be used, since this skill's resolution is chosen adaptively from the modularity fold-change criterion

## Inputs

- Preprocessed robustSingleCell environment for LCMV1 or LCMV2 (10X matrix.mtx/gene.tsv/barcode.tsv, quality-filtered)
- Highly variable genes from get.variable.genes
- Confounder variables (ribosomal.score, mitochondrial.score, cell.cycle.score, Exhaustion controlled.mean.score)
- Selected PCs from the robustSingleCell PCA step

## Outputs

- Per-cell high-resolution cluster assignment table written to cluster_assignments.csv
- Original vs. shuffled modularity values (mdlrty/mean.shfl) across tested clustering resolutions
- Filtered cluster set (small and CD79+ B-cell clusters removed) with overexpressed genes per cluster

## How to apply

Load the preprocessed LCMV1/LCMV2 environments (10X matrices filtered by read.data, variable genes from get.variable.genes, confounder scores from add.confounder.variables) and the PCs from the robustSingleCell PCA step, then run cluster.analysis, which performs Phenograph (KNN ratio) clustering across multiple resolutions on each replicate independently. Repeat the clustering over shuffled data to estimate the null modularity baseline (mean.shfl) for each resolution. Select the smallest tested KNN ratio whose original-to-shuffled modularity fold change (mdlrty/mean.shfl) exceeds 2 — in the tutorial this was 0.05 — and confirm this invariant before accepting assignments. Filter out small clusters, including CD79+ B-cell clusters, so only T cell clusters remain in the high-resolution assignment, then write the per-cell high-resolution cluster table to cluster_assignments.csv.

## Related tools

- **robustSingleCell** (Provides cluster.analysis (Phenograph/KNN-ratio clustering over resolution range), shuffled-data null simulations, filter_cluster_data, and cluster_assignments.csv export) — https://github.com/asmagen/robustSingleCell
- **R** (Runtime environment in which the robustSingleCell clustering workflow is executed)
- **slurm** (Parallel batch scheduler used to run clustering and shuffled-data simulations on HPC (e.g. --mem=8G interactive sessions))

## Examples

```
LCMV1 <- cluster.analysis(LCMV1, local = T)
```

## Evaluation signals

- The chosen clustering resolution yields a modularity fold change of original vs. shuffled analysis greater than 2 (mdlrty/mean.shfl > 2)
- The chosen KNN ratio is the smallest tested value satisfying the fold-change criterion (0.05 in the LCMV tutorial)
- cluster_assignments.csv exists and contains a per-cell high-resolution cluster label for every cell
- Small CD79+ B-cell clusters are absent from the retained high-resolution cluster set
- Clustering was run separately on each dataset (LCMV1/Rep1 and LCMV2/Rep2) to account for dataset-specific differences

## Limitations

- The robustSingleCell package is not currently maintained; bugs must be reported via GitHub issues and collaboration is needed to improve it
- The analysis requires at least 8G of memory and currently supports only slurm for parallel batch jobs
- Resolution selection is interactive — cluster.analysis prompts the user to choose the optimal resolution after displaying the modularity comparison

## Evidence

- [intro] robustSingleCell is a pipeline designed to identify robust cell subpopulations using scRNAseq data: "robustSingleCell is a pipeline designed to identify robust cell subpopulations using scRNAseq data"
- [readme] We then perform clustering analysis for a range of clustering resolutions.: "We then perform clustering analysis for a range of clustering resolutions."
- [readme] The analysis is repeated multiple times over shuffled data to estimate the appropriate clustering resolution and control for false discovery of clusters.: "The analysis is repeated multiple times over shuffled data to estimate the appropriate clustering resolution and control for false discovery of clusters."
- [readme] We choose 0.05 for our KNN ratio, which is the smallest value tested with `mdlrty/mean.shfl` > 2.: "We choose 0.05 for our KNN ratio, which is the smallest value tested with `mdlrty/mean.shfl` > 2."
- [readme] We select the appropriate resolution, typically the one where there is more than two (2) fold change modularity difference relative to the shuffled analysis.: "We select the appropriate resolution, typically the one where there is more than two (2) fold change modularity difference relative to the shuffled analysis."
