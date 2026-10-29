---
name: tf-activity-inference-from-scrna-seq
description: Use when raw scRNA-seq counts (dgCMatrix) are available together with
  a TF-gene regulatory prior (e.g., DoRothEA sets A/B/C) and per-TF activity scores
  per cell are needed, or when TF activities must be validated against TF perturbation
  labels.
when_to_use_negative:
- Only a pre-computed feature/expression table without raw counts and without a TF-gene
  regulatory prior is available — GEDI's knowledge-informed C-matrix formulation requires
  both
- The analysis targets nonlinear manifold structure — GEDI currently supports only
  linear manifold learning, with optional hyperellipsoid restriction
- The goal is differential expression between defined clusters rather than continuous,
  sample-aware TF activity inference along cell states
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0780
tools:
- name: GEDI
  role: Generative model that constrains latent variables with the DoRothEA prior
    C matrix (mode='Bsphere') to infer per-cell TF activities via getA.gedi/getADB.gedi
  repo: https://github.com/csglab/GEDI
- name: decoupleR
  role: Provides the eight baseline TF activity inference methods (aucell, gsva, mlm,
    ora, ulm, viper, wmean, wsum) benchmarked against GEDI on the same regulatory
    network
- name: scuttle
  role: Performs basic QC and filtering of low-quality cells before GEDI model fitting
- name: scran
  role: Used with scuttle for cell QC and low-expression gene filtering of the count
    matrix
- name: R
  role: Analysis language for the full workflow (QC, prior construction, GEDI fitting,
    decoupleR benchmark, plotting)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:84b6ea621b3d5158dc888305e3018af7cd95c69520072d8865c276072e73a02a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - R
  - decoupleR
  - scran
  - scuttle
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tf-activity-inference-from-scrna-seq

## Summary

Infers transcription factor (TF) activities in single cells by embedding a weighted DoRothEA regulatory prior matrix as a knowledge-informed constraint (C matrix) in the GEDI generative model, and benchmarks the resulting activities against eight decoupleR methods using AUROC against TF perturbation labels.

## When to use

Use when raw scRNA-seq counts (dgCMatrix) are available together with a TF-gene regulatory prior (e.g., DoRothEA sets A/B/C) and per-TF activity scores per cell are needed, or when TF activities must be validated against TF perturbation labels.

## When NOT to use

- Only a pre-computed feature/expression table without raw counts and without a TF-gene regulatory prior is available — GEDI's knowledge-informed C-matrix formulation requires both
- The analysis targets nonlinear manifold structure — GEDI currently supports only linear manifold learning, with optional hyperellipsoid restriction
- The goal is differential expression between defined clusters rather than continuous, sample-aware TF activity inference along cell states

## Inputs

- Single-cell raw counts as dgCMatrix (scRNA-seq), split into discovery and evaluation batches
- Weighted DoRothEA regulatory prior matrix (high-confidence TF-gene sets A, B, C, refined with GTEx whole blood)
- Per-cell TF perturbation labels for AUROC benchmarking

## Outputs

- GEDI TF activity matrix (activity x cell) from getADB.gedi
- Activity x latent variable matrix from getA.gedi
- Per-method mean-AUROC benchmark table comparing GEDI to eight decoupleR methods
- Per-TF activity UMAP projection figures (e.g., PAX5, TCF7) via plot_embedding

## How to apply

Perform scuttle/scran QC (filter cells on log10_total_counts > 2.5, log10_total_features_by_counts > 2, subsets_Mito_percent <= 20; keep genes with sum_genes > 3) and split data into a discovery batch and a held-out evaluation batch. Build the weighted DoRothEA regulatory prior restricted to high-confidence sets A, B, and C (refined with GTEx whole blood), restrict it to genes shared with the expression matrix, and remove columns causing singularity via QR decomposition (C <- C[, QR$pivot[seq_len(QR$rank)]]). Fit a GEDI model on raw counts as dgCMatrix with K=40 latent variables, mode='Bsphere' (hyperellipsoid restriction), oi_shrinkage=0.001, and 500 iterations, then retrieve TF activities via getA.gedi (activity x LVs) and getADB.gedi (activity x cell). For benchmarking, estimate TF activities on the held-out batch with the eight decoupleR methods (aucell, gsva, mlm, ora, ulm, viper, wmean, wsum) using the same regulatory network, compute per-TF AUROC against perturbation labels, and compare mean AUROC (GEDI hyperellipsoid achieved 0.86, among top performers). Project TF activities (e.g., PAX5, TCF7) onto UMAP embeddings with plot_embedding for visualization.

## Related tools

- **GEDI** (Generative model that constrains latent variables with the DoRothEA prior C matrix (mode='Bsphere') to infer per-cell TF activities via getA.gedi/getADB.gedi) — https://github.com/csglab/GEDI
- **decoupleR** (Provides the eight baseline TF activity inference methods (aucell, gsva, mlm, ora, ulm, viper, wmean, wsum) benchmarked against GEDI on the same regulatory network)
- **scuttle** (Performs basic QC and filtering of low-quality cells before GEDI model fitting)
- **scran** (Used with scuttle for cell QC and low-expression gene filtering of the count matrix)
- **R** (Analysis language for the full workflow (QC, prior construction, GEDI fitting, decoupleR benchmark, plotting))

## Examples

```
C <- C[, QR$pivot[seq_len(QR$rank)]]; gedi_model <- GEDI(counts = counts_dgCMatrix, C = C, K = 40, mode = 'Bsphere', oi_shrinkage = 0.001, nrun = 500); activities <- getADB.gedi(gedi_model)
```

## Evaluation signals

- Benchmark GEDI's mean AUROC for distinguishing TF-perturbed cells from other cells; a correctly applied run should place GEDI among the top performers with mean AUROC ≈ 0.86 versus the eight decoupleR methods
- The QR rank filter must remove singular prior columns (C has only QR$rank columns remaining); fitting with singular columns indicates an error
- Output dimensions should match expectations: getADB.gedi returns TF-activity x cell, getA.gedi returns activity x K latent variables (K=40)
- Per-TF activity projections (e.g., PAX5, TCF7) on UMAP should show coherent spatial activity patterns across the embedding

## Limitations

- GEDI supports only linear manifold learning (with optional hyperellipsoid/Bsphere restriction); other parametric manifold approximations are future extensions
- A GEDI model trained on the discovery batch may not cover TFs whose perturbation does not shift cell state along the main axes of variation, reducing benchmark AUROC
- Counterfactual perturbation response prediction via GEDI's vector field remains untested

## Evidence

- [abstract] GEDI can incorporate gene-level prior knowledge to infer pathway and regulatory network activities in single cells.: "GEDI can also incorporate gene-level prior knowledge to infer pathway and regulatory network activities in single cells."
- [methods] DoRothEA high-confidence sets and GTEx whole blood refinement of the regulatory network.: "using the whole blood data from GTEx and generated a weighted regulatory network"
- [methods] QR decomposition step to drop singular prior columns before GEDI fitting.: "C <- C[ , QR$pivot[seq_len(QR$rank)] ] # remove any columns that results in singularity"
- [methods] Restriction of TF-gene interactions to confidence sets A, B, and C.: "restricted the TF-gene interactions to the high-confidence sets A, B, and C"
- [readme] README overview of GEDI's unified scope including regulatory network analysis.: [evidence span withheld — anchor status: external_doc]
