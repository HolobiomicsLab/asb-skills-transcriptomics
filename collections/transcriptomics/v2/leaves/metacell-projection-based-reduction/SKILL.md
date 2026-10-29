---
name: metacell-projection-based-reduction
description: Use when a scRNA-seq dataset is too large for standard metacell construction
  (time/memory limits at hundreds of thousands of cells), or when built metacells
  must be projected onto a single-cell UMAP/embedding for visualization and QC.
when_to_use_negative:
- The dataset is small enough that standard direct metacell construction (MC2, SuperCell,
  SEACells) is already tractable — the projection variant is specifically motivated
  by scaling limits on large data
- Per-cell resolution is required for the downstream analysis (metacells deliberately
  aggregate profiles of the same biological state, reducing ~100-fold)
- The goal is interactive exploration/annotation of an already-built metacell model
  — MCView (shiny-app) is the appropriate tool rather than re-running projection
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SuperCell
  role: Provides the projection-based metacell construction variant used at graining
    level 75 to scale to the largest MOCA subsets in the benchmark
  repo: https://github.com/GfellerLab/SuperCell
- name: MetacellAnalysisToolkit
  role: Supplies mc_projection(), qc_boxplot() and the integrated pipeline to build,
    visualize and evaluate metacells with different methods
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: Seurat
  role: Single-cell normalization, variable feature selection, scaling, PCA and UMAP
    embedding that provide the low-dimensional space onto which metacells are projected
- name: SEACells
  role: Alternative metacell constructor (archetypal analysis on a nearest-neighbor
    kernel) included in the same benchmark; tractable only on subset sizes it can
    handle
  repo: https://github.com/dpeerlab/SEACells
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:7466b1f71edb9b3f90cfaad528a415d084a99750a3d7c9127d8e23c09f4b9609
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GfellerLab/SuperCell
  tools:
  - MetacellAnalysisToolkit
  - SEACells
  - Seurat
  - SuperCell
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# metacell-projection-based-reduction

## Summary

Reduces large single-cell RNA-seq datasets to metacell level using projection-based approaches (e.g. the projection-based SuperCell variant) and maps the resulting metacells back onto single-cell embeddings such as UMAP via mc_projection(). This strategy is what allows metacell construction to scale to very large atlases (e.g. the 900k-cell MOCA subsets) where direct construction is time- and memory-consuming.

## When to use

Use when a scRNA-seq dataset is too large for standard metacell construction (time/memory limits at hundreds of thousands of cells), or when built metacells must be projected onto a single-cell UMAP/embedding for visualization and QC.

## When NOT to use

- The dataset is small enough that standard direct metacell construction (MC2, SuperCell, SEACells) is already tractable — the projection variant is specifically motivated by scaling limits on large data
- Per-cell resolution is required for the downstream analysis (metacells deliberately aggregate profiles of the same biological state, reducing ~100-fold)
- The goal is interactive exploration/annotation of an already-built metacell model — MCView (shiny-app) is the appropriate tool rather than re-running projection

## Inputs

- scRNA-seq count matrices (e.g. 10x-formatted MOCA matrices from GEO GSE114953, 10k–900k cells)
- Seurat object with normalization, PCA and UMAP embedding (dims = 1:30 used for diffusion components in the tutorial)
- Cell-to-metacell membership table (cell_membership$membership)
- Target graining level (e.g. 75)

## Outputs

- Metacell object with cell-membership assignments and aggregated metacell-level expression profiles
- Metacell coordinates projected on the single-cell UMAP (mc_projection output)
- QC metrics per metacell (purity, compactness, separation, INV) and qc_boxplot visualizations
- Scaling benchmark measurements (CPU time, peak memory) e.g. assembled into scaling_benchmark.csv and fig5c_scaling_curves.png

## How to apply

Preprocess the single-cell data with Seurat (NormalizeData with LogNormalize, variable feature selection, scaling, PCA and UMAP embedding). Build metacells at a target graining level (e.g. 75) using the projection-based SuperCell variant, which — unlike most methods in the MOCA scaling benchmark — extends tractably to the largest dataset sizes (up to 900,000 cells from GSE114953). Project the resulting metacells back onto the single-cell UMAP with MetacellAnalysisToolkit's mc_projection(), passing the Seurat object, the metacell object and the cell-membership table. Evaluate the reduction with metacell-level QC metrics (purity, compactness, separation, inner normalized variance) and visualize with qc_boxplot(); in the benchmark context, record elapsed CPU time and peak memory per method/variant/size to confirm the projection variant extends where others fail.

## Related tools

- **SuperCell** (Provides the projection-based metacell construction variant used at graining level 75 to scale to the largest MOCA subsets in the benchmark) — https://github.com/GfellerLab/SuperCell
- **MetacellAnalysisToolkit** (Supplies mc_projection(), qc_boxplot() and the integrated pipeline to build, visualize and evaluate metacells with different methods) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **Seurat** (Single-cell normalization, variable feature selection, scaling, PCA and UMAP embedding that provide the low-dimensional space onto which metacells are projected)
- **SEACells** (Alternative metacell constructor (archetypal analysis on a nearest-neighbor kernel) included in the same benchmark; tractable only on subset sizes it can handle) — https://github.com/dpeerlab/SEACells

## Examples

```
mc_projection(sc.obj = sc_data, mc.obj = mc_data, cell.membership = membership_df)
```

## Evaluation signals

- The benchmark table (scaling_benchmark.csv) covers every method × variant × dataset-size combination, with the projection-based SuperCell variant reaching the 900k-cell dataset (Fig 5C behavior)
- Projected metacell positions on the single-cell UMAP are consistent with the cell types of their member cells (purity metric)
- QC boxplots of purity/compactness/separation/INV show metacells are compact and well-separated after projection
- Recorded elapsed CPU time and peak memory per run show the projection variant avoids the time/memory blow-up seen for non-divide-and-conquer methods on large MOCA subsets

## Limitations

- Except for the divide-and-conquer MC2 method, standard metacell construction is time- and memory-consuming on large datasets — this motivates, but also limits, direct (non-projection) use
- SEACells can only be run on the subset sizes tractable for that method, so full scaling curves are not comparable across all methods
- Version fragmentation: SuperCell2.0 lives on a separate branch with multimodal/semi-supervised changes; tanaylab/metacells 0.9 breaks compatibility with 0.8 result annotations, and MCView since 0.2.7 only supports the metacells package format
- Metacells are not cell types or parametric models — they are robust state descriptions and must be further analyzed as such

## Evidence

- [methods] we can project the metacells on the single-cell UMAP representation using the `mc_projection()` function: "we can project the metacells on the single-cell UMAP representation using the `mc_projection()` function"
- [abstract] a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods: "a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods"
- [readme] The python package contains various algorithmic improvements and is scalable for larger data sets (millions of cells).: [evidence span withheld — anchor status: external_doc]
- [readme] SuperCell2.0 introduces substantial improvements, including multimodal metacell identification, a semi-supervised workflow, and deeper integration with the Seurat framework.: [evidence span withheld — anchor status: not_found]
