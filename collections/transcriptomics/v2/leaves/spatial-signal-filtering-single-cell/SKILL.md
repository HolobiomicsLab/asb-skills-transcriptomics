---
name: spatial-signal-filtering-single-cell
description: Use when spatial transcriptomics slides (count matrix + spatial coordinates
  per cell) contain a local spatial signal that masks global cell subpopulations,
  and the goal is to expose those global migratory or regenerative subpopulations.
when_to_use_negative:
- When the goal is to enhance a temporal signal (e.g., liver circadian) from single-cell
  (non-spatial) data by filtering spatial zonation — a different SiFT configuration
  using prior zonation knowledge rather than per-slide coordinates
- When prior knowledge needed to define the cell-cell similarity kernel (here, spatial
  coordinates) is inaccurate or coarse-grained, since this 'may lead to a loss of
  signal'
- When downstream tasks require properly log-transformed expression, since SiFT-modified
  data 'is no longer properly log-transformed'
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SiFT (sift-sc)
  role: 'Core filtering engine: builds a cell-cell similarity kernel from spatial
    coordinates and removes the local spatial signal from the expression matrix'
  repo: https://github.com/nitzanlab/sift-sc
- name: SiFT
  role: Framework (sift.SiFT API, sift.kernels.KnnKernel) used per slide to filter
    the local spatial neighborhood signal
  repo: https://github.com/nitzanlab/sift-sc
- name: pip
  role: Installation channel for the sift-sc package ('pip install sift-sc')
- name: Python
  role: Language in which the filtering, concatenation, clustering, and pseudotime
    workflow is implemented
- name: scVI
  role: Latent-space integration baseline for comparison; contrasted with SiFT's gene-expression-level
    correction
  repo: https://github.com/scverse/scvi-tools
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:26668417cfe4168028c58c5be8d59e5c25b2688bca031143f20ada63897d3320
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - pip
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spatial-signal-filtering-single-cell

## Summary

Apply SiFT with a cell-cell similarity kernel built from spatial coordinates to filter local neighborhood signals from spatial transcriptomics data, exposing global subpopulations (e.g., ANXA2+ migratory hepatocytes in APAP-induced liver regeneration) that are obscured by local spatial zonation-like structure.

## When to use

Use when spatial transcriptomics slides (count matrix + spatial coordinates per cell) contain a local spatial signal that masks global cell subpopulations, and the goal is to expose those global migratory or regenerative subpopulations.

## When NOT to use

- When the goal is to enhance a temporal signal (e.g., liver circadian) from single-cell (non-spatial) data by filtering spatial zonation — a different SiFT configuration using prior zonation knowledge rather than per-slide coordinates
- When prior knowledge needed to define the cell-cell similarity kernel (here, spatial coordinates) is inaccurate or coarse-grained, since this 'may lead to a loss of signal'
- When downstream tasks require properly log-transformed expression, since SiFT-modified data 'is no longer properly log-transformed'

## Inputs

- Spatial transcriptomics slides from APAP-induced liver injury (GEO accession GSE223561) as AnnData objects with count matrices and per-cell spatial coordinates
- SiFT python package installed via 'pip install sift-sc'
- Optional: scFates and scanpy for pseudotime and marker-gene analysis

## Outputs

- SiFT-filtered expression matrix per slide (local spatial signal removed)
- Joint Leiden clustering / UMAP across concatenated filtered slides highlighting the Anxa2+ cluster
- Cluster marker gene table (markers.csv) with Anxa2 among top markers of cluster 1
- Per-cluster sample composition bar plot showing homogeneous proportions across slides
- Pseudotime gene grouping (pseudotime_groups.csv) from scFates; Fig. 5 panels saved as fig5_umap_clusters.png

## How to apply

Load each APAP-injury spatial transcriptomics slide (GSE223561) as an AnnData object containing the count matrix and spatial coordinates. For each slide, run sift.SiFT with a distance/KNN-type kernel built from the spatial coordinates (e.g., sift.kernels.KnnKernel) so the kernel captures similarity with respect to the local neighborhood; SiFT then projects the expression matrix onto this spatial signal and deducts the projection, yielding a filtered count matrix per slide. Concatenate the SiFT-filtered slides, embed them (PCA plus a neighbor graph), and perform joint Leiden clustering across samples so cluster identities are not slide-specific. Call marker genes per cluster with scanpy rank_genes_groups and verify that the cluster recovered (cluster 1) is the ANXA2+ migratory hepatocyte subpopulation with Anxa2 among its top markers; optionally run scFates pseudotime on the filtered data to partition genes into early/late groups. The rationale, per the article, is that 'global subpopulations involved in the regeneration process are exposed by filtering the local neighborhood of cells using the corresponding spatial coordinates'.

## Related tools

- **SiFT (sift-sc)** (Core filtering engine: builds a cell-cell similarity kernel from spatial coordinates and removes the local spatial signal from the expression matrix) — https://github.com/nitzanlab/sift-sc
- **SiFT** (Framework (sift.SiFT API, sift.kernels.KnnKernel) used per slide to filter the local spatial neighborhood signal) — https://github.com/nitzanlab/sift-sc
- **pip** (Installation channel for the sift-sc package ('pip install sift-sc'))
- **Python** (Language in which the filtering, concatenation, clustering, and pseudotime workflow is implemented)
- **scVI** (Latent-space integration baseline for comparison; contrasted with SiFT's gene-expression-level correction) — https://github.com/scverse/scvi-tools

## Examples

```
pip install sift-sc; import sift; sft = sift.SiFT(data=adata, kernel=sift.kernels.KnnKernel(n_neighbors=30, key_added='spatial')); sft.fit()
```

## Evaluation signals

- Joint Leiden clustering of SiFT-filtered slides yields a cluster 1 in which Anxa2 ranks among the top marker genes (scanpy rank_genes_groups)
- Every cluster contains comparable proportions of cells from each APAP-injury slide (homogeneous sample composition), matching the qualitative claims reported for Fig. 5
- Recovered cluster 1 shows a migratory GO-term phenotype consistent with the ANXA2+ migratory hepatocyte subpopulation
- scFates pseudotime on the filtered data partitions genes into coherent early and late pseudotime groups
- Output files fig5_umap_clusters.png, markers.csv, and pseudotime_groups.csv are produced and reproduce the Fig. 5 panels

## Limitations

- SiFT 'strongly relies on the prior knowledge used to define the cell–cell similarity kernel'; an inaccurate or coarse-grained mapping may lead to a loss of signal
- No ground truth reference exists to evaluate computational filtering results; evaluation must rely on application-specific integration metrics
- SiFT modifies gene expression so data is no longer properly log-transformed, a property expected in certain downstream analysis tasks
- Statistical implications of applying commonly used analysis tools to corrected gene expression data are unexplored

## Evidence

- [intro] global subpopulations exposed by filtering local neighborhoods: "global subpopulations involved in the regeneration process are exposed by filtering the local neighborhood of cells using the corresponding spatial coordinates"
- [discussion] spatial coordinates used to remove local signals in liver regeneration: "we applied SiFT to spatially resolved dataset of liver regeneration, where we used the spatial coordinates to remove local signals"
- [intro] kernel projection and subtraction workflow: "we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication"
- [methods] install via pip: "Through **pip**:: pip install sift-sc"
- [discussion] SiFT relies on prior knowledge for the kernel: "it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel."
