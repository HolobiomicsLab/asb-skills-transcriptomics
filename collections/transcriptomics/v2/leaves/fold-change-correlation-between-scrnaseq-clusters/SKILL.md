---
name: fold-change-correlation-between-scrnaseq-clusters
description: Use when two or more scRNAseq datasets have been independently clustered
  (e.g., TIL, dLN, and acute viral Arm CD4+ T cells) and you need to identify which
  populations share transcriptomic profiles across datasets.
when_to_use_negative:
- Datasets have not been individually clustered and analyzed first — the method reads
  preclustered datasets and cannot substitute for per-dataset clustering
- Only a single dataset is available and no cross-dataset population comparison is
  needed
- The goal is cell-level integration (e.g., batch-corrected embedding) rather than
  cluster-level transcriptomic profile comparison
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: robustSingleCell
  role: R package providing assess.cluster.similarity, get.robust.cluster.similarity,
    visualize.cluster.cors.heatmaps, and visualize.cluster.similarity.stats for FC-vector
    correlation analysis across clusters
  repo: github.com/asmagen/robustSingleCell
- name: R
  role: Execution environment for the robustSingleCell pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:f4f9cbb54c50f7b1afbfd6a59c0e6d85177d1ada93d7d41b374df5943ea3bce9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - robustSingleCell
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fold-change-correlation-between-scRNAseq-clusters

## Summary

A cross-dataset cluster similarity method in robustSingleCell that compares independently clustered scRNAseq datasets by computing fold-change (FC) vectors for each cluster and quantifying similarity between cluster pairs via Pearson correlation, retaining only statistically robust matches.

## When to use

Use when two or more scRNAseq datasets have been independently clustered (e.g., TIL, dLN, and acute viral Arm CD4+ T cells) and you need to identify which populations share transcriptomic profiles across datasets.

## When NOT to use

- Datasets have not been individually clustered and analyzed first — the method reads preclustered datasets and cannot substitute for per-dataset clustering
- Only a single dataset is available and no cross-dataset population comparison is needed
- The goal is cell-level integration (e.g., batch-corrected embedding) rather than cluster-level transcriptomic profile comparison

## Inputs

- Two or more independently clustered scRNAseq datasets with precomputed robustSingleCell analyses (e.g., LCMV1 and LCMV2 replicate folders with matrix.mtx, gene.tsv, barcode.tsv)
- Pooled project environment initialized with initialize.project and populated via read.preclustered.datasets
- Cluster fold-change (FC) vectors per dataset, computed by assess.cluster.similarity

## Outputs

- cluster.similarity object containing $similarity and $map
- filtered.similarity: robust cluster-pair matches passing min.sd = qnorm(.9) and max.q.val = 0.01
- robust.clusters list (union of matched cluster names across datasets)
- Correlation heatmaps of matched clusters' FC vectors across replicates (Figures 9–10 style) and cross-dataset similarity statistics

## How to apply

After clustering each dataset separately (to account for dataset-specific technical and biological differences), initialize a pooled project with initialize.project and load the precomputed analyses via read.preclustered.datasets. Compute fold-change vectors for every cluster in each dataset with assess.cluster.similarity, then quantify similarity between all cluster pairs using Pearson correlation of their FC vectors. Filter cluster-pair similarities with get.robust.cluster.similarity using min.sd = qnorm(.9) and max.q.val = 0.01 so only statistically robust, reproducible cross-replicate matches survive. Visualize the matched clusters' FC-vector correlations across replicates as a heatmap with visualize.cluster.cors.heatmaps, and recover the meta-cluster structure linking corresponding populations with visualize.cluster.similarity.stats.

## Related tools

- **robustSingleCell** (R package providing assess.cluster.similarity, get.robust.cluster.similarity, visualize.cluster.cors.heatmaps, and visualize.cluster.similarity.stats for FC-vector correlation analysis across clusters) — github.com/asmagen/robustSingleCell
- **R** (Execution environment for the robustSingleCell pipeline)

## Examples

```
cluster.similarity <- assess.cluster.similarity(pooled_env); filtered.similarity <- get.robust.cluster.similarity(pooled_env, cluster.similarity$similarity, min.sd = qnorm(.9), max.q.val = 0.01, rerun = F)
```

## Evaluation signals

- Only cluster pairs passing min.sd = qnorm(.9) and max.q.val = 0.01 appear in filtered.similarity as robust matches
- Generated correlation heatmap and similarity-statistics figures reproduce the meta-cluster structure reported in Figures 9 and 10
- robust.clusters equals the sorted union of cluster1 and cluster2 entries in filtered.similarity
- Tumor-specific TIL clusters should show little overlap with dLN and acute virus-responsive (Arm) clusters, consistent with the published finding of largely distinct TIL profiles

## Limitations

- The robustSingleCell package is not currently maintained, per the README
- Requires at least 8G of memory on a slurm HPC workload manager for the tutorial-scale analysis
- Each dataset must be clustered independently first, since per-dataset clustering accounts for dataset-specific technical and biological differences

## Evidence

- [methods] Compute fold-change (FC) vectors for every cluster in each dataset with assess.cluster.similarity, then quantify similarity between cluster pairs using Pearson correlation of FC vectors.: "cluster.similarity <- assess.cluster.similarity(pooled_env)"
- [methods] Robust cluster pairs are retained only after filtering with min.sd = qnorm(.9) and max.q.val = 0.01.: "filtered.similarity <- get.robust.cluster.similarity(
  pooled_env, similarity, min.sd = qnorm(.9), max.q.val = 0.01, rerun = F"
- [readme] We assessed the similarity between pairs of clusters and identify reproducible subpopulations across the two replicates. Figure 9 shows the correlation between clusters’ FC vectors across replicates.: "We assessed the similarity between pairs of clusters and identify
reproducible subpopulations across the two replicates."
- [readme] Datasets must be clustered separately before integration to account for dataset-specific technical and biological differences.: "We cluster each dataset separately to account for dataset-specific
technical and biological differences."
- [abstract] Tumor-specific TIL transcriptomic profiles are largely distinct from dLN and acute virus-responsive CD4+ T cells.: "identify TIL transcriptomic patterns strikingly distinct from acute and chronic anti-viral responses"
