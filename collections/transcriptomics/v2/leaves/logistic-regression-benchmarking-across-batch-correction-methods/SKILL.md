---
name: logistic-regression-benchmarking-across-batch-correction-methods
description: 'Use when comparing batch-correction methods on scRNA-seq data with multilevel
  batch variables: train each method on the same counts and labels, then evaluate
  whether corrected embeddings preserve cell-type identity via supervised classification
  accuracy.'
when_to_use_negative:
- The dataset has only a single batch variable and no cell-type labels to supervise
  the classification benchmark — multi-level batch handling and label-based accuracy
  scoring cannot be exercised
- The goal is de novo cell-type discovery or trajectory inference rather than comparing
  batch-correction quality on known labels
- The input is raw reads or a feature table rather than a cell-by-gene UMI count matrix
  in the format the methods expect
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scPhere
  role: Deep generative model producing the spherical (vMF) batch-corrected embedding
    being benchmarked; trained with multilevel batch vectors n_batch=[30, 3, 2]
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: Harmony
  role: Comparator batch-correction method whose embedding is classified with the
    same k-NN and logistic-regression classifiers
- name: LIGER
  role: Comparator batch-correction method (integrative NMF) evaluated with the same
    classification benchmark
- name: Seurat3 CCA
  role: Comparator integration method (canonical correlation analysis) evaluated with
    the same classification benchmark
- name: TensorFlow
  role: Backend (v1.14.0 with tensorflow-probability v0.7.0) used to train the scPhere
    VAE
  repo: https://github.com/klarman-cell-observatory/scPhere
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:6114740d4de1d7f469d9ae419dc87d13b02dfb9e63c796898f8779d474cfc7d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - Harmony
  - LIGER
  - Seurat3 CCA
  - TensorFlow
  - scPhere
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# logistic-regression-benchmarking-across-batch-correction-methods

## Summary

A benchmarking protocol that scores cell-type classification accuracy (logistic regression and k-NN classifiers on batch-corrected embeddings) to compare scPhere against Harmony, Seurat3 CCA, and LIGER on the ulcerative colitis colon dataset with multilevel batch factors (30 patients, 3 health statuses, 2 locations).

## When to use

Use when comparing batch-correction methods on scRNA-seq data with multilevel batch variables: train each method on the same counts and labels, then evaluate whether corrected embeddings preserve cell-type identity via supervised classification accuracy.

## When NOT to use

- The dataset has only a single batch variable and no cell-type labels to supervise the classification benchmark — multi-level batch handling and label-based accuracy scoring cannot be exercised
- The goal is de novo cell-type discovery or trajectory inference rather than comparing batch-correction quality on known labels
- The input is raw reads or a feature table rather than a cell-by-gene UMI count matrix in the format the methods expect

## Inputs

- Sparse UMI count matrix uc_epi.mtx from Single Cell Portal SCP551 (colon mucosa epithelial cells)
- Multilevel batch annotations: uc_epi_batch_patient.tsv (30 patients), uc_epi_batch_health.tsv (3 statuses), uc_epi_batch_location.tsv (2 locations), 0-based
- Cell-type labels uc_epi_celltype.tsv
- Batch-corrected embeddings from each method being compared (scPhere latent space, Harmony, LIGER, Seurat3 CCA)

## Outputs

- Trained scPhere model uc_epi_model_250epoch
- Latent embedding uc_epi_latent_250epoch.tsv (one row per cell, z_dim=2 on the unit sphere)
- Per-cell log-likelihoods uc_epi_ll_250epoch.tsv
- Training trace uc_epi_ll_250epoch_train.png (log-likelihood and KL-divergence per iteration)
- Per-method k-NN and logistic-regression cell-type classification accuracies for comparison (Fig. 3d-f)

## How to apply

