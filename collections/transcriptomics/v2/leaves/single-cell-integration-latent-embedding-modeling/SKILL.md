---
name: single-cell-integration-latent-embedding-modeling
description: Use when input is a multi-sample, multi-condition single-cell count matrix
  (e.g., sparse dgCMatrix) with sample/condition covariates, and you need an integrated
  latent embedding plus cluster-free DE or sample-variability deconvolution across
  conditions.
when_to_use_negative:
- The manifold structure of interest is strongly non-linear — GEDI currently supports
  only linear manifold learning, with an optional hyperellipsoid restriction, and
  other parametric manifolds are future extensions.
- You only need standard pseudobulk DE per pre-defined cell type/cluster — a sequential
  normalization + integration + DESeq2 pseudobulk workflow suffices and the unified
  model is unnecessary.
- You need validated single-cell imputation specifically — GEDI's gene-level imputations
  have not been benchmarked against existing imputation methods.
tools:
- name: GEDI
  role: Core generative model fitted via model$setup (mode='Bl2'/'Bsphere', randomSeed=1)
    to produce the latent embedding and cluster-free DE estimates
  repo: https://github.com/csglab/GEDI/
- name: scuttle
  role: 'Basic quality control (cell filtering: log10_total_counts > 2.5, features
    > 2, mitochondrial percent <= 20) before fitting'
- name: scran
  role: Pre-processing/normalization dependency loaded alongside scuttle and scater
  repo: https://bioconductor.org/packages/release/bioc/html/scran.html
- name: Harmony
  role: Provides the integrated space that GEDI's sample-specific transformations
    approximate for the COVID-19 cohort 1 dataset
