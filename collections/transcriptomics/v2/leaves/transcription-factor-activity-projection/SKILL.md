---
name: transcription-factor-activity-projection
description: Use when you have a pre-processed single-cell count matrix (e.g. PBMC
  SingleCellExperiment) plus a TF-target prior such as DoRothEA, and want cell-level
  TF regulon activity scores and projections.
when_to_use_negative:
- No TF-gene prior knowledge (e.g. DoRothEA) is available for the organism or tissue,
  since the skill depends on the C matrix to define regulons
- The data are not gene-level counts but a ratio-based modality (e.g. splicing inclusion/exclusion
  reads); use GEDI's ratio mode instead
- The goal is standard clustering or generic dimensionality reduction without an interpretable
  regulon-activity readout
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GEDI
  role: Generative model that incorporates the DoRothEA prior matrix C to infer and
    project TF regulon activities per cell (getA.gedi, getADB.gedi, plot_embedding)
  repo: https://github.com/csglab/GEDI
- name: scuttle
  role: Basic QC of the PBMC SingleCellExperiment (library-size, detected-feature,
    and mitochondrial filters) before fitting GEDI
  repo: https://bioconductor.org/packages/release/bioc/html/scater.html
- name: DoRothEA
  role: Source of the human TF-gene regulatory network used to build the prior matrix
    C_dorothea_tf_human.refined.rds
  repo: https://saezlab.github.io/dorothea/
- name: uwot
  role: Computes the 2D UMAP embedding (min_dist = 0.01, euclidean) on which TF activities
    are projected
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:277f441b9f48513b655758c0456e96d4a62c713704829930d40886043ba57554
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# transcription-factor-activity-projection

## Summary

Uses GEDI with a DoRothEA-derived TF-gene prior matrix (C) to project transcription factor regulon activities onto single cells in the PBMC/COVID-19 data, yielding interpretable, cell-type-restricted activity embeddings validated against TF mRNA abundance (74 of 89 TFs significant).

## When to use

Use when you have a pre-processed single-cell count matrix (e.g. PBMC SingleCellExperiment) plus a TF-target prior such as DoRothEA, and want cell-level TF regulon activity scores and projections.

## When NOT to use

- No TF-gene prior knowledge (e.g. DoRothEA) is available for the organism or tissue, since the skill depends on the C matrix to define regulons
- The data are not gene-level counts but a ratio-based modality (e.g. splicing inclusion/exclusion reads); use GEDI's ratio mode instead
- The goal is standard clustering or generic dimensionality reduction without an interpretable regulon-activity readout

## Inputs

- Pre-processed PBMC SingleCellExperiment with raw counts (filtered: log10 total counts > 2.5, log10 features > 2, mito % <= 20; genes kept if >5 counts in >3 cells)
- DoRothEA-derived prior matrix 'C_dorothea_tf_human.refined.rds' (genes x TFs, high-confidence sets A, B, C)
- Sample/donor annotation vector (donor.method)

## Outputs

- Fitted GEDI model (K = 40 LVs, mode = 'Bsphere') with convergence tracking plots
- TF activity x latent variable matrix from getA.gedi and activity x cell matrix from getADB.gedi
- 2D UMAP (min_dist = 0.01, euclidean) with per-TF activity projections (e.g. PAX5, TCF7) via GEDI::plot_embedding
- Per-TF correlation statistics between projected activity and decoded TF mRNA abundance (74/89 TFs significant)

## How to apply

Filter the PBMC SingleCellExperiment (log10 total counts > 2.5, log10 detected features > 2, mitochondrial percent <= 20) and keep genes with more than 5 counts in more than 3 cells. Subset both the raw count matrix (as dgCMatrix) and the DoRothEA-derived prior 'C_dorothea_tf_human.refined.rds' to their shared genes in the same order, and remove columns of C that cause singularity via QR decomposition (C <- C[, QR$pivot[seq_len(QR$rank)]]) — GEDI restricts DoRothEA interactions to high-confidence sets A, B, and C. Fit the GEDI model with the sample vector (donor.method), K = 40 latent variables, mode = 'Bsphere', oi_shrinkage = 0.001, randomSeed = 1, and 500 iterations, confirming convergence with model$plotTracking(). Reorder metadata by model$aux$cellIDs (GEDI reorders cells by sample), recover the embedding with svd.gedi (svd_res$v %*% diag(svd_res$d)) and a 2D UMAP (min_dist = 0.01, euclidean), then retrieve TF activities with getA.gedi (activity x LVs) and getADB.gedi (activity x cell) and project selected TFs (e.g. PAX5, TCF7) onto the UMAP with GEDI::plot_embedding. Rationale: the prior-informed latent factors make TF activities interpretable per cell, and their reliability is judged by correlation with decoded TF mRNA abundance.

## Related tools

- **GEDI** (Generative model that incorporates the DoRothEA prior matrix C to infer and project TF regulon activities per cell (getA.gedi, getADB.gedi, plot_embedding)) — https://github.com/csglab/GEDI
- **scuttle** (Basic QC of the PBMC SingleCellExperiment (library-size, detected-feature, and mitochondrial filters) before fitting GEDI) — https://bioconductor.org/packages/release/bioc/html/scater.html
- **DoRothEA** (Source of the human TF-gene regulatory network used to build the prior matrix C_dorothea_tf_human.refined.rds) — https://saezlab.github.io/dorothea/
- **uwot** (Computes the 2D UMAP embedding (min_dist = 0.01, euclidean) on which TF activities are projected)

## Examples

```
model <- GEDI(M = counts_dgCMatrix, s = donor.method, C = C, K = 40, mode = "Bsphere", oi_shrinkage = 0.001, randomSeed = 1, niter = 500); A_cell <- GEDI::getADB.gedi(model)
```

## Evaluation signals

- Convergence plots (model$plotTracking()) show stabilized components (o, Z, A, Bi, Qi, oi, si) after 500 iterations
- For 74 out of 89 TFs in the network, projected activity across cells correlates significantly with the decoded mRNA abundance of the encoding TF
- Activity projections reproduce cell-type-restricted patterns: PAX5 activity localized to B cells and TCF7 activity localized to CD4+ T cells
- QR-decomposition rank check confirms no singular columns remain in the prior matrix C before model fitting
- Metadata rows match GEDI's reordered cells (model$aux$cellIDs) so activity projections align with correct cell annotations

## Limitations

- Columns of the prior matrix C that cause singularity must be removed via QR pivoting, so not all TFs in DoRothEA can be scored (89 TFs in the final network)
- TF activity reliability is assessed by correlation with decoded TF mRNA abundance, which may not capture post-transcriptional regulation
- The GEDI model requires R (tested on R 4.0.0 / CentOS 7) and RcppEigen with gfortran, which can complicate installation

## Evidence

- [abstract] GEDI can also incorporate gene-level prior knowledge to infer pathway and regulatory network activities in single cells.: "GEDI can also incorporate gene-level prior knowledge to infer pathway and regulatory network activities in single cells."
- [methods] Remove singular prior columns via QR decomposition: "C <- C[ , QR$pivot[seq_len(QR$rank)] ] # remove any columns that results in singularity"
- [methods] DoRothEA human regulatory network is the prior source: "we downloaded the human [DoRothEA](https://saezlab.github.io/dorothea/) gene regulatory network"
- [methods] TF-gene interactions restricted to high-confidence sets: "restricted the TF-gene interactions to the high-confidence sets A, B, and C"
- [readme] Manuscript hosts a dedicated TF activity notebook: [evidence span withheld — anchor status: external_doc]
