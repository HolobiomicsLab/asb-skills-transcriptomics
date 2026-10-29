---
name: single-cell-batch-integration-topic-modeling
description: Use when a perturbation single-cell atlas spans multiple sequencing batches
  with cell states batch-confounded (e.g., wild-type vs Tal1-/- chimeric mouse embryo
  batches) and PCA-based latent spaces fail to mix batches while preserving cell-type
  separation.
when_to_use_negative:
- The dataset is single-batch or cell states are not batch-confounded, so a standard
  scanpy PCA latent space suffices without technical disentanglement.
- Detected batch-confounded or perturbation-induced cell states cannot be validated
  with orthogonal biological evidence, since the authors caution these may reflect
  under-correction of technical effects rather than true biology.
- Input is already a normalized/feature-extracted matrix rather than raw counts —
  CODAL's count-based topic model expects raw count data (transformed via deviance
  residuals for the encoder).
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0099
tools:
- name: CODAL
  role: Variational autoencoder topic model with mutual information regularization
    that disentangles technical covariates (batch, chimeric status, embryo mixing,
    collection day) from biological topics to build the batch-corrected latent space
  repo: https://mira-multiome.readthedocs.io
- name: MIRA
  role: Parent Python package whose architecture CODAL extends; provides the topic-modeling
    inference used for the batched latent space
  repo: https://github.com/cistrome/MIRA
- name: scanpy
  role: Standard preprocessing workflow (concatenate batches, cell/gene count filtering
    at 400/30, normalize_total target_sum 10000, log1p, HVG selection) and PCA baseline
    for comparison
- name: Leiden algorithm
  role: Clustering of the integrated dataset to isolate the HE-prog cluster and high-resolution
    subclustering of Tom+ Tal1-/- cells into subclusters A-E
- name: UMAP-learn
  role: Embedding of the ILR-transformed CODAL topic compositions (min_dist 0.1, negative_sample_rate
    3) for visual assessment of batch mixing and cell-type separation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:2acd169c2f33ed507561dc713e9ab8f279328a286ee9fe412247176b329b7f70
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CODAL
  - Leiden algorithm
  - MIRA
  - UMAP-learn
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# single-cell-batch-integration-topic-modeling

## Summary

Disentangle technical batch effects from biological cell states in multi-batch single-cell RNA-seq atlases using CODAL, a variational autoencoder topic model extending MIRA with mutual information regularization, then cluster cells in the ILR-transformed topic-composition latent space to detect batch-confounded or perturbation-induced cell states.

## When to use

Use when a perturbation single-cell atlas spans multiple sequencing batches with cell states batch-confounded (e.g., wild-type vs Tal1-/- chimeric mouse embryo batches) and PCA-based latent spaces fail to mix batches while preserving cell-type separation.

## When NOT to use

- The dataset is single-batch or cell states are not batch-confounded, so a standard scanpy PCA latent space suffices without technical disentanglement.
- Detected batch-confounded or perturbation-induced cell states cannot be validated with orthogonal biological evidence, since the authors caution these may reflect under-correction of technical effects rather than true biology.
- Input is already a normalized/feature-extracted matrix rather than raw counts — CODAL's count-based topic model expects raw count data (transformed via deviance residuals for the encoder).

## Inputs

- Multi-batch single-cell RNA-seq count matrix (AnnData/h5ad), e.g., concatenated mouse gastrulation atlas batches (Pijuan-Sala et al., E6.5-E8.5, wild-type + Tal1-/- chimeric)
- Categorical technical covariate annotations per cell (sequencing batch, chimeric status, embryo mixing, collection day)
- Hyperparameter search specification (e.g., N_topics range 35-80, number of tuning iterations)

## Outputs

- Fitted CODAL topic model (e.g., 63 topics) with batch-corrected latent topic compositions per cell
- ILR-transformed topic-composition latent space with k-NN graph and UMAP embedding (min_dist 0.1, negative_sample_rate 3)
- Leiden clusters of the integrated dataset and high-resolution subclusters of the target population (HE-prog subclusters A-E)
- Marker-expression tabulation per subcluster (e.g., fig6_heprog_marker_expression_table.csv with Emcn, Hoxb9/Hoxc8/Cdx4, Kdr/Sox17/Esam)

## How to apply

