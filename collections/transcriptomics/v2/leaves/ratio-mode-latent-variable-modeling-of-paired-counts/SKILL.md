---
name: ratio-mode-latent-variable-modeling-of-paired-counts
description: Use when input consists of paired, cell-by-event count matrices (inclusion/exclusion
  or spliced/unspliced reads) from multi-sample single-cell data and the goal is a
  harmonized latent embedding plus imputed ratio (e.g., logit PSI) values.
when_to_use_negative:
- Input is a single count matrix (gene expression) rather than paired ratio-based
  measurements (inclusion/exclusion, spliced/unspliced) — use GEDI's standard single-measurement
  mode instead
- The goal requires nonlinear manifold learning — GEDI currently supports only linear
  manifold learning with an optional hyperellipsoid restriction
- Data are not multi-sample — inter-sample harmonization (the main benefit of this
  mode) has no counterpart to remove
tools:
- name: GEDI
  role: Fits the binomial ratio-mode latent variable model to paired inclusion/exclusion
    counts; provides plotTracking, svd.gedi, plot_embedding, and getY.gedi
  repo: github.com/csglab/GEDI
- name: uwot
  role: UMAP 2D embedding of the GEDI svd integration space (min_dist = 0.01, euclidean
    metric)
  repo: https://cran.r-project.org/web/packages/uwot/index.html
- name: HDF5Array
  role: HDF5-backed loading of the SingleCellExperiment containing raw_M1 and raw_M2
    splicing count matrices
- name: Matrix
  role: Converts filtered M1/M2 count matrices to dgCMatrix sparse matrices for GEDI
    input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:553b7244b9424c82ddfffdaaaafe215f5ed75e908234a84f62b1b91e64e76c74
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GEDI
  - HDF5Array
  - Matrix
  - R
  - ggplot2
  - uwot
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ratio-mode-latent-variable-modeling-of-paired-counts

## Summary

GEDI's ratio mode fits a Bayesian latent variable model to paired count matrices (e.g., exon inclusion raw_M1 / exclusion raw_M2 from Quantas) by modeling the log-ratio with a binomial data-generating distribution, enabling inter-sample harmonization and imputed logit(PSI) estimates for ratio-based single-cell modalities like alternative splicing.

## When to use

Use when input consists of paired, cell-by-event count matrices (inclusion/exclusion or spliced/unspliced reads) from multi-sample single-cell data and the goal is a harmonized latent embedding plus imputed ratio (e.g., logit PSI) values.

## When NOT to use

- Input is a single count matrix (gene expression) rather than paired ratio-based measurements (inclusion/exclusion, spliced/unspliced) — use GEDI's standard single-measurement mode instead
- The goal requires nonlinear manifold learning — GEDI currently supports only linear manifold learning with an optional hyperellipsoid restriction
- Data are not multi-sample — inter-sample harmonization (the main benefit of this mode) has no counterpart to remove

## Inputs

- Paired cell-by-event count matrices: exon inclusion counts (raw_M1) and exclusion counts (raw_M2) from a Quantas-processed splicing study, e.g., stored as SingleCellExperiment assays in an HDF5-backed directory './my_h5_se'
- Cell metadata with Sample (study) and major_cell_type annotations for plotting and validation

## Outputs

- Fitted GEDI model object (e.g., saved as 'Tasic_gedi_model.rds') with trackable components o, Z, Bi, Qi, oi, si
- Integrated reference-space embedding via svd.gedi (svd_res$v %*% diag(svd_res$d)) and a 2D UMAP colored by Sample and major_cell_type
- Imputed logit(PSI) matrix from GEDI::getY.gedi for projecting individual cassette exons onto the embedding

## How to apply

