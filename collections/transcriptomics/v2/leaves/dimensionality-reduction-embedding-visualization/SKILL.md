---
name: dimensionality-reduction-embedding-visualization
description: Use when you have multi-sample or multi-condition single-cell count matrices
  (dgCMatrix) needing an interpretable latent embedding, or when projecting prior-knowledge
  gene signatures (DoRothEA TF regulons, pathways) onto per-cell activities for visualization.
when_to_use_negative:
- Your goal is a non-linear manifold embedding beyond the linear (optionally hyperellipsoid-restricted)
  models GEDI supports — the paper notes other parametric manifold approximations
  are future extensions
- Input is already a reduced embedding or feature table rather than raw single-cell
  counts with a matched prior network
- Single-sample, static data where no inter-sample harmonization or multi-condition
  modeling is needed
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GEDI
  role: Core generative model performing the unified latent embedding, integration,
    imputation, and prior-knowledge-based activity inference
  repo: https://github.com/csglab/GEDI
- name: uwot
  role: UMAP embedding used to visualize GEDI TF activities (e.g., PAX5, TCF7) in
    2D via plot_embedding
  repo: https://cran.r-project.org/web/packages/uwot/index.html
- name: scran
  role: QC, low-expressed-gene filtering, and preprocessing of count matrices before
    running GEDI
  repo: https://bioconductor.org/packages/release/bioc/html/scran.html
- name: decoupleR
  role: Baseline TF activity inference methods (aucell, gsva, mlm, ora, ulm, viper,
    wmean, wsum) used to benchmark GEDI activities
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:228555b576b6fc7d7a9e65297f3f3d5d5da9be2b4adf24eb831e8fef9a811dd6
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
  - uwot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# dimensionality-reduction-embedding-visualization

## Summary

Build interpretable latent embeddings of multi-sample, multi-condition single-cell data with GEDI, a unified Bayesian generative model that integrates normalization, denoising, cluster-free DGE, and prior-knowledge-based pathway/GRN activity inference, then visualize TF or pathway activities on UMAP embeddings.

## When to use

Use when you have multi-sample or multi-condition single-cell count matrices (dgCMatrix) needing an interpretable latent embedding, or when projecting prior-knowledge gene signatures (DoRothEA TF regulons, pathways) onto per-cell activities for visualization.

## When NOT to use

- Your goal is a non-linear manifold embedding beyond the linear (optionally hyperellipsoid-restricted) models GEDI supports — the paper notes other parametric manifold approximations are future extensions
- Input is already a reduced embedding or feature table rather than raw single-cell counts with a matched prior network
- Single-sample, static data where no inter-sample harmonization or multi-condition modeling is needed

## Inputs

- Raw single-cell RNA-seq count matrix (dgCMatrix), cells x genes
- Prior-knowledge C matrix of TF-gene or gene-set interactions (e.g., weighted DoRothEA sets A/B/C refined with GTEx whole blood)
- Sample/condition metadata for multi-sample integration
- Per-TF or per-gene-set activity matrices from getA.gedi / getADB.gedi for visualization

## Outputs

- GEDI model object with latent vectors (sample-level) and cell-level representations
- Activity x latent-variable matrix (getA.gedi) and activity x cell matrix (getADB.gedi)
- UMAP embeddings colored by inferred TF activities (e.g., PAX5, TCF7) as per-TF figure files
- Per-TF sample parameters/coefficients for cluster-free DGE and vector-field analyses

## How to apply

