---
name: cdi-differential-expression-marker-identification
description: Use when you have a clustered (or resolution-swept) Seurat scRNA-seq
  object and need per-cluster top markers ranked by the co-dependency index (CDI)
  to score cluster specificity or benchmark cluster configurations.
when_to_use_negative:
- Datasets with multiple biological replicates where DE methods must account for replicate
  variation — the CDI approach does not model this (a stated limitation)
- Input is raw/unprocessed counts without prior normalization, PCA, and clustering
  — the CDI marker call operates per-cluster on preprocessed data
- Artifact genes highly expressed exclusively in a small subset of cells from a single
  replicate are present and not filtered — they can produce spurious clusters and
  misleading markers
tools:
- name: scMiko
  role: Provides multiSpecificity() for CDI-based DE marker identification (deg_prefilter,
    cdi_bins, min.pct parameters) and multiCluster() for generating the cluster configurations
    being scored
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework hosting the preprocessed Seurat object (counts, normalized data,
    PCA) on which CDI marker identification is run
  repo: https://satijalab.org/seurat/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:c6a067b3cf3274378601a84ff1ca7bd834547148a8345bd480eb2f43c8a03fac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cdi-differential-expression-marker-identification

## Summary

Identifies binary differentially-expressed genes (bDEGs) and the top co-dependency index (CDI) marker for each cell cluster in scRNA-seq data using scMiko's CDI-based differential expression, providing the marker specificity curves that drive resolution optimization in scPipeline.

## When to use

Use when you have a clustered (or resolution-swept) Seurat scRNA-seq object and need per-cluster top markers ranked by the co-dependency index (CDI) to score cluster specificity or benchmark cluster configurations.

## When NOT to use

- Datasets with multiple biological replicates where DE methods must account for replicate variation — the CDI approach does not model this (a stated limitation)
- Input is raw/unprocessed counts without prior normalization, PCA, and clustering — the CDI marker call operates per-cluster on preprocessed data
- Artifact genes highly expressed exclusively in a small subset of cells from a single replicate are present and not filtered — they can produce spurious clusters and misleading markers

## Inputs

- Preprocessed Seurat object (.rds, e.g., so_tyser2021_220621.rds, 1195 cells) with normalized/scaled data and PCA
- Cluster configurations from multiCluster() across a resolution grid (e.g., Louvain resolutions 0.5-3 or c(0.05, 0.1, 0.2, 0.3, 0.4, 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2))
- Count/normalized expression matrix (assay 'RNA')

## Outputs

- Top CDI marker (bDEG) per cluster
- Per-cluster specificity curves over cdi_bins = seq(0, 1, by = 0.01)
- Specificity index (AUC of specificity curve) per resolution/config
- specificity_summary.csv (resolution, AUC, cluster-level specificity)

## How to apply

Load the preprocessed Seurat object (e.g., so_tyser2021_220621.rds) and, for each candidate cluster configuration, run scMiko's multiSpecificity() with deg_prefilter=TRUE, cdi_bins=seq(0, 1, by=0.01), and min.pct=0.1 so that lowly expressed genes are prefILTERed before CDI computation. The CDI identifies binary differentially-expressed genes (bDEGs) per cluster and selects the single top CDI marker per cluster, from which specificity curves are built across CDI bins. Integrate each configuration's specificity curve into a resolution-specific specificity index (AUC of the curve) to compare alternative cluster resolutions or curated annotations. Note the method's stated limitation: CDI does not account for variation in biological replicates, so interpret markers from single-replicate datasets cautiously.

## Related tools

- **scMiko** (Provides multiSpecificity() for CDI-based DE marker identification (deg_prefilter, cdi_bins, min.pct parameters) and multiCluster() for generating the cluster configurations being scored) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework hosting the preprocessed Seurat object (counts, normalized data, PCA) on which CDI marker identification is run) — https://satijalab.org/seurat/

## Examples

```
so.ms <- multiSpecificity(object = so.query, deg_prefilter = TRUE, cdi_bins = seq(0, 1, by = 0.01), min.pct = 0.1)
```

## Evaluation signals

- On Tyser 2021 human gastrulation data, the resolution selected by maximal specificity-index AUC should exceed the author-curated annotation's index (0.69 vs. 0.56 in Fig. 3d-g)
- The vignette benchmark should reproduce a specificity index of 0.614 at resolution 0.5 with an elbow at 1.0
- Every cluster in each configuration yields exactly one top CDI marker, computed over the full cdi_bins = seq(0, 1, by = 0.01) grid
- Genes failing the min.pct = 0.1 prefilter must be excluded before CDI computation
- Specificity summary table must contain resolution, AUC, and cluster-level specificity columns and render resolution/specificity/dot plots

## Limitations

- A known limitation of existing DE methods for scRNA-seq — and the CDI approach is no exception — is the failure to account for variation in biological replicates
- In single-replicate datasets it is difficult to determine whether clusters (and thus their markers) represent technical artifacts or real biology
- Artifact genes highly expressed exclusively in a small subset of cells from a single experimental replicate can produce distinct spurious clusters and confound marker identification
- Marker-based annotations are only as good as the cell-type prescribed to the original dataset; no validation of source catalog curations is provided

## Evidence

- [abstract] co-dependency index (CDI)-based differential expression: "(i) co-dependency index (CDI)-based differential expression"
- [intro] CDI validates bDEGs: "we validate the co-dependency index (CDI) as a differential expression (DE) method that identifies binary differentially-expressed genes (bDEGs)"
- [methods] multiSpecificity parameters and top CDI marker: [evidence span withheld — anchor status: not_found]
- [discussion] CDI replicate limitation: "A known limitation of existing DE methods for scRNA-seq is the failure to account for variation in biological replicates, and the CDI approach is no exception"
