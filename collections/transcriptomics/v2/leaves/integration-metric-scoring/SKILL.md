---
name: integration-metric-scoring
description: Use when benchmarking whether an integration method (e.g. GEDI) removes
  technical/batch effects while preserving biological variability against competitors
  across multiple datasets and latent-dimension settings.
when_to_use_negative:
- When the goal is evaluating a single method's biological discovery (e.g. cluster-free
  DGE, pathway activity, or vector field analysis) rather than comparative benchmarking
  of integration performance
- When the input embedding was not produced on the same QC-filtered data and latent-factor
  range as the competitors, since score comparisons across method-dataset-K combinations
  would be invalid
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GEDI
  role: Integration method under test; latent space recovered via svd.gedi and svd_res$v
    %*% diag(svd_res$d)
  repo: https://github.com/csglab/GEDI
- name: Harmony
  role: Comparison integration method run over the same datasets and latent-factor
    range
- name: LIGER
  role: Comparison integration method whose inter-sample harmonization inspired GEDI's
    design
- name: Seurat
  role: Comparison integration method run on the same datasets
- name: uwot
  role: 2D UMAP embedding of each integrated space (min_dist = 0.01, euclidean) for
    visual scoring by sample and cell type
  repo: https://cran.r-project.org/web/packages/uwot/index.html
- name: scuttle
  role: QC filtering of cells (log10_total_counts > 2.5, log10_total_features_by_counts
    > 2, Mito percent <= 20) prior to scoring
  repo: https://bioconductor.org/packages/release/bioc/html/scran.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:01783942d8326157db2c9f0461f2154cccc90b544c1e0b26eb5d637bed4a4597
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - Harmony
  - LIGER
  - R
  - Seurat
  - rsvd
  - scater
  - scran
  - scuttle
  - uwot
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# integration-metric-scoring

## Summary

Scores single-cell data-integration methods (GEDI, Seurat, LIGER, Harmony, BBKNN, scVI, Scanorama, CSS, PCA) across datasets and latent-factor counts using Luecken-style metrics that quantify batch removal and cell-type conservation, aggregating them into an overall ranking written to integration_benchmark_scores.csv.

## When to use

Use when benchmarking whether an integration method (e.g. GEDI) removes technical/batch effects while preserving biological variability against competitors across multiple datasets and latent-dimension settings.

## When NOT to use

- When the goal is evaluating a single method's biological discovery (e.g. cluster-free DGE, pathway activity, or vector field analysis) rather than comparative benchmarking of integration performance
- When the input embedding was not produced on the same QC-filtered data and latent-factor range as the competitors, since score comparisons across method-dataset-K combinations would be invalid

## Inputs

- QC-filtered single-cell count matrices (dgCMatrix) from PBMC (pbmcsca), pancreas, and Tabula Muris datasets
- Integrated latent embeddings from each method (GEDI via svd.gedi output; competing methods' reduced-dim spaces)
- Sample labels and cell-type annotations for each dataset
- Latent-factor sweep range (e.g. K = 5 to 40)

## Outputs

- integration_benchmark_scores.csv with per-run scores for every method-dataset-factor combination
- Overall score ranking of integration methods per dataset
- UMAP embeddings (uwot::umap, min_dist = 0.01, euclidean metric) colored by sample and cell type

## How to apply

Run each integration method (GEDI initialized with new("GEDI") and model$setup on raw counts as dgCMatrix with mode="Bsphere" and oi_shrinkage=0.001, plus Seurat, LIGER, Harmony, BBKNN, scVI, Scanorama, CSS and a PCA baseline) on the same QC-filtered datasets — e.g. PBMC via SeuratData pbmcsca, pancreas, and Tabula Muris — sweeping the number of latent factors K (e.g. K = 5 to 40). Recover each method's integrated embedding (for GEDI via svd.gedi and svd_res$v %*% diag(svd_res$d)) and optionally embed in 2D with uwot::umap(min_dist = 0.01, metric = "euclidean"). For every method-dataset-K combination, compute Luecken-style integration metrics that separately quantify batch removal and biological (cell-type) conservation, then aggregate them into an overall score and write per-run and overall rankings to integration_benchmark_scores.csv. The rationale is GEDI's claim that it deconvolves intra-sample and inter-sample sources of variability, so scoring must capture both removal of technical effects and preservation of cell-type structure.

## Related tools

- **GEDI** (Integration method under test; latent space recovered via svd.gedi and svd_res$v %*% diag(svd_res$d)) — https://github.com/csglab/GEDI
- **Harmony** (Comparison integration method run over the same datasets and latent-factor range)
- **LIGER** (Comparison integration method whose inter-sample harmonization inspired GEDI's design)
- **Seurat** (Comparison integration method run on the same datasets)
- **uwot** (2D UMAP embedding of each integrated space (min_dist = 0.01, euclidean) for visual scoring by sample and cell type) — https://cran.r-project.org/web/packages/uwot/index.html
- **scuttle** (QC filtering of cells (log10_total_counts > 2.5, log10_total_features_by_counts > 2, Mito percent <= 20) prior to scoring) — https://bioconductor.org/packages/release/bioc/html/scran.html

## Examples

```
model <- new("GEDI"); model <- model$setup(data, mode="Bsphere", oi_shrinkage=0.001, randomSeed=1); model <- model$run() # K sweep 5-40, then score with Luecken metrics
```

## Evaluation signals

- integration_benchmark_scores.csv contains a scored entry for every method-dataset-factor combination (e.g. 9 methods x 3 datasets x each K)
- GEDI's overall score places it consistently among the top-performing integration methods, matching the paper's report that it performs on par with state-of-the-art integration methods
- Per-dataset scores are consistent across the K sweep (e.g. K = 5 to 40), reflecting the paper's claim that GEDI's ranking is robust to the number of latent factors
- Batch-removal and biological-conservation sub-metrics are both computed (Luecken-style), not a single blended score alone
- UMAP plots of each integrated space are colored by both sample and cell type so batch mixing and cell-type separation can be visually inspected

## Limitations

- GEDI was benchmarked only in its linear manifold mode (with optional hyperellipsoid restriction); other parametric manifold approximations are future extensions
- The same latent-factor range must be swept for all methods, or the comparison ranking is not comparable to the paper's result
- GEDI's imputation of gene-level expression values was not benchmarked against dedicated imputation methods, so this scoring covers integration embeddings only

## Evidence

- [results] GEDI was consistently among the top-performing integration methods across the PBMC, pancreas, and Tabula Muris benchmarks, regardless of the number of latent factors used for low-dimensional projection of data.: [evidence span withheld — anchor status: not_found]
- [methods] Filtering low-quality cells threshold: "sce$log10_total_counts > 2.5 & sce$log10_total_features_by_counts > 2 & sce$subsets_Mito_percent <= 20"
- [abstract] GEDI enabling cross-sample cell state mapping: "GEDI enables cross-sample cell state mapping on par with state-of-the-art integration methods, cluster-free differential gene expression analysis"
- [readme] GEDI package installation from GitHub: [evidence span withheld — anchor status: external_doc]
- [intro] GEDI deconvolving variability sources: "uniquely capable of deconvolving the effects of multiple technical and/or biological sources of sample-to-sample variability"
