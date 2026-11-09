---
name: latent-factor-sweep-benchmarking
description: Use when a multi-sample single-cell integration method must be shown
  competitive against alternatives (Seurat, Harmony, LIGER, scVI, Scanorama, BBKNN,
  CSS) across datasets while sweeping the number of latent factors K.
when_to_use_negative:
- When analyzing a single already-fitted GEDI model for downstream tasks (cluster-free
  DGE, pathway/TF activity) rather than comparing methods — this skill is only for
  integration benchmarking
- When the goal is to evaluate GEDI's gene-level expression imputations against other
  imputation methods — the article states this comparison 'remains to be tested' and
  is out of scope here
- When the input is not raw per-cell counts needing inter-sample harmonization (e.g.,
  data already integrated or ratio-based splicing/stability modalities handled by
  GEDI's other modes)
tools:
- name: GEDI
  role: Method under test; fit per dataset with new('GEDI') and model$setup(mode='Bsphere',
    oi_shrinkage=0.001, randomSeed=1), K swept over the latent-factor range; latent
    space recovered via svd.gedi
  repo: https://github.com/csglab/GEDI
- name: scuttle
  role: QC of cells (log10_total_counts > 2.5, log10_total_features_by_counts > 2,
    subsets_Mito_percent <= 20) applied identically to every dataset before any method
    runs
- name: scran
  role: Pre-processing library alongside scuttle/scater for filtering low-quality
    cells and low-expressed genes
  repo: https://bioconductor.org/packages/release/bioc/html/scran.html
- name: uwot
  role: 2D UMAP embedding of every method's recovered integrated space with min_dist
    = 0.01, metric = 'euclidean' for visual comparison
  repo: https://cran.r-project.org/web/packages/uwot/index.html
- name: rsvd
  role: SVD dependency used to recover GEDI's latent embedding (svd_res$v %*% diag(svd_res$d))
  repo: https://cran.r-project.org/web/packages/rsvd/index.html
- name: Seurat
  role: Comparison integration method run on the same datasets over the same K range
- name: Harmony
  role: Comparison integration method in the benchmark; also used in the paper for
    COVID-19 cohort 1 integrated space
- name: LIGER
  role: Comparison integration method (its inter-sample harmonization approach inspired
    part of GEDI)
- name: scVI
  role: Comparison integration method scored alongside GEDI
- name: Scanorama
  role: Comparison integration method scored alongside GEDI
- name: BBKNN
  role: Comparison integration method scored alongside GEDI
- name: CSS
  role: Comparison integration method scored alongside GEDI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:73f26e5d6e595519b1095ee290ff8f23cafd435115a6b31b8a7eb34f1421b003
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - BBKNN
  - CSS
  - GEDI
  - Harmony
  - LIGER
  - R
  - Scanorama
  - Seurat
  - rsvd
  - scVI
  - scater
  - scran
  - scuttle
  - uwot
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# latent-factor-sweep-benchmarking

## Summary

Benchmarks GEDI's cross-sample integration against competing single-cell integration methods (Seurat, LIGER, Harmony, BBKNN, scVI, Scanorama, CSS, PCA) while sweeping the number of latent factors (e.g., K = 5 to 40), scoring batch removal and biological conservation with Luecken-style metrics.

## When to use

Use when a multi-sample single-cell integration method must be shown competitive against alternatives (Seurat, Harmony, LIGER, scVI, Scanorama, BBKNN, CSS) across datasets while sweeping the number of latent factors K.

## When NOT to use

- When analyzing a single already-fitted GEDI model for downstream tasks (cluster-free DGE, pathway/TF activity) rather than comparing methods — this skill is only for integration benchmarking
- When the goal is to evaluate GEDI's gene-level expression imputations against other imputation methods — the article states this comparison 'remains to be tested' and is out of scope here
- When the input is not raw per-cell counts needing inter-sample harmonization (e.g., data already integrated or ratio-based splicing/stability modalities handled by GEDI's other modes)

## Inputs

- Raw scRNA-seq count matrices as dgCMatrix from PBMC (SeuratData pbmcsca), pancreas, and Tabula Muris datasets
- Cell-type annotations for each dataset (for biological-conservation scoring)
- Sample/dataset labels for batch-removal scoring

## Outputs

- integration_benchmark_scores.csv with per-run (method × dataset × K) and overall Luecken-style scores
- 2D UMAP embeddings (uwot::umap, min_dist = 0.01, metric = 'euclidean') of each method's integrated latent space
- GEDI::plot_embedding figures colored by sample and cell type

## How to apply

Pre-process each dataset (PBMC via SeuratData pbmcsca, pancreas, Tabula Muris) with scuttle QC — retain cells with log10_total_counts > 2.5, log10_total_features_by_counts > 2, and subsets_Mito_percent <= 20, then keep genes with >3 counts in >3 cells (sum_genes <- rowSums(assay(sce, 'counts')>3); genes_use <- sum_genes>3). Fit GEDI per dataset with new('GEDI'), model$setup with raw counts as a dgCMatrix, mode = 'Bsphere', oi_shrinkage = 0.001, randomSeed = 1, sweeping K over a latent-factor range (e.g., K = 5 to 40) and checking convergence with model$plotTracking(). Run the comparison methods over the same datasets and latent-factor range; recover each method's integrated space (GEDI via svd.gedi and svd_res$v %*% diag(svd_res$d)). Embed every result in 2D with uwot::umap(min_dist = 0.01, metric = 'euclidean') and visualize with GEDI::plot_embedding colored by sample and cell type. Finally compute Luecken-style metrics quantifying batch removal and cell-type conservation per method-dataset-K combination, aggregate into an overall score, and write all scores to integration_benchmark_scores.csv.

## Related tools

- **GEDI** (Method under test; fit per dataset with new('GEDI') and model$setup(mode='Bsphere', oi_shrinkage=0.001, randomSeed=1), K swept over the latent-factor range; latent space recovered via svd.gedi) — https://github.com/csglab/GEDI
- **scuttle** (QC of cells (log10_total_counts > 2.5, log10_total_features_by_counts > 2, subsets_Mito_percent <= 20) applied identically to every dataset before any method runs)
- **scran** (Pre-processing library alongside scuttle/scater for filtering low-quality cells and low-expressed genes) — https://bioconductor.org/packages/release/bioc/html/scran.html
- **uwot** (2D UMAP embedding of every method's recovered integrated space with min_dist = 0.01, metric = 'euclidean' for visual comparison) — https://cran.r-project.org/web/packages/uwot/index.html
- **rsvd** (SVD dependency used to recover GEDI's latent embedding (svd_res$v %*% diag(svd_res$d))) — https://cran.r-project.org/web/packages/rsvd/index.html
- **Seurat** (Comparison integration method run on the same datasets over the same K range)
- **Harmony** (Comparison integration method in the benchmark; also used in the paper for COVID-19 cohort 1 integrated space)
- **LIGER** (Comparison integration method (its inter-sample harmonization approach inspired part of GEDI))
- **scVI** (Comparison integration method scored alongside GEDI)
- **Scanorama** (Comparison integration method scored alongside GEDI)
- **BBKNN** (Comparison integration method scored alongside GEDI)
- **CSS** (Comparison integration method scored alongside GEDI)

## Examples

```
model <- new("GEDI"); model$setup(counts_dgCMatrix, mode="Bsphere", oi_shrinkage=0.001, randomSeed=1)
```

## Evaluation signals

- integration_benchmark_scores.csv contains a scored entry for every method-dataset-latent-factor combination (no gaps in the method × dataset × K grid)
- GEDI's aggregated overall score places it consistently among the top-performing integration methods, matching the paper's report of parity with state-of-the-art integration
- Convergence of every GEDI fit was confirmed via model$plotTracking() before scores were recorded
- Top rank stability holds regardless of the number of latent factors used for low-dimensional projection of the data
- QC thresholds (log10_total_counts > 2.5, log10_total_features_by_counts > 2, subsets_Mito_percent <= 20; genes with >3 counts in >3 cells) were applied uniformly before all methods

## Limitations

- GEDI currently supports only linear manifold learning (optionally restricted to a hyperellipsoid); other parametric manifold approximations are future extensions
- Results depend on the chosen K sweep range (e.g., K = 5 to 40) and on the Luecken-style metric aggregation used for the overall score
- Environment sensitivity: code tested with R 4.0.0 on CentOS Linux 7; installing GEDI requires Rcpp/RcppEigen and gfortran
- Properly modeling sample-to-sample variability is key for unbiased differential expression, so poorly converged or mis-specified K values can distort both integration and downstream comparisons

## Evidence

- [methods] QC thresholds for cell filtering: "sum_genes<- rowSums(assay(sce, "counts")>3)
genes_use<- sum_genes>3"
- [methods] GEDI run follows pre-processing: "After pre-processing, we are ready to run GEDI."
- [abstract] GEDI parity with integration state-of-the-art: "GEDI enables cross-sample cell state mapping on par with state-of-the-art integration methods, cluster-free differential gene expression analysis"
- [readme] Package install command for GEDI: [evidence span withheld — anchor status: external_doc]
- [readme] Tested environment: [evidence span withheld — anchor status: external_doc]
