---
name: latent-variable-model-optimization
description: Use when fitting a generative latent embedding (GEDI) to raw single-cell
  counts (dgCMatrix) with optional prior knowledge matrix, needing explicit K, mode,
  oi_shrinkage, randomSeed and iteration settings plus convergence checks.
when_to_use_negative:
- Data has not passed cell/gene QC (log10 total counts > 2.5, log10 detected features
  > 2, mitochondrial percent <= 20; genes with > 5 counts in > 3 cells) — fit on the
  filtered matrix
- The analysis requires nonlinear manifold learning — GEDI currently supports only
  linear manifold learning with optional hyperellipsoid restriction
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: GEDI
  role: Core generative model fit with K, mode, oi_shrinkage, randomSeed and iteration
    control; provides plotTracking, svd.gedi, getA.gedi, getADB.gedi
  repo: https://github.com/csglab/GEDI
- name: scuttle
  role: Upstream QC of the SingleCellExperiment used to define the cell filter before
    model fitting
- name: scran
  role: Bioconductor dependency used in the pre-processing notebooks feeding the model
  repo: https://bioconductor.org/packages/release/bioc/html/scran.html
- name: scater
  role: Bioconductor dependency used in the pre-processing notebooks feeding the model
  repo: https://bioconductor.org/packages/release/bioc/html/scater.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:82d7d5f51762faa234a60b60f3670c352227e1757678b3e5e31d027ca9d255c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - DoRothEA
  - GEDI
  - HDF5Array
  - ggplot2
  - scater
  - scran
  - scuttle
  - uwot
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# latent-variable-model-optimization

## Summary

Configure and optimize a GEDI Bayesian latent-variable model on single-cell count data, using a fixed number of latent variables (K), optional prior matrix, manifold mode and shrinkage settings, then verify convergence of model components before downstream use.

## When to use

Use when fitting a generative latent embedding (GEDI) to raw single-cell counts (dgCMatrix) with optional prior knowledge matrix, needing explicit K, mode, oi_shrinkage, randomSeed and iteration settings plus convergence checks.

## When NOT to use

- Data has not passed cell/gene QC (log10 total counts > 2.5, log10 detected features > 2, mitochondrial percent <= 20; genes with > 5 counts in > 3 cells) — fit on the filtered matrix
- The analysis requires nonlinear manifold learning — GEDI currently supports only linear manifold learning with optional hyperellipsoid restriction

## Inputs

- Raw single-cell count matrix as dgCMatrix (M)
- Sample/condition vector keyed to cells (e.g. donor.method)
- Optional gene-level prior matrix C (e.g. DoRothEA-derived TF-gene matrix 'C_dorothea_tf_human.refined.rds'), pre-subset to shared genes and de-singulatized by QR pivot

## Outputs

- Trained GEDI model object with tracked components (o, Z, A, Bi, Qi, oi, si)
- SVD-based integrated embedding (svd_res$v %*% diag(svd_res$d)) for UMAP projection
- Latent-variable activity matrices via getA.gedi (activity x LVs) and getADB.gedi (activity x cell)

## How to apply

After QC filtering of cells and genes and preparing the raw counts as a dgCMatrix (M) plus any prior matrix C (subset both to shared genes in the same order and remove C columns causing singularity via QR decomposition using QR$pivot[seq_len(QR$rank)]), set up the GEDI model with K = 40 latent variables, mode = 'Bsphere' and oi_shrinkage = 0.001, supplying the sample vector (e.g. donor.method) to model inter-sample variability. Initialize latent variables with randomSeed = 1 and optimize for 500 iterations so the run is reproducible. Assess convergence with model$plotTracking(), confirming that the tracked components (o, Z, A, Bi, Qi, oi, si) have stabilized; if not, continue optimization. Because GEDI reorders cells by sample, reorder cell metadata via model$aux$cellIDs before any downstream analysis; recover the integrated embedding as svd_res$v %*% diag(svd_res$d).

## Related tools

- **GEDI** (Core generative model fit with K, mode, oi_shrinkage, randomSeed and iteration control; provides plotTracking, svd.gedi, getA.gedi, getADB.gedi) — https://github.com/csglab/GEDI
- **scuttle** (Upstream QC of the SingleCellExperiment used to define the cell filter before model fitting)
- **scran** (Bioconductor dependency used in the pre-processing notebooks feeding the model) — https://bioconductor.org/packages/release/bioc/html/scran.html
- **scater** (Bioconductor dependency used in the pre-processing notebooks feeding the model) — https://bioconductor.org/packages/release/bioc/html/scater.html

## Examples

```
model <- gedi(sample=donor.method, M=counts_dgCMatrix, C=C_prior, K=40, mode='Bsphere', oi_shrinkage=0.001, randomSeed=1, iter=500); model$plotTracking()
```

## Evaluation signals

- model$plotTracking() shows all tracked components (o, Z, A, Bi, Qi, oi, si) stabilized by 500 iterations
- Prior matrix C passes the QR rank check and no singularity errors are raised during optimization
- The dgCMatrix M and C share the same gene set in identical order before fitting
- Downstream UMAP of svd_res$v %*% diag(svd_res$d) shows a coherent integrated space across samples
- Identical randomSeed = 1 reproduces the same latent variable initialization across runs

## Limitations

- GEDI supports only linear manifold learning, with optional restriction of the manifold to a hyperellipsoid; other parametric manifold approximations are future extensions
- GEDI reorders cells by sample, so metadata must be re-matched via model$aux$cellIDs or results will be misaligned
- The model was tested with R 4.0.0 on CentOS Linux 7; RcppEigen requires gfortran to install

## Evidence

- [methods] C <- C[ , QR$pivot[seq_len(QR$rank)] ] # remove any columns that results in singularity: "C <- C[ , QR$pivot[seq_len(QR$rank)] ] # remove any columns that results in singularity"
- [discussion] GEDI currently supports linear manifold learning, with the option to further restrict the manifold to a hyperellipsoid: "GEDI currently supports linear manifold learning, with the option to further restrict the manifold to a hyperellipsoid"
- [readme] The code was tested using R 4.0.0 running on CentOS Linux 7.: [evidence span withheld — anchor status: external_doc]
- [methods] After pre-processing, we are ready to run GEDI.: "After pre-processing, we are ready to run GEDI."
- [readme] A generative model that unifies integration, cluster-free differential expression, pathway and regulatory network analysis, data normalization and imputation.: [evidence span withheld — anchor status: external_doc]
