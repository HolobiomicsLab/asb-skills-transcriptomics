---
name: sample-to-sample-variability-analysis
description: Use when you have raw single-cell RNA-seq count matrices spanning multiple
  samples and conditions (e.
when_to_use_negative:
- Input is already a feature/sample table with no raw single-cell counts — GEDI's
  generative count model cannot be fit
- Nonlinear manifold learning is required — GEDI currently supports only linear manifold
  learning (with optional hyperellipsoid restriction)
- Only a single sample or single condition is present — sample-to-sample/inter-condition
  modeling has nothing to deconvolve
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: GEDI
  role: Generative model fitting sample-specific manifold parameters (Qi) as a function
    of condition priors; source of latent embeddings, vector fields, and TF activities
  repo: https://github.com/csglab/GEDI
- name: Harmony
  role: Provides the integrated space for cohort 1 of the COVID-19 dataset used for
    transcriptomic vector field computation
- name: scuttle
  role: Basic QC and filtering of low-quality cells prior to running GEDI
- name: scran
  role: Single-cell normalization and preprocessing before GEDI model fitting
- name: uwot
  role: UMAP embedding used alongside GEDI outputs in notebooks
- name: R
  role: Implementation language for GEDI, notebooks, and downstream SVM evaluation
    (e.g., cross-cohort AUROC computation)
  repo: https://github.com/csglab/GEDI_manuscript
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:8f6575dfeb06695b6b83cae7cf771c72456bad3f6357a503bd1712b3a3f6a7a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - Harmony
  - R
  - scran
  - scuttle
  - uwot
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sample-to-sample-variability-analysis

## Summary

Analyzes sample-to-sample variability in multi-sample, multi-condition single-cell data using GEDI, a generative Bayesian model that unifies integration, denoising, cluster-free differential expression, and pathway/GRN activity inference into one interpretable latent embedding.

## When to use

Use when you have raw single-cell RNA-seq count matrices spanning multiple samples and conditions (e.g., COVID-19 vs. healthy cohorts) and need to deconvolve technical/biological sample-to-sample variability, integrate cohorts, or model sample-specific transformations as a function of disease status.

## When NOT to use

- Input is already a feature/sample table with no raw single-cell counts — GEDI's generative count model cannot be fit
- Nonlinear manifold learning is required — GEDI currently supports only linear manifold learning (with optional hyperellipsoid restriction)
- Only a single sample or single condition is present — sample-to-sample/inter-condition modeling has nothing to deconvolve

## Inputs

- Raw single-cell count matrix (genes x cells) per sample, e.g. COVID-19 cohort 1 atlas or PBMC data from Ding et al.
- Sample-level metadata matrix H encoding cohort and condition (e.g., healthy vs. severe COVID-19)
- Optional gene-level prior knowledge network (e.g., DoRothEA human GRN, high-confidence sets A/B/C)
- Trained GEDI model RDS file (e.g., from Zenodo record 8222040)
- Harmony-integrated embedding of cohort 1 samples (for vector field analysis)

## Outputs

- Sample-specific manifold parameter matrix Q (concatenated model$params$Qi; genes x latent variables by samples)
- Sample-characteristic predictions with AUROC (e.g., covid_svm_auroc.csv with values 1.0 and 0.97)
- Transcriptomic vector field over the integrated cell embedding, colored by cell type
- TF activity gradients (getA.gedi/getADB.gedi) and differential expression tests (getDiffExp.gedi)
- Denoised/imputed single-cell expression values and cluster-free DGE results

## How to apply

Pre-process with scuttle/scran/scater QC (filter cells on log10_total_counts > 2.5, log10_total_features_by_counts > 2, subsets_Mito_percent <= 20; filter genes with rowSums(counts>3) > 3), then run GEDI on the raw count matrix with a sample-level metadata/prior matrix H encoding condition (e.g., disease severity), removing columns of C that cause singularity via QR pivoting. Monitor convergence of the o, Z, Bi, Qi, oi and si components with model$plotTracking(). Assemble the sample-specific manifold parameter matrix Q by concatenating model$params$Qi across samples (rows = genes x latent variables, columns = samples); for sample-level prediction, select the top 20 most variable parameters via order(apply(Q,1,sd), decreasing=T)[1:20] and train an SVM on one cohort to predict disease status in an independent cohort (COVID-19 example: AUROC 1.0 and 0.97 across the two splits). For cell-level dynamics, compute transcriptomic vector fields in a Harmony-integrated embedding with svd.vectorField.gedi and TF activity gradients with svd.activityGradient.gedi (e.g., SPI1, CEBPA, SP1 in HLA-DRlo S100Ahi monocytes), validating concordance between gradient direction and vector field.

## Related tools

- **GEDI** (Generative model fitting sample-specific manifold parameters (Qi) as a function of condition priors; source of latent embeddings, vector fields, and TF activities) — https://github.com/csglab/GEDI
- **Harmony** (Provides the integrated space for cohort 1 of the COVID-19 dataset used for transcriptomic vector field computation)
- **scuttle** (Basic QC and filtering of low-quality cells prior to running GEDI)
- **scran** (Single-cell normalization and preprocessing before GEDI model fitting)
- **uwot** (UMAP embedding used alongside GEDI outputs in notebooks)
- **R** (Implementation language for GEDI, notebooks, and downstream SVM evaluation (e.g., cross-cohort AUROC computation)) — https://github.com/csglab/GEDI_manuscript

## Examples

```
top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20]
```

## Evaluation signals

- model$plotTracking() shows convergence of the o, Z, Bi, Qi, oi and si components
- Cross-cohort SVM on top-20 most variable Qi parameters reaches AUROC 1.0 (train cohort 2 / test cohort 1) and 0.97 (reverse split)
- Cell-type ranking of mean vector-field magnitudes reproduces the reported Fig. 3d order: plasmablasts, then HLA-DRlo S100Ahi monocytes, then neutrophils
- SPI1 activity gradient direction aligns with the severe-COVID vector field in monocytes and SPI1 mRNA is significantly up-regulated (getDiffExp.gedi)
- C matrix has no singular columns after QR-based column removal (C <- C[ , QR$pivot[seq_len(QR$rank)]])

## Limitations

- GEDI currently supports only linear manifold learning (with optional hyperellipsoid restriction); other parametric manifold approximations are future extensions
- GEDI's gene-level expression imputations have not been benchmarked against existing single-cell imputation methods
- Counterfactual/perturbation-response prediction via the vector field remains untested
- Requires gfortran for RcppEigen; code was tested with R 4.0.0 on CentOS Linux 7

## Evidence

- [abstract] GEDI identifies latent space variations: "We introduce GEDI, a generative model that identifies latent space variations in multi-sample, multi-condition single-cell datasets"
- [readme] GEDI unifies analysis steps: [evidence span withheld — anchor status: external_doc]
- [methods] Top-20 parameter selection: "top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20]"
- [discussion] GEDI approximates Harmony space as function of disease status: [evidence span withheld — anchor status: not_found]
- [discussion] Linear manifold limitation: "GEDI currently supports linear manifold learning, with the option to further restrict the manifold to a hyperellipsoid"
