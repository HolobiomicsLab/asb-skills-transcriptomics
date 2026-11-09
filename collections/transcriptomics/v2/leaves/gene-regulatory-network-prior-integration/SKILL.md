---
name: gene-regulatory-network-prior-integration
description: Use when single-cell RNA-seq count data must be embedded with prior knowledge
  from a TF regulatory network (e.g., DoRothEA) so latent variables correspond to
  TF regulon activities, e.g., to score TF activity per cell or benchmark TF perturbation
  inference.
when_to_use_negative:
- The goal is a purely data-driven latent embedding without prior knowledge — omit
  the C prior and use GEDI's standard integration workflow instead.
- Input is not raw counts from scRNA-seq modeled with GEDI (e.g., pre-normalized feature
  tables); the prior-subset-then-QR workflow assumes a shared gene-space count matrix
  matching the DoRothEA network.
- The regulatory network is unrelated to the measured transcriptome such that after
  intersecting genes, singular-column removal via QR leaves no usable prior columns.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GEDI
  role: Fits the generative latent-variable model with the TF prior matrix C as prior
    knowledge and retrieves TF activities via getA.gedi/getADB.gedi; plot_embedding
    projects TF activity onto UMAP
  repo: https://github.com/csglab/GEDI
- name: DoRothEA
  role: Source of the human TF-gene regulatory network used to build the prior matrix
    C (high-confidence sets A, B, C, refined with GTEx whole blood)
  repo: https://saezlab.github.io/dorothea/
- name: scuttle
  role: Performs QC filtering of cells (counts, detected features, mitochondrial percent)
    before fitting the prior-informed model
  repo: https://csglab.github.io/GEDI_manuscript/
- name: scran
  role: Supports QC and low-expression gene filtering of the PBMC SingleCellExperiment
    prior to GEDI model setup
  repo: https://csglab.github.io/GEDI_manuscript/
- name: uwot
  role: Computes the 2D UMAP (min_dist = 0.01, euclidean metric) from the svd.gedi
    embedding onto which TF activities are projected
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:ed573443ae398c694bf57006eb71baf86e1733773b892eab66c5e48a308ae4a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - DoRothEA
  - GEDI
  - HDF5Array
  - R
  - decoupleR
  - ggplot2
  - scater
  - scran
  - scuttle
  - uwot
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gene-regulatory-network-prior-integration

## Summary

Integrate a TF-gene regulatory network prior (e.g., DoRothEA-derived) into the GEDI latent-variable model so that latent variables encode interpretable transcription factor regulon activities in single cells, validated by correlation of projected TF activity with the decoded abundance of the TF-encoding mRNA.

## When to use

Use when single-cell RNA-seq count data must be embedded with prior knowledge from a TF regulatory network (e.g., DoRothEA) so latent variables correspond to TF regulon activities, e.g., to score TF activity per cell or benchmark TF perturbation inference.

## When NOT to use

- The goal is a purely data-driven latent embedding without prior knowledge — omit the C prior and use GEDI's standard integration workflow instead.
- Input is not raw counts from scRNA-seq modeled with GEDI (e.g., pre-normalized feature tables); the prior-subset-then-QR workflow assumes a shared gene-space count matrix matching the DoRothEA network.
- The regulatory network is unrelated to the measured transcriptome such that after intersecting genes, singular-column removal via QR leaves no usable prior columns.

## Inputs

- Raw single-cell RNA-seq count matrix as dgCMatrix (e.g., PBMC SingleCellExperiment 'counts' assay)
- DoRothEA-derived prior matrix 'C_dorothea_tf_human.refined.rds' restricted to high-confidence TF sets A, B, C
- Sample/donor annotation vector (e.g., donor.method)
- Cell/gene QC-filtered expression data (log10 total counts > 2.5, log10 detected features > 2, mitochondrial percent <= 20; genes with >5 counts in >3 cells)

## Outputs

- Fitted GEDI model with K=40 latent variables and validated convergence tracks
- TF activity matrix per cell (getADB.gedi, activity x cell)
- TF activity x latent-variable matrix (getA.gedi)
- UMAP projections of TF activity (e.g., PAX5, TCF7) saved as figure files

## How to apply