Perform QC with scuttle/scran (e.g., log10_total_counts > 2.5, log10_total_features_by_counts > 2, subsets_Mito_percent <= 20; genes kept if rowSums(counts>3) > 3), then supply raw counts as a dgCMatrix to GEDI. Build the prior C matrix (e.g., weighted DoRothEA sets A, B, C refined with GTEx whole blood), restrict to genes shared with the expression matrix, and remove singular columns via QR decomposition with C <- C[, QR$pivot[seq_len(QR$rank)]]. Run GEDI with linear or hyperellipsoid ('Bsphere') manifold learning (e.g., K=40 latent vectors, oi_shrinkage=0.001, 500 iterations), then retrieve activities with getA.gedi (activity x latent variables) and getADB.gedi (activity x cell). Embed cells (e.g., UMAP via uwot) and overlay per-TF activities such as PAX5 or TCF7 with plot_embedding to produce per-TF activity figures; TF activities can then be benchmarked against perturbation labels (e.g., AUROC vs decoupleR methods like ulm, viper, aucell).

## Related tools

- **GEDI** (Core generative model performing the unified latent embedding, integration, imputation, and prior-knowledge-based activity inference) — https://github.com/csglab/GEDI
- **uwot** (UMAP embedding used to visualize GEDI TF activities (e.g., PAX5, TCF7) in 2D via plot_embedding) — https://cran.r-project.org/web/packages/uwot/index.html
- **scran** (QC, low-expressed-gene filtering, and preprocessing of count matrices before running GEDI) — https://bioconductor.org/packages/release/bioc/html/scran.html
- **decoupleR** (Baseline TF activity inference methods (aucell, gsva, mlm, ora, ulm, viper, wmean, wsum) used to benchmark GEDI activities)

## Examples

```
devtools::install_github("csglab/GEDI"); model <- gedi(counts = counts_dgCMatrix, C = C, K = 40, mode = 'Bsphere', oi_shrinkage = 0.001, iterations = 500)
```

## Evaluation signals

- The prior C matrix, restricted to shared genes, must contain no singular columns after QR pivoting (QR$rank defines retained columns)
- Inferred TF activities should discriminate perturbation status — GEDI reached mean AUROC 0.86 distinguishing TF-perturbed cells in the benchmark, comparable to or better than eight decoupleR methods
- Reproduced manuscript notebooks (e.g., pbmc_tf_activity.html) should regenerate Fig. 5 TF-activity projections onto UMAP embeddings
- Pre-filtered counts must match QC thresholds (log10_total_counts > 2.5, log10_total_features_by_counts > 2, subsets_Mito_percent <= 20; genes with rowSums(counts>3) > 3)
- Published GEDI models for the tutorial are downloadable from Zenodo (e.g., doi 10.5281/zenodo.8222040) for output validation

## Limitations

- GEDI currently supports linear manifold learning, optionally restricted to a hyperellipsoid; other parametric manifold approximations are future extensions
- GEDI's imputation performance versus dedicated single-cell imputation methods remains untested
- Counterfactual prediction in unobserved cell types (e.g., perturbation response via the vector field) remains untested
- The benchmark shows GEDI performs best for TFs whose perturbation shifts cells along main axes of variation; results require more extensive analyses of factors like DE magnitude and sequencing depth

## Evidence

- [abstract] We introduce GEDI, a generative model that identifies latent space variations in multi-sample, multi-condition single-cell datasets: "We introduce GEDI, a generative model that identifies latent space variations in multi-sample, multi-condition single-cell datasets"
- [intro] GEDI incorporates various single-cell analysis steps within a unified Bayesian framework that includes data integration across samples/conditions, data imputation and denoising, cluster-free DGE: "GEDI incorporates various single-cell analysis steps within a unified Bayesian framework that includes data integration across samples/conditions, data imputation and denoising, cluster-free DGE"
- [methods] C <- C[ , QR$pivot[seq_len(QR$rank)] ] # remove any columns that results in singularity: "C <- C[ , QR$pivot[seq_len(QR$rank)] ] # remove any columns that results in singularity"
- [discussion] GEDI currently supports linear manifold learning, with the option to further restrict the manifold to a hyperellipsoid: "GEDI currently supports linear manifold learning, with the option to further restrict the manifold to a hyperellipsoid"
- [readme] A generative model that unifies integration, cluster-free differential expression, pathway and regulatory network analysis, data normalization and imputation.: [evidence span withheld — anchor status: external_doc]