- name: LEMUR
  role: Latent embedding regression comparator; showed lower cluster-free DE correlation
    (median r = 0.14 vs GEDI's 0.4) on simulated data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:904ee539df84ec9754ee8fe177633f2f9c9e621ce7584b575eea03923e99d9e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - DESeq2
  - GEDI
  - Harmony
  - LEMUR
  - R
  - miloDE
  - scran
  - scuttle
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# single-cell-integration-latent-embedding-modeling

## Summary

Fit GEDI, a Bayesian generative model, to multi-sample, multi-condition single-cell count matrices to jointly perform normalization, denoising, inter-sample harmonization, and interpretable latent embedding that supports cluster-free differential gene expression. It replaces sequential normalization-then-integration-then-DGE workflows that ignore the interplay between integration and differential expression.

## When to use

Use when input is a multi-sample, multi-condition single-cell count matrix (e.g., sparse dgCMatrix) with sample/condition covariates, and you need an integrated latent embedding plus cluster-free DE or sample-variability deconvolution across conditions.

## When NOT to use

- The manifold structure of interest is strongly non-linear — GEDI currently supports only linear manifold learning, with an optional hyperellipsoid restriction, and other parametric manifolds are future extensions.
- You only need standard pseudobulk DE per pre-defined cell type/cluster — a sequential normalization + integration + DESeq2 pseudobulk workflow suffices and the unified model is unnecessary.
- You need validated single-cell imputation specifically — GEDI's gene-level imputations have not been benchmarked against existing imputation methods.

## Inputs

- Single-cell raw count matrix (genes x cells) as sparse dgCMatrix
- Sample annotation vector mapping cells to samples and conditions (e.g., disease status)
- Covariate/design matrix C (cells x covariates) encoding sample and condition effects
- Optional gene-level prior knowledge (e.g., DoRothEA TF-gene regulons, gene sets) for interpretable factors
- Optional pre-computed integrated embedding (e.g., from Harmony) to approximate

## Outputs

- GEDI model object with interpretable latent embedding of cells
- Denoised/imputed gene expression profiles per cell (MAP estimates of latent true profiles)
- Transcriptomic vector field and per-cell cluster-free differential expression log-fold-changes
- Sample-specific transformations and pathway/regulon activity projections per cell
- Convergence tracking plots (model$plotTracking)

## How to apply

Preprocess with scuttle/scran/scater: filter low-quality cells (log10_total_counts > 2.5, log10_total_features_by_counts > 2, subsets_Mito_percent <= 20) and low-expressed genes (keep genes with >3 cells having counts >3), then convert counts to sparse dgCMatrix. Construct a covariate/design matrix C mapping samples to conditions (remove columns causing singularity via QR rank), and fit GEDI via model$setup with n latent components, choosing mode='Bl2' (hyperplane, L2 norm of the whole B matrix fixed) or mode='Bsphere' (hyperellipsoid, L2 norms of B columns fixed), initializing with randomSeed = 1. Convergence is inspected with model$plotTracking. The resulting latent embedding, which can initialize from Harmony's integrated space (e.g., cohort 1 of the COVID-19 dataset), deconvolves sample-to-sample variability and yields a transcriptomic vector field from which cluster-free log-fold-changes per cell can be estimated; pathway/GRN activity can be projected by restricting the model with gene-level prior knowledge such as DoRothEA regulons.

## Related tools

- **GEDI** (Core generative model fitted via model$setup (mode='Bl2'/'Bsphere', randomSeed=1) to produce the latent embedding and cluster-free DE estimates) — https://github.com/csglab/GEDI/
- **scuttle** (Basic quality control (cell filtering: log10_total_counts > 2.5, features > 2, mitochondrial percent <= 20) before fitting)
- **scran** (Pre-processing/normalization dependency loaded alongside scuttle and scater) — https://bioconductor.org/packages/release/bioc/html/scran.html
- **Harmony** (Provides the integrated space that GEDI's sample-specific transformations approximate for the COVID-19 cohort 1 dataset)
- **LEMUR** (Latent embedding regression comparator; showed lower cluster-free DE correlation (median r = 0.14 vs GEDI's 0.4) on simulated data)

## Examples

```
devtools::install_github("csglab/GEDI"); model <- GEDI::model$setup(X = counts_dgCMatrix, C = C, mode = 'Bl2', randomSeed = 1)
```

## Evaluation signals

- model$plotTracking shows the objective has converged after fitting.
- No singularity in the C matrix: columns causing singular design matrices must have been removed via QR pivot/rank before fitting.
- On the simulated cohort-level benchmark (COVID-19 cohort 1 parameters), GEDI hyperplane (Bl2) single-cell DE estimates reach median Pearson r ≈ 0.4 against the ground-truth DE vector per cell, versus ≈ 0.14 for LEMUR.
- AUROC of up-regulated gene detection ≈ 0.79 (GEDI Bl2) vs ≈ 0.72 (LEMUR), and down-regulated ≈ 0.78 vs ≈ 0.55.
- Top latent parameters selected by standard deviation (e.g., top 20 most variable parameters of Q) capture reproducible sample-to-sample variability.

## Limitations

- GEDI currently supports only linear manifold learning, with an optional restriction of the manifold to a hyperellipsoid; other parametric manifold approximations are future extensions.
- It remains untested whether GEDI's gene-level expression imputations are competitive with existing single-cell imputation methods.
- Counterfactual prediction in unobserved cell types (e.g., perturbation response via the vector field) remains untested.
- More extensive analyses are needed to understand how the number of DE genes per cell, DE magnitude, cell-cell/gene-gene correlations, variances, sample/cell counts, and sequencing depth affect DE performance.
- The reference implementation was tested with R 4.0.0 on CentOS Linux 7 and requires Rcpp/RcppEigen (with gfortran).

## Evidence

- [abstract] GEDI is a generative model that identifies latent space variations in multi-sample, multi-condition single-cell datasets: "We introduce GEDI, a generative model that identifies latent space variations in multi-sample, multi-condition single-cell datasets"
- [intro] GEDI unifies analysis steps in a single Bayesian framework: "GEDI incorporates various single-cell analysis steps within a unified Bayesian framework that includes data integration across samples/conditions, data imputation and de"
- [methods] Cell and gene filtering thresholds used before fitting: "filter<- sce$log10_total_counts > 2.5 &
    sce$log10_total_features_by_counts > 2 &
    sce$subsets_Mito_percent <= 20"
- [discussion] Harmony space used as input for COVID-19 cohort 1: "we used the integrated space identified by Harmony41 for the samples in cohort 1 of the COVID-19 dataset"
- [readme] README overview of the package scope: [evidence span withheld — anchor status: external_doc]
- [readme] README installation command: [evidence span withheld — anchor status: external_doc]