Obtain a DoRothEA-derived regulatory prior matrix C, restricting TF-gene interactions to high-confidence sets A, B, and C (optionally refined with GTEx whole blood), then subset C and the raw count matrix (dgCMatrix) to shared genes in the same order. Remove columns of C that cause singularity via QR decomposition (C <- C[, QR$pivot[seq_len(QR$rank)]]) before model fitting. Fit the GEDI model with the sample vector as auxiliary covariate, K = 40 latent variables, mode = 'Bsphere' (hyperellipsoid restriction), oi_shrinkage = 0.001, randomSeed = 1, and ~500 optimization iterations, checking convergence with model$plotTracking(). Retrieve TF activities with getA.gedi (activity x LVs) and getADB.gedi (activity x cell), reorder cell metadata by model$aux$cellIDs since GEDI reorders cells by sample, and project per-TF activities (e.g., PAX5, TCF7) onto a UMAP with GEDI::plot_embedding. Validation relies on activity patterns being cell-type restricted (PAX5 in B cells, TCF7 in CD4+ T cells) and on significant correlation between projected TF activity and decoded TF mRNA abundance across cells.

## Related tools

- **GEDI** (Fits the generative latent-variable model with the TF prior matrix C as prior knowledge and retrieves TF activities via getA.gedi/getADB.gedi; plot_embedding projects TF activity onto UMAP) — https://github.com/csglab/GEDI
- **DoRothEA** (Source of the human TF-gene regulatory network used to build the prior matrix C (high-confidence sets A, B, C, refined with GTEx whole blood)) — https://saezlab.github.io/dorothea/
- **scuttle** (Performs QC filtering of cells (counts, detected features, mitochondrial percent) before fitting the prior-informed model) — https://csglab.github.io/GEDI_manuscript/
- **scran** (Supports QC and low-expression gene filtering of the PBMC SingleCellExperiment prior to GEDI model setup) — https://csglab.github.io/GEDI_manuscript/
- **uwot** (Computes the 2D UMAP (min_dist = 0.01, euclidean metric) from the svd.gedi embedding onto which TF activities are projected)

## Examples

```
model <- gedi.opt(M=M, C=C, aux=donor.method, K=40, mode='Bsphere', oi_shrinkage=0.001, randomSeed=1); ADB <- getADB.gedi(model, M)
```

## Evaluation signals

- Convergence: model$plotTracking() shows stabilized components (o, Z, A, Bi, Qi, oi, si) after ~500 iterations
- For 74 out of 89 TFs in the network, projected TF activity across cells correlates significantly with decoded mRNA abundance of the TF gene
- Projected activity is cell-type restricted: PAX5 activity localizes to B cells and TCF7 activity to CD4+ T cells on the UMAP
- In the perturbation benchmark, GEDI (hyperellipsoid mode) reaches mean AUROC ~0.86 for distinguishing TF-perturbed cells versus eight decoupleR methods (aucell, gsva, mlm, ora, ulm, viper, wmean, wsum)
- Schema check: getA.gedi returns activity x LVs and getADB.gedi returns activity x cell matrices; metadata must be reordered by model$aux$cellIDs after GEDI reorders cells by sample

## Limitations

- Columns of the prior matrix C that cause singularity must be removed via QR decomposition before fitting; failure to do so breaks model optimization.
- GEDI currently supports only linear manifold learning, with the option to restrict the manifold to a hyperellipsoid; richer nonlinear priors are future extensions.
- GEDI reorders cells by sample, so metadata must be realigned via model$aux$cellIDs before downstream comparisons.

## Evidence

- [abstract] GEDI can incorporate gene-level prior knowledge to infer pathway and regulatory network activities in single cells.: "GEDI can also incorporate gene-level prior knowledge to infer pathway and regulatory network activities in single cells."
- [methods] QR-based removal of singular prior columns: "C <- C[ , QR$pivot[seq_len(QR$rank)] ] # remove any columns that results in singularity"
- [methods] High-confidence TF set restriction: "restricted the TF-gene interactions to the high-confidence sets A, B, and C"
- [methods] DoRothEA network source: "we downloaded the human [DoRothEA](https://saezlab.github.io/dorothea/) gene regulatory network"
