---
name: perturbation-response-auroc-benchmarking
description: Use when you have a single-cell TF perturbation dataset with true perturbation
  labels and a TF regulatory prior (e.g., DoRothEA), and need to quantify and compare
  how well activity-inference methods recover perturbation status via AUROC.
when_to_use_negative:
- The input lacks ground-truth TF perturbation labels, since AUROC against perturbation
  status cannot be computed
- The goal is standard sample integration or cluster-free differential expression
  rather than TF perturbation benchmarking
- No TF-gene regulatory prior (e.g., DoRothEA) is available for the organism or tissue
  in question
tools:
- name: GEDI
  role: Fit the prior-informed generative model (K=40, mode='Bsphere', oi_shrinkage=0.001,
    500 iterations) and extract TF activity scores with getA.gedi/getADB.gedi
  repo: https://github.com/csglab/GEDI/
- name: decoupleR
  role: Compute baseline TF activity estimates on the evaluation batch with aucell,
    gsva, mlm, ora, ulm, viper, wmean, and wsum using the same regulatory network
- name: R
  role: Implementation language for the full benchmark workflow, with Rcpp/RcppEigen
    dependencies for GEDI
  repo: https://github.com/csglab/GEDI/
- name: scuttle
  role: Quality control of cells and low-expression gene filtering before model fitting
- name: scran
  role: Single-cell preprocessing/QC alongside scuttle in the discovery/evaluation
    split
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:f89156240c5500f07d886b5ed04e15c5bc98da968fd9638904b50703c66b5e52
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# perturbation-response-auroc-benchmarking

## Summary

Benchmark TF perturbation response prediction by comparing GEDI-inferred TF activities (hyperellipsoid mode) against eight decoupleR activity-inference methods, scoring mean AUROC of perturbed vs. non-perturbed cells on a held-out single-cell batch.

## When to use

Use when you have a single-cell TF perturbation dataset with true perturbation labels and a TF regulatory prior (e.g., DoRothEA), and need to quantify and compare how well activity-inference methods recover perturbation status via AUROC.

## When NOT to use

- The input lacks ground-truth TF perturbation labels, since AUROC against perturbation status cannot be computed
- The goal is standard sample integration or cluster-free differential expression rather than TF perturbation benchmarking
- No TF-gene regulatory prior (e.g., DoRothEA) is available for the organism or tissue in question

## Inputs

- Single-cell TF perturbation count matrix (dgCMatrix), split into discovery and held-out evaluation batches
- True per-cell TF perturbation labels for the evaluation batch
- Weighted DoRothEA regulatory prior matrix (high-confidence sets A, B, C, refined with GTEx whole blood)

## Outputs

- Per-method mean-AUROC benchmark table (GEDI vs. aucell, gsva, mlm, ora, ulm, viper, wmean, wsum)
- Per-TF activity scores (activity x cell) from GEDI getADB.gedi and decoupleR methods
- Per-TF UMAP activity projection figures (e.g., PAX5, TCF7) via plot_embedding

## How to apply

Split the single-cell TF perturbation count matrix into a discovery batch (from which per-TF gene signatures are learned) and a held-out evaluation batch, applying scuttle/scran QC and low-expression gene filtering to both. Build the weighted DoRothEA regulatory prior restricted to high-confidence sets A, B, and C (refined with GTEx whole blood), intersect it with the genes in the expression matrix, and drop singular columns via QR decomposition (C <- C[, QR$pivot[seq_len(QR$rank)]]). Fit a GEDI model on the discovery batch (raw counts as dgCMatrix, C prior, K=40, mode='Bsphere' hyperellipsoid, oi_shrinkage=0.001, 500 iterations) and retrieve TF activities with getA.gedi and getADB.gedi. Estimate TF activities on the evaluation batch with the eight decoupleR methods (aucell, gsva, mlm, ora, ulm, viper, wmean, wsum) using the identical regulatory network, then compute per-TF AUROC against true perturbation labels and average per method to assemble the benchmark table.

## Related tools

- **GEDI** (Fit the prior-informed generative model (K=40, mode='Bsphere', oi_shrinkage=0.001, 500 iterations) and extract TF activity scores with getA.gedi/getADB.gedi) — https://github.com/csglab/GEDI/
- **decoupleR** (Compute baseline TF activity estimates on the evaluation batch with aucell, gsva, mlm, ora, ulm, viper, wmean, and wsum using the same regulatory network)
- **R** (Implementation language for the full benchmark workflow, with Rcpp/RcppEigen dependencies for GEDI) — https://github.com/csglab/GEDI/
- **scuttle** (Quality control of cells and low-expression gene filtering before model fitting)
- **scran** (Single-cell preprocessing/QC alongside scuttle in the discovery/evaluation split)

## Examples

```
C <- C[ , QR$pivot[seq_len(QR$rank)] ]  # drop singular columns from the DoRothEA C prior before fitting GEDI (K=40, mode='Bsphere', oi_shrinkage=0.001)
```

## Evaluation signals

- GEDI with mode='Bsphere' achieves a mean AUROC of ~0.86 for distinguishing TF-perturbed cells from other cells, ranking among the top performers versus the eight decoupleR methods
- C prior matrix contains no singular columns after QR filtering (QR$rank matches the retained column count)
- Discovery and evaluation batches are disjoint; TF activities are computed on the held-out batch only using the same regulatory network across methods
- AUROC per TF is computed against binary true perturbation labels and averaged per method to fill the benchmark table
- Projected TF activities (e.g., PAX5, TCF7) render successfully on UMAP embeddings via plot_embedding

## Limitations

- Only TFs whose perturbation shifts cell state along main axes of variation are well captured; perturbations orthogonal to the manifold may be missed
- GEDI currently supports only linear manifold learning (with optional hyperellipsoid restriction), limiting nonlinear perturbation geometries
- Counterfactual prediction of perturbation responses in unobserved cell types via GEDI's vector field remains untested

## Evidence

- [abstract] GEDI incorporates prior knowledge into activity inference: "GEDI can also incorporate gene-level prior knowledge to infer pathway and regulatory network activities in single cells."
- [methods] Singular column removal from the C prior matrix: "C <- C[ , QR$pivot[seq_len(QR$rank)] ] # remove any columns that results in singularity"
- [methods] DoRothEA high-confidence restriction: "restricted the TF-gene interactions to the high-confidence sets A, B, and C"
- [discussion] Projecting activities onto cells: "gene set activities can be projected onto individual cells, thus providing a natural approach to perform single-cell gene set enrichment analysis"
- [readme] Manuscript notebook for TF activity analysis: [evidence span withheld — anchor status: external_doc]