Download the colon mucosa epithelial data from Single Cell Portal study SCP551 (uc_epi.mtx, uc_epi_batch_patient.tsv, uc_epi_batch_health.tsv, uc_epi_batch_location.tsv, uc_epi_celltype.tsv) and load the sparse matrix with scPhere's read_mtx. Build a SCPHERE model with n_gene equal to the gene count, multilevel batch specification n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', observation_dist='nb', batch_invariant=False, seed=0, and train with the Trainer (max_epoch=250, mb_size=128, learning_rate=0.001). Save the model (uc_epi_model_250epoch), embed all cells via model.encode into uc_epi_latent_250epoch.tsv, and compute per-cell log-likelihoods. Generate reference embeddings for the same cells with Harmony, LIGER, and Seurat3 CCA, then for each embedding train k-NN and logistic-regression classifiers on the deposited cell-type labels, reproducing the comparison in Fig. 3d-f and Supplementary Figs. 6-9. The rationale is that a batch correction that removes technical/biological batch variation while preserving biology should allow classifiers to recover cell types more accurately; scPhere's accuracies should meet or exceed the three comparators.

## Related tools

- **scPhere** (Deep generative model producing the spherical (vMF) batch-corrected embedding being benchmarked; trained with multilevel batch vectors n_batch=[30, 3, 2]) — https://github.com/klarman-cell-observatory/scPhere
- **Harmony** (Comparator batch-correction method whose embedding is classified with the same k-NN and logistic-regression classifiers)
- **LIGER** (Comparator batch-correction method (integrative NMF) evaluated with the same classification benchmark)
- **Seurat3 CCA** (Comparator integration method (canonical correlation analysis) evaluated with the same classification benchmark)
- **TensorFlow** (Backend (v1.14.0 with tensorflow-probability v0.7.0) used to train the scPhere VAE) — https://github.com/klarman-cell-observatory/scPhere

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0); trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001); trainer.train(); z_mean = model.encode(x, batch)
```

## Evaluation signals

- All four scPhere output files exist: uc_epi_model_250epoch, uc_epi_latent_250epoch.tsv, uc_epi_ll_250epoch.tsv, and the training trace png
- scPhere's k-NN and logistic-regression cell-type classification accuracies are at least as high as those of Harmony, Seurat3 CCA, and LIGER, consistent with Fig. 3d-f
- The embedding is 2-dimensional (z_dim=2) with points on the unit hypersphere (latent_dist='vmf') and is visualizable in 3D via the R PlotSphere function with cell-type colors
- The training trace shows log-likelihood and KL-divergence over 250 epochs converging without divergence, using mb_size=128 and learning_rate=0.001
- Batch vectors are 0-based and match n_batch=[30, 3, 2] (30 patients, 3 health statuses, 2 locations); the accuracy result reproduces across stromal, epithelial, and immune cell compartments (Supplementary Figs. 6-9)

## Limitations

- The benchmark was performed on batch_invariant=False models; batch-invariant scPhere (needed to map new, unseen patients) is a separate configuration and requires the major batch vector assumption
- scPhere uses TensorFlow v1.14.0 and tensorflow-probability v0.7.0, tested on Python 3.6.7 — newer TensorFlow versions may be incompatible
- Efficient k-NN and other processing tools for data in hyperbolic space ('wn' latent_dist) are still lacking, so hyperbolic embeddings are harder to benchmark the same way
- Downloading SCP551 data requires a Google account login to Google Cloud

## Evidence

- [discussion] scPhere performed better than state-of-the-art batch-correction methods such as Harmony, Seurat3 CCA, LIGER: "scPhere performed better than state-of-the-art batch-correction methods such as Harmony, Seurat3 CCA, LIGER"
- [discussion] in our benchmarking with IBD cells with 30 patients, three disease statuses, and two spatial locations: "in our benchmarking with IBD cells with 30 patients, three disease statuses, and two spatial locations"
- [methods] model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0): "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)"
- [methods] trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001): "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001)"
- [readme] The batch vector should be 0-based: "The batch vector should be 0-based"
- [intro] current applications of VAEs for scRNA-seq data can only handle a single-batch vector (factor), whereas biologically relevant datasets typically have multiple such factors: "current applications of VAEs for scRNA-seq data can only handle a single-batch vector (factor), whereas biologically relevant datasets typically have multiple such factors"
