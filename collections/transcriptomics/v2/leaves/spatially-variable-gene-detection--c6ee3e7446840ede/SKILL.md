---
name: spatially-variable-gene-detection
description: Use when you have a spatial transcriptomics count matrix with 2D (or
  2-3D) spatial coordinates and need per-gene SVG rankings and p-values without pre-clustering
  cells or imposing a detection threshold.
when_to_use_negative:
- The goal is differential expression between labeled conditions (e.g. wild-type vs
  knockout) — the article states comparisons between multiple conditions cannot be
  easily conducted.
- The results must feed directly into a cluster-oriented scRNA-seq DE analysis — the
  article notes singleCellHaystack results are not straightforward to incorporate
  into cluster-oriented analyses.
- Runtime on very large spatial data is the dominant constraint — SPARK-X is generally
  the fastest method, with singleCellHaystack having relatively longer runtimes.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack (R)
  role: Primary SVG/differentially-active-feature prediction method run on 2D spatial
    coordinates; produces per-gene ranks and p-values
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack (Python)
  role: Python implementation (PyPI 0.0.5) of singleCellHaystack >= 1.0.0 for scalable
    application to spatial datasets via scanpy AnnData
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: SPARK-X
  role: Competing SVG method used for cross-method rank/p-value comparison and runtime
    benchmarking
- name: SeuratWrappers
  role: Provides Seurat access to community methods (e.g. singleCellHaystack via wrappers)
    for spatial feature detection
  repo: https://github.com/satijalab/seurat-wrappers
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:e52966ffd143ddf145c601cc9e7a029df6eccf39185cb3cf497a457fb8d111ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - Python
  - R
  - SPARK-X
  - SeuratWrappers
  - scanpy
  - singleCellHaystack (Python)
  - singleCellHaystack (R)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# spatially-variable-gene-detection

## Summary

Rank genes by spatially non-random activity in spatial transcriptomics data (MERFISH, Visium, Slide-seqV2, HDST) using singleCellHaystack, which applies Kullback-Leibler divergence on 2D spatial coordinates without clustering, and benchmark/compare rankings against SPARK-X.

## When to use

Use when you have a spatial transcriptomics count matrix with 2D (or 2-3D) spatial coordinates and need per-gene SVG rankings and p-values without pre-clustering cells or imposing a detection threshold.

## When NOT to use

- The goal is differential expression between labeled conditions (e.g. wild-type vs knockout) — the article states comparisons between multiple conditions cannot be easily conducted.
- The results must feed directly into a cluster-oriented scRNA-seq DE analysis — the article notes singleCellHaystack results are not straightforward to incorporate into cluster-oriented analyses.
- Runtime on very large spatial data is the dominant constraint — SPARK-X is generally the fastest method, with singleCellHaystack having relatively longer runtimes.

## Inputs

- Gene-by-cell (or gene-by-bead) spatial transcriptomics count matrix as .h5ad / AnnData (e.g. MERFISH, Visium, Slide-seqV2, HDST data from Broad Single Cell Portal accessions SCP420, SCP815)
- 2D spatial coordinates (puck coordinates) for every cell/bead
- Optionally a restricted gene set such as the top 1000 highly variable genes

## Outputs

- Per-gene SVG ranks and p-values from singleCellHaystack (res.top_features ranked list)
- Per-gene SVG ranks and p-values from SPARK-X for the same inputs
- Side-by-side rank/p-value comparison table (svg_rank_comparison.csv) with agreement flags for target genes

## How to apply

Load the count matrix and spatial coordinates into an AnnData object with scanpy (sc.read_h5ad), attaching the 2D puck/spatial coordinates as the input positions. Run singleCellHaystack (R >= 1.0.0 from CRAN, or Python 0.0.5 from PyPI) on all genes using the 2D coordinates directly — the method uses continuous activity levels rather than a hard binary detection threshold and does not assume a specific statistical distribution. Optionally run SPARK-X on the identical matrix and coordinates to obtain competing per-gene SVG p-values and ranks, then extract each method's top_features ranking (res.top_features(n=...)) and compute per-gene rank differences. Validate agreement by checking that cross-method Spearman rank correlations of SVG rankings fall in the ~0.70-0.95 range, and write per-gene rank/p-value side-by-side comparisons (e.g. svg_rank_comparison.csv) flagging discrepant genes.

## Related tools

- **singleCellHaystack (R)** (Primary SVG/differentially-active-feature prediction method run on 2D spatial coordinates; produces per-gene ranks and p-values) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack (Python)** (Python implementation (PyPI 0.0.5) of singleCellHaystack >= 1.0.0 for scalable application to spatial datasets via scanpy AnnData) — https://github.com/ddiez/singleCellHaystack-py
- **SPARK-X** (Competing SVG method used for cross-method rank/p-value comparison and runtime benchmarking)
- **SeuratWrappers** (Provides Seurat access to community methods (e.g. singleCellHaystack via wrappers) for spatial feature detection) — https://github.com/satijalab/seurat-wrappers

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Cross-method Spearman rank correlations between singleCellHaystack and SPARK-X SVG rankings fall in the typical 0.70-0.95 range.
- Known discrepant genes reproduce: in HDST, Gm10925 ranks 13th by singleCellHaystack (p=2.7e-39) vs 105th by SPARK-X (p=0.013); in Slide-seqV2 hippocampus, Plp1 ranks 20th vs 1139th and Mal 55th vs 1543rd.
- Output schema check: res.top_features returns a ranked gene list with per-gene p-values, exportable as svg_rank_comparison.csv with per-gene ranks and agreement flags.
- Runtime benchmark sanity: SPARK-X should generally be the fastest method, followed by singleCellHaystack, when run on one core across MERFISH/Visium/Slide-seqV2/HDST top-1000-HVG inputs.

## Limitations

- Comparisons between multiple conditions (e.g. wild-type and knockout) cannot be easily conducted with singleCellHaystack.
- singleCellHaystack has relatively longer runtimes than SPARK-X for spatial transcriptomics, though it offers better sensitivity.
- The Python implementation is in beta: weights.advanced.Q, the seeding method for grid points, and hierarchical clustering for cluster_genes are not yet available.
- Compared to cluster-based approaches, integrating singleCellHaystack results into cluster-oriented scRNA-seq analyses is not straightforward.

## Evidence

- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned inside any input space (1D, 2D, or >2D).: [evidence span withheld — anchor status: not_found]
- [abstract] it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [discussion] For spatial transcriptomics the fastest method available is SPARK-X.: "For spatial transcriptomics the fastest method available is SPARK-X."
- [discussion] The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes.: "The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes."
- [methods] res = hs.haystack(adata, basis="pca")
res.top_features(n=10): "res = hs.haystack(adata, basis="pca")"
