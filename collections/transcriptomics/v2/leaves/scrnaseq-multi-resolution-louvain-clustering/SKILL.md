---
name: scrnaseq-multi-resolution-louvain-clustering
description: 'Use when an scRNA-seq dataset must be clustered and there is no principled
  resolution choice: cluster across a resolution grid, compute marker specificity
  per configuration, and pick the resolution maximizing the specificity index.'
when_to_use_negative:
- The dataset consists of a single experimental replicate with no replicates — cluster
  stability and reproducibility cannot be assessed, so spurious clusters from artifact
  genes may be indistinguishable from real biology
- Input is raw counts lacking preprocessing (normalization/PCA); the criterion operates
  on already-processed Seurat objects
- A downstream annotation method requires matching a pre-specified, externally fixed
  cluster count, overriding specificity-based resolution selection
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMiko
  role: R package providing multiCluster() for grid-resolution Louvain clustering
    and multiSpecificity() for CDI-based marker-specificity scoring of each configuration
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Underlying framework in which the Seurat object is loaded, clustered, and
    annotated
- name: scPipeline
  role: Modular Rmarkdown workflow that packages this resolution-optimization step
    into reproducible dashboard reports
  repo: https://github.com/NMikolajewicz/scPipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:8afbdbe235d06c9cf16ba0e70dbac0cfc9f232a223be80ac305b139353111693
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# scRNAseq-multi-resolution-louvain-clustering

## Summary

Selects the optimal Louvain clustering resolution for a scRNA-seq dataset by clustering across a resolution grid and scoring each configuration with a CDI-based marker-specificity index (AUC of the specificity curve), as implemented in scMiko's multiCluster()/multiSpecificity() and validated on the Tyser 2021 human gastrulation dataset.

## When to use

Use when an scRNA-seq dataset must be clustered and there is no principled resolution choice: cluster across a resolution grid, compute marker specificity per configuration, and pick the resolution maximizing the specificity index.

## When NOT to use

- The dataset consists of a single experimental replicate with no replicates — cluster stability and reproducibility cannot be assessed, so spurious clusters from artifact genes may be indistinguishable from real biology
- Input is raw counts lacking preprocessing (normalization/PCA); the criterion operates on already-processed Seurat objects
- A downstream annotation method requires matching a pre-specified, externally fixed cluster count, overriding specificity-based resolution selection

## Inputs

- Preprocessed Seurat object (e.g., so_tyser2021_220621.rds) with normalized expression, PCA, and author-annotated cluster labels (e.g., 'sub_cluster')
- Resolution grid (e.g., 0.5–3 or c(0.05, 0.1, 0.2, 0.3, 0.4, 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2))
- CDI parameters: cdi_bins=seq(0, 1, by=0.01), min.pct=0.1, deg_prefilter=TRUE

## Outputs

- Optimal ('Speak') resolution and secondary elbow resolutions
- Specificity summary table (resolution, AUC, cluster-level specificity) written to specificity_summary.csv
- Resolution/specificity plots and cluster marker dot plots
- ARI of the Speak configuration against ground-truth clusters

## How to apply

Load the preprocessed Seurat object (e.g., so_tyser2021_220621.rds, 1195 cells) and cluster cells across a grid of Louvain resolutions using multiCluster() with algorithm=1, pca_var=0.9, group_singletons=FALSE — the article's Fig. 3d-g grid spans 0.5–3 and the vignette grid uses c(0.05, 0.1, 0.2, 0.3, 0.4, 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2). For each configuration, run multiSpecificity() with deg_prefilter=TRUE, cdi_bins=seq(0, 1, by=0.01), and min.pct=0.1 to identify the top co-dependency index (CDI) marker per cluster and build specificity curves. Compute the resolution-specific specificity index as the AUC of the specificity curve, select the resolution with maximal AUC ('Speak'), and note secondary 'elbow' resolutions corresponding to multi-level biologically relevant configurations. Evaluate any reference annotation (e.g., the author-curated 'sub_cluster' labels) with the same metric and compute ARI against it; on Tyser 2021 the Speak configuration scored 0.69 vs. 0.56 for curated clusters (vignette: 0.614 at resolution 0.5 with an elbow at 1.0). The rationale is that reliable annotation begins with identifying the optimal clustering configuration, so resolution is chosen by marker specificity rather than arbitrary defaults.

## Related tools

- **scMiko** (R package providing multiCluster() for grid-resolution Louvain clustering and multiSpecificity() for CDI-based marker-specificity scoring of each configuration) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Underlying framework in which the Seurat object is loaded, clustered, and annotated)
- **scPipeline** (Modular Rmarkdown workflow that packages this resolution-optimization step into reproducible dashboard reports) — https://github.com/NMikolajewicz/scPipeline

## Examples

```
devtools::install_github(repo = "NMikolajewicz/scMiko"); so.query <- readRDS("../data/demo/so_tyser2021_220621.rds")
```

## Evaluation signals

- The optimal resolution's specificity index (AUC) must exceed the AUC computed for the author-curated annotation (Tyser 2021: 0.69 vs. 0.56; vignette: 0.614 at resolution 0.5)
- An 'elbow' in the specificity-vs-resolution curve should correspond to a biologically relevant multi-level configuration (vignette elbow at resolution 1.0)
- specificity_summary.csv must contain one row per candidate resolution with resolution, AUC, and cluster-level specificity columns
- ARI computed between the Speak configuration and ground-truth clusters should be reported and non-trivially positive
- Clustering parameters must match the protocol: algorithm=1, pca_var=0.9, group_singletons=FALSE; multiSpecificity with deg_prefilter=TRUE and cdi_bins=seq(0, 1, by=0.01)

## Limitations

- A limitation of the method relates to the stability and reproducibility of clusters, especially in single-replicate data sets
- In the absence of experimental replicates it is difficult to determine whether clusters represent technical artifacts or real biology
- Artifact genes highly expressed in a small subset of cells from a single experimental replicate have been shown to produce distinct spurious clusters
- The CDI DE approach does not account for variation in biological replicates

## Evidence

- [abstract] cluster resolution optimization using a marker-specificity criterion: "(ii) cluster resolution optimization using a marker-specificity criterion"
- [discussion] Reliable annotation begins with identifying the optimal clustering configuration.: "Reliable annotation begins with identifying the optimal clustering configuration."
- [methods] Tyser 2021 Seurat object and CDI parameters: "so.query <- readRDS("../data/demo/so_tyser2021_220621.rds")"
- [methods] The miko scoring pipeline is a 3-step workflow: "The miko scoring pipeline is a 3-step workflow that is implemented in nullScore(), mikoScore(), and sigScore() functions."
- [methods] scMiko installation: "devtools::install_github(repo = "NMikolajewicz/scMiko")"
