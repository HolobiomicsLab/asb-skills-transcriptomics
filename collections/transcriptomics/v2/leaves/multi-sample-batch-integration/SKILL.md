---
name: multi-sample-batch-integration
description: Use when you need to remove technical/sample effects from multi-sample
  single-cell RNA-seq data while preserving cell-type variability, and to quantify
  how GEDI compares with other integration methods across datasets and latent-factor
  settings.
when_to_use_negative:
- Data are single-sample with no inter-sample technical variability to harmonize —
  GEDI's integration objective is unnecessary
- The modality is ratio-based (splicing/stability) or the goal is cluster-free differential
  expression, pathway, or TF activity analysis rather than cross-method integration
  benchmarking
- Only gene-level imputation quality is of interest — the article notes GEDI's imputations
  have not been benchmarked against existing imputation methods
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GEDI
  role: Primary generative model producing the integrated latent embedding (mode =
    "Bsphere", oi_shrinkage = 0.001); installed from GitHub
  repo: https://github.com/csglab/GEDI/
- name: scuttle
  role: Performs QC filtering of low-quality cells and low-expressed genes before
    integration
- name: scran
  role: Bioconductor dependency used in pre-processing alongside scuttle/scater
  repo: https://bioconductor.org/packages/release/bioc/html/scran.html
- name: scater
  role: Bioconductor dependency used in pre-processing alongside scuttle/scran
  repo: https://bioconductor.org/packages/release/bioc/html/scater.html
- name: uwot
  role: Computes 2D UMAP embeddings of each method's integrated space (min_dist =
    0.01, euclidean metric)
  repo: https://cran.r-project.org/web/packages/uwot/index.html
- name: rsvd
  role: Required R package; supports the randomized SVD step (svd.gedi) recovering
    GEDI's embedding
  repo: https://cran.r-project.org/web/packages/rsvd/index.html
- name: Seurat
  role: Comparison integration method run over the same latent-factor range on the
    same datasets
- name: Harmony
  role: Comparison integration method in the benchmark
- name: LIGER
  role: Comparison integration method in the benchmark
- name: GEDI_manuscript
  role: Repository of R markdown notebooks reproducing the manuscript's benchmark
    figures and analyses
  repo: https://github.com/csglab/GEDI_manuscript
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:81a3f6a36a5d36e46789d5feaaf5e2ef97deb320ee0fc268ebf3e5b6004fdbc0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - GEDI_manuscript
  - Harmony
  - LIGER
  - R
  - Seurat
  - rsvd
  - scater
  - scran
  - scuttle
  - uwot
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multi-sample-batch-integration

## Summary

Benchmark GEDI, a generative Bayesian framework that unifies inter-sample integration, denoising, and low-dimensional embedding, against state-of-the-art single-cell integration methods (Seurat, LIGER, Harmony, BBKNN, scVI, Scanorama, CSS, PCA) across PBMC, pancreas, and Tabula Muris datasets, scoring batch removal and biological conservation.

## When to use

Use when you need to remove technical/sample effects from multi-sample single-cell RNA-seq data while preserving cell-type variability, and to quantify how GEDI compares with other integration methods across datasets and latent-factor settings.

## When NOT to use

- Data are single-sample with no inter-sample technical variability to harmonize — GEDI's integration objective is unnecessary
- The modality is ratio-based (splicing/stability) or the goal is cluster-free differential expression, pathway, or TF activity analysis rather than cross-method integration benchmarking
- Only gene-level imputation quality is of interest — the article notes GEDI's imputations have not been benchmarked against existing imputation methods

## Inputs

- Raw single-cell RNA-seq count matrices (dgCMatrix) from multi-sample datasets: PBMC (SeuratData pbmcsca / Ding et al.), pancreas, Tabula Muris
- Cell and gene metadata (sample labels, cell-type annotations)
- Latent factor range to sweep (e.g., K = 5 to 40)

## Outputs

- Integrated low-dimensional embeddings per method-dataset-K combination (e.g., GEDI: svd_res$v %*% diag(svd_res$d))
- 2D UMAP embeddings (uwot::umap, min_dist = 0.01, euclidean metric) colored by sample and cell type
- integration_benchmark_scores.csv with per-run Luecken-style scores and an overall score ranking

## How to apply

