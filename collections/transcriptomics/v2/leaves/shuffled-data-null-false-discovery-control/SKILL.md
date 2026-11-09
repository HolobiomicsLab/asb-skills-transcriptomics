---
name: shuffled-data-null-false-discovery-control
description: Use when clustering scRNA-seq data (e.g., LCMV1/LCMV2 10X CD44+ T cell
  matrices with selected PCs) and you must pick a clustering resolution without over-discovering
  spurious clusters.
when_to_use_negative:
- The data have not passed quality filtering, variable-gene selection, and confounder
  correction — the shuffle-based control assumes the read.data/get.variable.genes/add.confounder.variables
  steps are complete
- You only need a single fixed-resolution clustering and have no need to control false
  discovery (e.g., exploratory plots), since running repeated shuffled analyses is
  computationally expensive
- The biological question calls for cross-dataset cluster similarity/integration rather
  than per-replicate clustering resolution selection
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: robustSingleCell
  role: Provides cluster.analysis, which runs Phenograph (KNN ratio) clustering over
    multiple resolutions and repeats it over shuffled data to estimate the null modularity
    baseline
  repo: https://github.com/asmagen/robustSingleCell
- name: R
  role: Language in which the robustSingleCell pipeline and the shuffled-data clustering
    control are executed
- name: slurm
  role: Parallel batch-job manager supported by robustSingleCell for running the repeated
    clustering simulations on HPC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:564a751fc74167fb72c4d8916c7e3e204d4240ed4b94a1257895cb97efd8e7de
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
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# shuffled-data-null-false-discovery-control

## Summary

A false-discovery control for scRNA-seq clustering (robustSingleCell) in which Phenograph clustering is run over a range of KNN-ratio resolutions on both the original data and repeatedly shuffled data, and the resolution is accepted only when original-to-shuffled modularity fold change (mdlrty/mean.shfl) exceeds 2.

## When to use

Use when clustering scRNA-seq data (e.g., LCMV1/LCMV2 10X CD44+ T cell matrices with selected PCs) and you must pick a clustering resolution without over-discovering spurious clusters.

## When NOT to use

- The data have not passed quality filtering, variable-gene selection, and confounder correction — the shuffle-based control assumes the read.data/get.variable.genes/add.confounder.variables steps are complete
- You only need a single fixed-resolution clustering and have no need to control false discovery (e.g., exploratory plots), since running repeated shuffled analyses is computationally expensive
- The biological question calls for cross-dataset cluster similarity/integration rather than per-replicate clustering resolution selection

## Inputs

- Preprocessed 10X scRNA-seq environments (matrix.mtx, gene.tsv, barcode.tsv) for LCMV1 and LCMV2 replicates
- Selected principal components from the robustSingleCell PCA step
- Highly variable gene set and confounder scores (ribosomal, mitochondrial, cell cycle, exhaustion) attached to each environment

## Outputs

- Clustering modularity plot of original vs. shuffled data across resolutions with per-resolution fold-change values (mdlrty/mean.shfl)
- Selected optimal clustering resolution (e.g., KNN ratio 0.05)
- High-resolution cluster assignments per cell (cluster_assignments.csv) after removing small CD79+ B-cell clusters

## How to apply

Load the quality-filtered 10X environments (LCMV1, LCMV2) and the PCs chosen by the robustSingleCell PCA step, then run robustSingleCell cluster.analysis (KNN-ratio/Phenograph clustering) over a range of resolutions on each replicate. Repeat the clustering multiple times over shuffled data to estimate the null modularity baseline for each resolution. Select the optimal resolution as the smallest tested KNN ratio whose original-to-shuffled modularity fold change (mdlrty/mean.shfl) exceeds 2 — in the tutorial this is 0.05. Retain only clusters passing the false-discovery threshold and remove small CD79+ B-cell clusters via filter_cluster_data, then summarize and write the per-cell high-resolution assignment table (cluster_assignments.csv). The rationale is that genuine structure yields modularity far above the shuffled-data null, so fold change > 2 guards against false discovery of clusters.

## Related tools

- **robustSingleCell** (Provides cluster.analysis, which runs Phenograph (KNN ratio) clustering over multiple resolutions and repeats it over shuffled data to estimate the null modularity baseline) — https://github.com/asmagen/robustSingleCell
- **R** (Language in which the robustSingleCell pipeline and the shuffled-data clustering control are executed)
- **slurm** (Parallel batch-job manager supported by robustSingleCell for running the repeated clustering simulations on HPC)

## Examples

```
LCMV1 <- cluster.analysis(LCMV1, local = T)
```

## Evaluation signals

- Chosen resolution satisfies mdlrty/mean.shfl > 2 (original modularity more than twofold above the shuffled-data mean)
- The chosen resolution is the smallest tested KNN ratio meeting the fold-change criterion (0.05 in the LCMV tutorial)
- cluster_assignments.csv contains a per-cell high-resolution cluster assignment and contains no small CD79+ B-cell clusters after filter_cluster_data
- Modularity bar plot (Clustering.modularity.png) shows fold-change labels for each tested resolution, confirming shuffled simulations were run for every resolution
- Clustering was performed independently on each replicate (LCMV1 and LCMV2) to account for dataset-specific technical and biological differences

## Limitations

- The robustSingleCell package is not currently maintained; the authors ask users to contact them to collaborate on improvements
- Running repeated shuffled-data clusterings is computationally heavy; the pipeline requires at least 8G of memory on slurm and benefits from parallel (local = F) execution
- The package currently supports slurm only for parallel batch jobs

## Evidence

- [methods] Shuffled-data clustering repeated to estimate null: "We then perform clustering analysis for a range of clustering resolutions. The analysis is repeated multiple times over shuffled data to estimate the appropriate clustering resolution and control for"
- [methods] Fold-change threshold and chosen resolution: "We choose 0.05 for our KNN ratio, which is the smallest value tested with `mdlrty/mean.shfl` \> 2."
- [readme] Resolution selection rule: "We select the appropriate resolution, typically the one where there is more than two (2) fold change modularity difference relative to the shuffled analysis."
- [readme] Purpose of the pipeline: "robustSingleCell is a pipeline designed to identify robust cell
subpopulations
using scRNAseq data"
- [readme] Slurm and memory requirement: "The analysis requires at least 8G of memory on
*slurm* \[3\] high performance computing workload manager"
- [intro] Package maintenance status: "**This package is not maintained at the moment -- please contact if you're interested in collaborating and improving this project.**"