Preprocess concatenated batches with the scanpy standard workflow: filter cells with fewer than 400 counts and genes with fewer than 30 counts, run normalize_total with target_sum=10000, log1p-transform, and select highly variable genes with a minimum dispersion threshold (0.3 in the paper's methods; the task card uses 0.5). Define categorical technical covariates (sequencing batch, chimeric status, embryo mixing, collection day) and run Bayesian hyperparameter tuning (Gaussian Process with Matern 5/2 kernel, e.g., 60 iterations over N_topics 35-80, with rung-based trial pruning at the 8th and 16th epochs); the Pijuan-Sala re-analysis retained a 63-topic model. Fit the CODAL topic model via cyclical stochastic minibatch gradient ascent with MINE-based mutual information regularization (1-Lipschitz constraint via spectral normalization), transform the learned topic compositions with isometric log-ratio (ILR), build a k-NN graph using Manhattan distance, and compute UMAPs (min_dist 0.1, negative_sample_rate 3). Compare batch mixing and cell-type discrimination against the scanpy PCA baseline; then run Leiden clustering, selecting the HE-prog cluster, and re-run Leiden at high resolution on the perturbed (Tom+ Tal1-/-) subset to resolve subclusters such as A-E, which can be annotated via marker genes (e.g., Emcn, Hoxb9/Hoxc8/Cdx4, Kdr/Sox17/Esam) and shared topic compositions.

## Related tools

- **CODAL** (Variational autoencoder topic model with mutual information regularization that disentangles technical covariates (batch, chimeric status, embryo mixing, collection day) from biological topics to build the batch-corrected latent space) — https://mira-multiome.readthedocs.io
- **MIRA** (Parent Python package whose architecture CODAL extends; provides the topic-modeling inference used for the batched latent space) — https://github.com/cistrome/MIRA
- **scanpy** (Standard preprocessing workflow (concatenate batches, cell/gene count filtering at 400/30, normalize_total target_sum 10000, log1p, HVG selection) and PCA baseline for comparison)
- **Leiden algorithm** (Clustering of the integrated dataset to isolate the HE-prog cluster and high-resolution subclustering of Tom+ Tal1-/- cells into subclusters A-E)
- **UMAP-learn** (Embedding of the ILR-transformed CODAL topic compositions (min_dist 0.1, negative_sample_rate 3) for visual assessment of batch mixing and cell-type separation)

## Examples

```
conda create --name mira-env -c conda-forge -c pytorch -c bioconda scanpy jupyter leidenalg && pip3 install mira-multiome
```

## Evaluation signals

- The CODAL UMAP shows batch intermixing and cell-type discrimination comparable to or better than the scanpy PCA baseline.
- Hyperparameter tuning converges on a stable N_topics (the re-analysis kept a 63-topic model after 60 Bayesian iterations over 35-80).
- Each high-resolution Leiden subcluster (A-E) of Tom+ Tal1-/- HE-prog cells has a marker-expression profile consistent with the mesodermal cell-type annotations reported in Pijuan-Sala et al.
- Subcluster identities are corroborated by shared topic latent variable composition, not marker genes alone.
- Perturbation-induced subclusters are supported by orthogonal biological evidence, ruling out under-correction of technical effects.

## Limitations

- The authors caution that new batch-confounded or perturbation-induced cell states should be validated using orthogonal biological evidence to rule out under-correction of technical effects.
- CODAL does not remove the need for good experimental design: experimentalists must include appropriate controls, e.g., a control group of cells unaffected by the perturbation present in both treatment and control batches.
- Further improvements would require better characterization of technical sources of bias, an acknowledged open limitation.
- Software version numbers and computational implementation details are not reported in the paper's discussion.

## Evidence

- [abstract] CODAL, a variational autoencoder-based statistical model which uses a mutual information regularization technique to explicitly disentangle factors related to technical and biological: "Here we propose CODAL, a variational autoencoder-based statistical model which uses a mutual information regularization technique to explicitly disentangle factors related to technical and biological"
- [methods] cell/gene count filtering and normalization thresholds: "cells with fewer than 400 counts and genes with fewer than 30 counts filtered out"
- [methods] ILR-transformed k-NN graph construction: "We then calculate distances between transformed cell latent variables using the Manhattan distance to create a k-nearest neighbors (k-NN) graph"
- [methods] Bayesian hyperparameter optimization procedure: "We use a Gaussian Process model with a Matern kernel68 to predict the distribution of scores given hyperparameters."
- [methods] HE-prog high-resolution subclustering: "ran the Leiden algorithm to obtain high-resolution subclusters"
- [discussion] orthogonal validation caution: "we caution that the existence of new batch-confounded or perturbation-induced cell states should be validated using orthogonal biological evidence to rule out the under-correction of technical effects"
- [intro] CODAL extends MIRA architecture: "The CODAL model architecture extends MIRA28, our earlier method for variational topic modeling of single-cell RNA-seq and ATAC-seq data."
- [readme] installation route from README: "pip3 install mira-multiome"