Pre-process each dataset (e.g., SeuratData pbmcsca for PBMC, plus pancreas and Tabula Muris) with scuttle QC: keep cells with log10_total_counts > 2.5, log10_total_features_by_counts > 2, and mitochondrial percentage <= 20, then retain genes with more than 3 counts in more than 3 cells. Run GEDI via new("GEDI"), model$setup on raw counts as a dgCMatrix with mode = "Bsphere" and oi_shrinkage = 0.001 (randomSeed = 1), sweeping the number of latent factors K (e.g., 5 to 40) and checking convergence with model$plotTracking(). Run each comparison method (Seurat, LIGER, Harmony, BBKNN, scVI, Scanorama, CSS, PCA baseline) over the same K range, recover each integrated space (for GEDI via svd.gedi and svd_res$v %*% diag(svd_res$d)), and embed in 2D with uwot::umap(min_dist = 0.01, metric = "euclidean"), visualizing with GEDI::plot_embedding colored by sample and cell type. Finally, compute Luecken-style integration metrics quantifying batch removal and cell-type conservation for every method-dataset-K combination, aggregate into an overall score, and rank methods in integration_benchmark_scores.csv.

## Related tools

- **GEDI** (Primary generative model producing the integrated latent embedding (mode = "Bsphere", oi_shrinkage = 0.001); installed from GitHub) — https://github.com/csglab/GEDI/
- **scuttle** (Performs QC filtering of low-quality cells and low-expressed genes before integration)
- **scran** (Bioconductor dependency used in pre-processing alongside scuttle/scater) — https://bioconductor.org/packages/release/bioc/html/scran.html
- **scater** (Bioconductor dependency used in pre-processing alongside scuttle/scran) — https://bioconductor.org/packages/release/bioc/html/scater.html
- **uwot** (Computes 2D UMAP embeddings of each method's integrated space (min_dist = 0.01, euclidean metric)) — https://cran.r-project.org/web/packages/uwot/index.html
- **rsvd** (Required R package; supports the randomized SVD step (svd.gedi) recovering GEDI's embedding) — https://cran.r-project.org/web/packages/rsvd/index.html
- **Seurat** (Comparison integration method run over the same latent-factor range on the same datasets)
- **Harmony** (Comparison integration method in the benchmark)
- **LIGER** (Comparison integration method in the benchmark)
- **GEDI_manuscript** (Repository of R markdown notebooks reproducing the manuscript's benchmark figures and analyses) — https://github.com/csglab/GEDI_manuscript

## Examples

```
model <- new("GEDI"); model$setup(data, mode="Bsphere", K=K, oi_shrinkage=0.001, randomSeed=1); model$run(); svd_res <- svd.gedi(model); emb <- svd_res$v %*% diag(svd_res$d)
```

## Evaluation signals

- integration_benchmark_scores.csv contains a scored entry for every method-dataset-factor combination (9 methods x 3 datasets x every K swept)
- GEDI's aggregated overall score places it consistently among the top-performing integration methods, matching the paper's report of on-par performance with state-of-the-art methods
- Convergence of each GEDI run is verified with model$plotTracking() before extracting embeddings
- UMAP plots colored by sample show reduced sample mixing separation while cell-type structure remains visually preserved across datasets
- Luecken-style metrics show both batch-removal and cell-type conservation (bio-conservation) components computed, not batch removal alone

## Limitations

- More extensive analyses are needed to understand factors (number of DE genes per cell, DE magnitude, correlations, variances, sample/cell counts, sequencing depth) that influence performance of GEDI and other methods
- GEDI's gene-level expression imputations have not been benchmarked against existing single-cell imputation methods
- The GEDI package was tested using R 4.0.0 on CentOS Linux 7 and requires gfortran for RcppEigen
- Only Luecken-style metric aggregation and the specific K sweep in the tutorials were used; performance across other K ranges or datasets is not established here

## Evidence

- [results] GEDI was consistently among the top-performing integration methods across the PBMC, pancreas, and Tabula Muris benchmarks: [evidence span withheld — anchor status: not_found]
- [methods] QC filter thresholds: "filter<- sce$log10_total_counts > 2.5 &
    sce$log10_total_features_by_counts > 2 &
    sce$subsets_Mito_percent <= 20"
- [methods] Gene filter threshold: "sum_genes<- rowSums(assay(sce, "counts")>3)
genes_use<- sum_genes>3"
- [abstract] GEDI abstract claim: "GEDI enables cross-sample cell state mapping on par with state-of-the-art integration methods, cluster-free differential gene expression analysis"
- [readme] README usage pointers: [evidence span withheld — anchor status: external_doc]
- [intro] Unifying objective: "GEDI incorporates various single-cell analysis steps within a unified Bayesian framework that includes data integration across samples/conditions, data imputation and denoising, cluster-free DGE"
