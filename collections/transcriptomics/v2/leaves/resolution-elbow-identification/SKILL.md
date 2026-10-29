---
name: resolution-elbow-identification
description: Use when clustering scRNA-seq data and an objective, marker-based criterion
  is needed to choose among candidate Louvain resolutions, rather than defaulting
  to a resolution like 0.8 or relying solely on author-curated annotations.
when_to_use_negative:
- When clusters derive from single-replicate data where distinguishing technical artifacts
  from real biology is unreliable — the article notes cluster stability and reproducibility
  are a known limitation of the specificity-based criterion
- When the input is not cell-level scRNA-seq expression (e.g., already a marker/feature
  table) — the criterion requires re-clustering and per-cluster CDI marker detection
- When a fixed, externally mandated clustering resolution must be used (e.g., reproducing
  a published clustering verbatim rather than optimizing it)
edam_operation: http://edamontology.org/operation_3643
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMiko
  role: Provides multiCluster() for sweeping Louvain resolutions (algorithm=1, pca_var=0.9,
    group_singletons=FALSE) and multiSpecificity() for CDI-based per-cluster top-marker
    detection and specificity curves; installed via devtools::install_github('NMikolajewicz/scMiko')
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework underlying the clustering and differential expression used to build
    and score each candidate cluster configuration
- name: rbpatt2019/chooseR
  role: Alternative/complementary framework for guided clustering-parameter selection
    via sub-sampling cluster robustness metrics
  repo: https://github.com/rbpatt2019/cluster.stability.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:65c279d7ee8c05ccc0498abf9e97abc1ccac97cee20e52dad0a2691d2f69b9fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - rbpatt2019/chooseR
  - scMiko
  edam_operation: http://edamontology.org/operation_3643
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# resolution-elbow-identification

## Summary

Selects the optimal single-cell clustering resolution by sweeping Louvain resolutions, scoring each configuration with a CDI-based marker-specificity criterion (specificity index = AUC of the specificity curve), and picking the resolution with maximal AUC ('Speak') plus secondary elbow resolutions corresponding to multi-level biologically relevant configurations.

## When to use

Use when clustering scRNA-seq data and an objective, marker-based criterion is needed to choose among candidate Louvain resolutions, rather than defaulting to a resolution like 0.8 or relying solely on author-curated annotations.

## When NOT to use

- When clusters derive from single-replicate data where distinguishing technical artifacts from real biology is unreliable — the article notes cluster stability and reproducibility are a known limitation of the specificity-based criterion
- When the input is not cell-level scRNA-seq expression (e.g., already a marker/feature table) — the criterion requires re-clustering and per-cluster CDI marker detection
- When a fixed, externally mandated clustering resolution must be used (e.g., reproducing a published clustering verbatim rather than optimizing it)

## Inputs

- Preprocessed Seurat object (e.g., so_tyser2021_220621.rds, 1195 cells) with author/curated cluster annotations (e.g., 'sub_cluster')
- Grid of candidate Louvain clustering resolutions
- CDI computation parameters (cdi_bins, min.pct, deg_prefilter)

## Outputs

- specificity_summary.csv (resolution, AUC/specificity index, cluster-level specificity)
- Resolution plot (specificity index vs. resolution with Speak and elbow resolutions marked)
- Cluster specificity plots and marker dot plots for the chosen configuration
- ARI of the Speak configuration against ground-truth clusters

## How to apply

