---
name: clustering-benchmark-against-author-annotations
description: Use when evaluating a proposed clustering/annotation strategy against
  an author-curated clustering on a published scRNA-seq dataset (e.g. reproducing
  a paper's resolution-selection criterion on a reference atlas with known sub_cluster
  labels).
when_to_use_negative:
- The input dataset has no author-curated cluster annotation to benchmark against
  — use the specificity criterion alone rather than a comparison workflow
- The input is raw counts requiring QC/normalization first — scPipeline preprocessing
  modules must run before multiCluster()
- Cluster stability across experimental replicates is the primary concern — the article
  notes clusters, especially in single-replicate datasets, may reflect technical artifacts
  rather than real biology
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMiko
  role: R package providing multiCluster() for multi-resolution Louvain clustering
    and multiSpecificity() for CDI-based marker specificity scoring
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework on which scMiko is built; provides the Seurat object (so.query)
    holding counts and cluster metadata
  repo: https://satijalab.org/seurat/
- name: scPipeline
  role: Rmarkdown workflow collection that wraps the clustering and specificity modules
    into reproducible dashboard reports
  repo: https://github.com/NMikolajewicz/scPipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:aa5b606db418f1fa43284c89e6f9826d95fdcef354f611c1723fc06a0fdc1d64
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# clustering-benchmark-against-author-annotations

## Summary

Reproduce an unsupervised clustering run over a grid of Louvain resolutions on a published scRNA-seq dataset, score each configuration with a CDI-based marker-specificity index, and compare the resulting optimal clusters against the author-curated annotation (specificity index and ARI). Used to benchmark scMiko/scPipeline's specificity-based resolution criterion on ground-truth annotated data.

## When to use

Use when evaluating a proposed clustering/annotation strategy against an author-curated clustering on a published scRNA-seq dataset (e.g. reproducing a paper's resolution-selection criterion on a reference atlas with known sub_cluster labels).

## When NOT to use

- The input dataset has no author-curated cluster annotation to benchmark against — use the specificity criterion alone rather than a comparison workflow
- The input is raw counts requiring QC/normalization first — scPipeline preprocessing modules must run before multiCluster()
- Cluster stability across experimental replicates is the primary concern — the article notes clusters, especially in single-replicate datasets, may reflect technical artifacts rather than real biology

## Inputs

- Preprocessed Seurat object (.rds) with author-curated cluster annotations, e.g. so_tyser2021_220621.rds (1195 cells)
- Author ground-truth cluster labels (e.g. 'sub_cluster' metadata column)
- Range of candidate Louvain resolutions to scan

## Outputs

- specificity_summary.csv (resolution, AUC/specificity index, cluster-level specificity)
- Resolution-selection plot and cluster specificity plot
- Top CDI markers per cluster at each resolution
- ARI between the optimal-resolution configuration and the author-curated clusters
- Marker dot plots comparing Speak vs curated clusters

## How to apply

Load the preprocessed Seurat object (e.g. so_tyser2021_220621.rds, 1195 cells) and visualize the author-annotated populations ('sub_cluster'). Cluster cells across a grid of candidate Louvain resolutions using scMiko's multiCluster() (algorithm=1, pca_var=0.9, group_singletons=FALSE; e.g. resolutions 0.5–3 for Tyser 2021 or c(0.05, 0.1, 0.2, 0.3, 0.4, 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2) for the vignette grid). For each configuration, run multiSpecificity() (deg_prefilter=TRUE, cdi_bins=seq(0, 1, by=0.01), min.pct=0.1) to identify the top CDI marker per cluster and build specificity curves. Compute the resolution-specific specificity index (AUC of the specificity curve) and select the resolution with maximal AUC ('Speak'), noting secondary 'elbow' resolutions for multi-level configurations. Score the author-curated annotation with the same specificity metric and compute the ARI of the Speak configuration against the ground-truth clusters; on Tyser 2021 the expected outcome is specificity 0.69 for Speak vs 0.56 for author-curated clusters (vignette: 0.614 at resolution 0.5 with an elbow at 1.0).

## Related tools

- **scMiko** (R package providing multiCluster() for multi-resolution Louvain clustering and multiSpecificity() for CDI-based marker specificity scoring) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework on which scMiko is built; provides the Seurat object (so.query) holding counts and cluster metadata) — https://satijalab.org/seurat/
- **scPipeline** (Rmarkdown workflow collection that wraps the clustering and specificity modules into reproducible dashboard reports) — https://github.com/NMikolajewicz/scPipeline

## Examples

```
so.query <- readRDS("../data/demo/so_tyser2021_220621.rds"); so.cls <- multiCluster(object = so.query, algorithm = 1, resolutions = seq(0.5, 3, by = 0.5), pca_var = 0.9, group_singletons = FALSE); spec <- multiSpecificity(object = so.cls, deg_prefilter = TRUE, cdi_bins = seq(0, 1, by = 0.01), min.pct = 0.1)
```

## Evaluation signals

- The optimal-resolution (Speak) specificity index must exceed the author-curated configuration's index (Tyser 2021: 0.69 vs 0.56)
- The vignette reference value of 0.614 at resolution 0.5 with an elbow at resolution 1.0 should be reproduced on the same data
- The specificity_summary.csv must contain one row per candidate resolution with a monotonic, comparable AUC definition across configurations
- ARI between the Speak configuration and the author sub_cluster labels must be reported to quantify concordance independent of the specificity metric
- Visual inspection should confirm Speak clusters resolve biologically coherent populations with more sensitive top markers than the curated clusters

## Limitations

- The specificity-based resolution selection is limited by the stability and reproducibility of clusters, especially in single-replicate datasets
- In the absence of experimental replicates it is difficult to determine whether clusters represent technical artifacts or real biology; artifact genes expressed in a small subset of cells from a single replicate can produce distinct spurious clusters
- The CDI DE method does not account for variation in biological replicates

## Evidence

- [abstract] cluster resolution optimization using a marker-specificity criterion: "cluster resolution optimization using a marker-specificity criterion"
- [intro] propose specificity criterion: "propose a specificity-based resolution criterion to identify optimal cluster configurations"
- [discussion] optimal clustering as annotation starting point: "Reliable annotation begins with identifying the optimal clustering configuration."
- [methods] demo Seurat object load: "so.query <- readRDS("../data/demo/so_tyser2021_220621.rds")"
- [methods] scMiko installation: "devtools::install_github(repo = "NMikolajewicz/scMiko")"
- [methods] Tyser 2021 dataset annotated: "we will annotate the Human Gastrulation dataset reported by Tyser 2021"