Load exon inclusion counts (raw_M1) and exclusion counts (raw_M2) processed with the Quantas pipeline (e.g., from an HDF5-backed SingleCellExperiment in './my_h5_se'). Filter cells with log10(colSums(M1)) > 4.5 and log10(colSums(M2)) > 4, and events with rowSums(M1) > 50 and rowSums(M2) > 50, then convert to dgCMatrix sparse matrices. Configure GEDI with M = list(M1, M2) so the paired observations' log-ratio is modeled with a binomial distribution, using K = 20 latent variables, mode = 'Bsphere', oi_shrinkage = 0.001, randomSeed = 1, and 150 optimization iterations. Check convergence with model$plotTracking() (components o, Z, Bi, Qi, oi, si and MSE), and note that GEDI reorders cells by sample, so reorder metadata via model$aux$cellIDs. Recover the integrated space with svd.gedi (embedding = svd_res$v %*% diag(svd_res$d)), embed in 2D with uwot::umap (min_dist = 0.01, euclidean metric), and retrieve imputed logit(PSI) values with GEDI::getY.gedi to project specific cassette exons (e.g., Nrxn1 'CA-18189-887480-931921-932011-933369[INC][12/3]' and Nptn 'CA-20320-3287-39423-39771-44447[INC][8/120][UPT]').

## Related tools

- **GEDI** (Fits the binomial ratio-mode latent variable model to paired inclusion/exclusion counts; provides plotTracking, svd.gedi, plot_embedding, and getY.gedi) — github.com/csglab/GEDI
- **uwot** (UMAP 2D embedding of the GEDI svd integration space (min_dist = 0.01, euclidean metric)) — https://cran.r-project.org/web/packages/uwot/index.html
- **HDF5Array** (HDF5-backed loading of the SingleCellExperiment containing raw_M1 and raw_M2 splicing count matrices)
- **Matrix** (Converts filtered M1/M2 count matrices to dgCMatrix sparse matrices for GEDI input)

## Examples

```
model <- GEDI(M = list(M1, M2), K = 20, mode = 'Bsphere', oi_shrinkage = 0.001, randomSeed = 1, iter = 150)
```

## Evaluation signals

- model$plotTracking() shows convergence of model components (o, Z, Bi, Qi, oi, si) and mean squared error over the 150 iterations
- Integrated UMAP shows no obvious separation by study (Tasic 2016 vs 2018) while maintaining clear cell-type separation
- Imputed logit(PSI) embeddings of the Nrxn1 exon 20 and Nptn exon 2 events display structured, cell-type-specific patterns consistent with the published figure
- Cell and event filter thresholds are satisfied: log10(colSums(M1)) > 4.5, log10(colSums(M2)) > 4, rowSums(M1) > 50, rowSums(M2) > 50
- Cell-level metadata align correctly with GEDI's sample-reordered output via model$aux$cellIDs

## Limitations

- GEDI currently supports only linear manifold learning (with optional hyperellipsoid restriction); other parametric manifold approximations are future extensions
- No competing method exists for inter-sample harmonization of ratio-based modalities, so benchmarks against alternatives in this mode are limited (naïve PSI-based baselines outperformed in the cited task)
- Untested whether GEDI's imputations are competitive with existing single-cell imputation methods at the gene level

## Evidence

- [readme] Ratio-mode splicing tutorial exists in the GEDI package: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract claim that GEDI models paired exon inclusion/exclusion reads: "joint analysis of exon inclusion/exclusion reads to model alternative cassette exon splicing, or spliced/unspliced reads to model the mRNA stability landscapes"
- [methods] Methods cell-level filter for the paired count matrices: "filter<- which(log10(colSums(M1)) > 4.5 & log10(colSums(M2)) > 4)"
- [methods] Methods event-level filter for the paired count matrices: "filter <- which( rowSums(M1) > 50 & rowSums(M2) > 50 ) # Filter events"
- [intro] Motivation from the introduction: no method existed for harmonizing ratio-based modalities: "Few methods can perform dimensionality reduction on the latent space of ratio-based modalities"