Cluster cells across a grid of candidate Louvain resolutions (e.g., scope resolutions 0.5–3 as in Fig. 3d-g, or vignette grid c(0.05, 0.1, 0.2, 0.3, 0.4, 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2)) using multiCluster() (algorithm=1, pca_var=0.9, group_singletons=FALSE) on a preprocessed Seurat object. For each configuration, run multiSpecificity() with deg_prefilter=TRUE, cdi_bins=seq(0, 1, by=0.01), min.pct=0.1 to identify the top co-dependency index (CDI) marker per cluster and build specificity curves. Compute the resolution-specific specificity index as the AUC of the specificity curve and select the resolution with maximal AUC (Speak), while noting secondary 'elbow' resolutions that reveal multi-level biologically relevant cluster configurations. Evaluate any reference/author-curated annotation with the same metric for comparison, and compute ARI of the chosen configuration against ground-truth clusters. On Tyser 2021 human gastrulation data (so_tyser2021_220621.rds, 1195 cells), Speak exceeded the author-curated sub_cluster configuration (specificity index 0.69 vs. 0.56; vignette reports 0.614 at resolution 0.5 with an elbow at 1.0). Write resolution, AUC, and cluster-level specificity to specificity_summary.csv and render resolution, cluster-specificity, and dot plots.

## Related tools

- **scMiko** (Provides multiCluster() for sweeping Louvain resolutions (algorithm=1, pca_var=0.9, group_singletons=FALSE) and multiSpecificity() for CDI-based per-cluster top-marker detection and specificity curves; installed via devtools::install_github('NMikolajewicz/scMiko')) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework underlying the clustering and differential expression used to build and score each candidate cluster configuration)
- **rbpatt2019/chooseR** (Alternative/complementary framework for guided clustering-parameter selection via sub-sampling cluster robustness metrics) — https://github.com/rbpatt2019/cluster.stability.git

## Examples

```
so.query <- readRDS("../data/demo/so_tyser2021_220621.rds"); so.query <- multiCluster(so.query, resolutions = c(0.05,0.1,0.2,0.3,0.4,0.5,0.75,1,1.25,1.5,1.75,2), algorithm = 1, pca_var = 0.9, group_singletons = FALSE); so.query <- multiSpecificity(so.query, deg_prefilter = TRUE, cdi_bins = seq(0,1,by=0.01), min.pct = 0.1)
```

## Evaluation signals

- The selected resolution's specificity index (AUC) exceeds that of the reference/author-curated configuration (expected 0.69 vs. 0.56 on Tyser 2021; vignette reports 0.614 at resolution 0.5 with an elbow at 1.0)
- Top CDI markers per cluster are more sensitive/visually coherent than markers of the comparison configuration, confirmed by visual inspection
- specificity_summary.csv contains one row per candidate resolution with AUC and cluster-level specificity, and the maximal-AUC row matches the chosen resolution
- ARI between the Speak configuration and ground-truth clusters is computed and reported alongside the specificity comparison
- Specificity curves are built with cdi_bins=seq(0, 1, by=0.01) and min.pct=0.1, so every resolution's AUC is computed over the same bin grid (comparability invariant)

## Limitations

- A known limitation of the specificity-based resolution selection relates to the stability and reproducibility of clusters, especially in single-replicate data sets
- In the absence of experimental replicates it is difficult to determine whether clusters represent technical artifacts or real biology
- Artifact genes highly expressed in a small subset of cells from a single replicate can produce distinct spurious clusters, distorting the resolution sweep
- The CDI DE method does not account for variation in biological replicates

## Evidence

- [intro] propose a specificity-based resolution criterion to identify optimal cluster configurations: "propose a specificity-based resolution criterion to identify optimal cluster configurations"
- [abstract] cluster resolution optimization using a marker-specificity criterion: "(ii) cluster resolution optimization using a marker-specificity criterion"
- [discussion] Reliable annotation begins with identifying the optimal clustering configuration.: "Reliable annotation begins with identifying the optimal clustering configuration."
- [abstract] co-dependency index (CDI)-based differential expression: "(i) co-dependency index (CDI)-based differential expression"
- [discussion] We note that a limitation of our method relates to the stability and reproducibility of clusters, especially in single-replicate data sets.: "We note that a limitation of our method relates to the stability and reproducibility of clusters, especially in single-replicate data sets."
- [methods] we will annotate the Human Gastrulation dataset reported by Tyser 2021: "we will annotate the Human Gastrulation dataset reported by Tyser 2021"
