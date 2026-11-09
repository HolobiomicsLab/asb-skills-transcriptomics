---
name: cluster-specificity-curve-analysis
description: Use when a preprocessed Seurat scRNA-seq object must be clustered at
  an unknown resolution and you need an objective, marker-specificity-based criterion
  (rather than eyeballing) to pick the configuration that yields the most specific
  cluster markers before cell-type annotation.
when_to_use_negative:
- The clustering configuration is already fixed and validated (e.g., a published curated
  annotation you intend to use as-is) — re-optimization is unnecessary
- The data is a single-replicate dataset where cluster stability/reproducibility is
  questionable and spurious clusters from artifact genes cannot be distinguished from
  real biology
- The goal is cell-type label assignment rather than resolution selection — use the
  Miko scoring pipeline or sc-type instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scMiko
  role: Provides multiCluster() to sweep Louvain resolutions and multiSpecificity()
    to compute CDI-based marker specificity curves and AUC indices per resolution
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework underlying the Seurat object input and the Louvain clustering (FindClusters-style,
    algorithm=1) executed via multiCluster()
  repo: https://satijalab.org/seurat/
- name: scPipeline
  role: Modular Rmarkdown workflow that packages the resolution-optimization step
    into reproducible dashboard reports
  repo: https://github.com/NMikolajewicz/scPipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:fb0eb8acd95dac519564274544be798b3ed6bc51a1528f9fde61111894e4ff6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  - scPipeline
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cluster-specificity-curve-analysis

## Summary

Selects the optimal Louvain clustering resolution by sweeping resolutions and, for each configuration, measuring marker specificity via the co-dependency index (CDI); the resolution maximizing the AUC of the specificity curve (termed 'Speak') is chosen as the starting point for annotation.

## When to use

Use when a preprocessed Seurat scRNA-seq object must be clustered at an unknown resolution and you need an objective, marker-specificity-based criterion (rather than eyeballing) to pick the configuration that yields the most specific cluster markers before cell-type annotation.

## When NOT to use

- The clustering configuration is already fixed and validated (e.g., a published curated annotation you intend to use as-is) — re-optimization is unnecessary
- The data is a single-replicate dataset where cluster stability/reproducibility is questionable and spurious clusters from artifact genes cannot be distinguished from real biology
- The goal is cell-type label assignment rather than resolution selection — use the Miko scoring pipeline or sc-type instead

## Inputs

- Preprocessed Seurat object (.rds, e.g., so_tyser2021_220621.rds) with PCA/neighbors computed
- Candidate resolution grid (vector of Louvain resolutions, e.g., 0.5-3 or c(0.05,...,2))
- Optional author-curated cluster labels (e.g., 'sub_cluster' metadata column) for comparison

## Outputs

- specificity_summary.csv (resolution, AUC/specificity index, cluster-level specificity)
- Resolution plot (AUC vs resolution) and cluster specificity plots
- Chosen optimal ('Speak') resolution and secondary elbow resolutions
- ARI value comparing the Speak configuration to ground-truth/curated clusters

## How to apply

Load the preprocessed Seurat object (e.g., so_tyser2021_220621.rds, 1195 cells) and cluster across a candidate resolution grid with multiCluster() (algorithm=1, pca_var=0.9, group_singletons=FALSE; e.g., resolutions 0.5-3 or a fine grid c(0.05, 0.1, 0.2, 0.3, 0.4, 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2)). For each configuration, run multiSpecificity() with deg_prefilter=TRUE, cdi_bins=seq(0, 1, by=0.01), and min.pct=0.1 to find the top CDI marker per cluster and build specificity curves. Compute the resolution-specific specificity index as the AUC of the specificity curve, take the maximal-AUC resolution ('Speak') as optimal, and note secondary 'elbow' resolutions corresponding to multi-level biologically relevant configurations. Evaluate any curated annotation (e.g., the author 'sub_cluster' labels) with the same metric for comparison, and compute ARI of the Speak configuration against ground-truth clusters. On Tyser 2021 human gastrulation data, this yielded a specificity index of 0.69 for Speak vs 0.56 for author-curated clusters (vignette: 0.614 at resolution 0.5 with an elbow at 1.0).

## Related tools

- **scMiko** (Provides multiCluster() to sweep Louvain resolutions and multiSpecificity() to compute CDI-based marker specificity curves and AUC indices per resolution) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework underlying the Seurat object input and the Louvain clustering (FindClusters-style, algorithm=1) executed via multiCluster()) — https://satijalab.org/seurat/
- **scPipeline** (Modular Rmarkdown workflow that packages the resolution-optimization step into reproducible dashboard reports) — https://github.com/NMikolajewicz/scPipeline

## Examples

```
so.query <- multiCluster(object = so.query, resolutions = c(0.05,0.1,0.2,0.3,0.4,0.5,0.75,1,1.25,1.5,1.75,2), algorithm = 1, pca_var = 0.9, group_singletons = FALSE); so.query <- multiSpecificity(object = so.query, deg_prefilter = TRUE, cdi_bins = seq(0, 1, by = 0.01), min.pct = 0.1)
```

## Evaluation signals

- The maximal-AUC resolution's specificity index exceeds that of the author-curated configuration (Tyser 2021 benchmark: 0.69 for Speak vs 0.56 for author-curated clusters)
- Vignette cross-check: specificity index ≈ 0.614 at resolution 0.5 with a secondary elbow at resolution 1.0
- specificity_summary.csv contains one row per candidate resolution with resolution, AUC, and cluster-level specificity columns
- ARI of the Speak configuration against ground-truth clusters is computed and reported
- Each cluster in every swept configuration has a top CDI marker (min.pct=0.1, cdi_bins over [0,1] at 0.01 steps), so specificity curves are non-degenerate

## Limitations

- The method's reliability depends on the stability and reproducibility of clusters, especially in single-replicate datasets
- In the absence of experimental replicates it is difficult to determine whether clusters represent technical artifacts or real biology
- Artifact genes highly expressed in a small subset of cells from a single replicate can produce distinct spurious clusters that distort the resolution sweep
- CDI-based DE does not account for variation in biological replicates

## Evidence

- [abstract] cluster resolution optimization using a marker-specificity criterion: "(ii) cluster resolution optimization using a marker-specificity criterion"
- [intro] propose a specificity-based resolution criterion to identify optimal cluster configurations: "propose a specificity-based resolution criterion to identify optimal cluster configurations"
- [discussion] Reliable annotation begins with identifying the optimal clustering configuration.: "Reliable annotation begins with identifying the optimal clustering configuration."
- [discussion] Specificity-based resolution selection limited by stability and reproducibility of clusters, especially in single-replicate datasets: "We note that a limitation of our method relates to the stability and reproducibility of clusters, especially in single-replicate data sets."
