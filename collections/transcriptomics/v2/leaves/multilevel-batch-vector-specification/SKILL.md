---
name: multilevel-batch-vector-specification
description: Use when scRNA-seq data have several batch covariates at once (e.g.,
  30 patients, 3 disease statuses, 2 locations) and single-factor methods like Harmony,
  LIGER, or Seurat3 CCA are insufficient.
when_to_use_negative:
- The dataset has only a single batch factor or no batch vectors — use n_batch=0 with
  an artificial all -1 batch vector instead of a multi-level specification
- The goal is mapping new, unseen patients without their batch annotations — set batch_invariant=True
  rather than batch_invariant=False
- Latent structure is hierarchical or branched developmental trajectory, where the
  hyperbolic 'wn' latent_dist is preferable to the spherical 'vmf' setting
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scPhere
  role: Provides the SCPHERE model class whose n_batch parameter accepts a list of
    batch counts per factor, plus Trainer, encode, and get_log_likelihood utilities
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend dependency (v1.14.0 with tensorflow-probability v0.7.0) for training
    the multi-level VAE
- name: Harmony
  role: Single-batch-factor comparator used to benchmark the multi-level correction
    quality
- name: LIGER
  role: Single-batch-factor comparator used to benchmark the multi-level correction
    quality
- name: Seurat3 CCA
  role: Single-batch-factor comparator used to benchmark the multi-level correction
    quality
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:504da393956d30f65cdc73d3c152a359ac82b277f0f3f7b2f4bb96ed7414cc08
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# multilevel-batch-vector-specification

## Summary

Specifies multiple simultaneous technical and biological batch factors (e.g., patient, health status, tissue location) in a scPhere VAE by passing a list of batch counts to n_batch, enabling batch correction for datasets where existing methods handle only a single batch variable.

## When to use

Use when scRNA-seq data have several batch covariates at once (e.g., 30 patients, 3 disease statuses, 2 locations) and single-factor methods like Harmony, LIGER, or Seurat3 CCA are insufficient.

## When NOT to use

- The dataset has only a single batch factor or no batch vectors — use n_batch=0 with an artificial all -1 batch vector instead of a multi-level specification
- The goal is mapping new, unseen patients without their batch annotations — set batch_invariant=True rather than batch_invariant=False
- Latent structure is hierarchical or branched developmental trajectory, where the hyperbolic 'wn' latent_dist is preferable to the spherical 'vmf' setting

## Inputs

- Sparse gene-by-cell count matrix in .mtx format (e.g., uc_epi.mtx from Single Cell Portal study SCP551)
- Per-factor batch annotation TSV files with 0-based integer codes (uc_epi_batch_patient.tsv, uc_epi_batch_health.tsv, uc_epi_batch_location.tsv)
- Optional cell-type label TSV for downstream evaluation (uc_epi_celltype.tsv)

## Outputs

- Trained multi-level scPhere model (uc_epi_model_250epoch)
- Latent embedding matrix for all cells (uc_epi_latent_250epoch.tsv)
- Per-cell log-likelihoods (uc_epi_ll_250epoch.tsv)
- Training trace plot of log-likelihood and KL-divergence (uc_epi_ll_250epoch_train.png)

## How to apply

Load the count matrix with scPhere's read_mtx utility, transpose it, and read each batch annotation TSV (e.g., uc_epi_batch_patient.tsv, uc_epi_batch_health.tsv, uc_epi_batch_location.tsv) with pandas, concatenating them column-wise into one batch matrix with 0-based codes. Build the model as SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0), where n_batch lists the number of batches per factor (30 patients, 3 health statuses, 2 locations). Train with Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001), monitoring the log-likelihood and KL-divergence trace. The rationale is that standard VAEs for scRNA-seq accept only a single batch vector, whereas biologically relevant datasets typically have multiple such factors; scPhere's multi-level n_batch specification handles them jointly. Set batch_invariant=True instead if you need a model that maps unseen new data (e.g., new patients) without batch vectors as input.

## Related tools

- **scPhere** (Provides the SCPHERE model class whose n_batch parameter accepts a list of batch counts per factor, plus Trainer, encode, and get_log_likelihood utilities) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend dependency (v1.14.0 with tensorflow-probability v0.7.0) for training the multi-level VAE)
- **Harmony** (Single-batch-factor comparator used to benchmark the multi-level correction quality)
- **LIGER** (Single-batch-factor comparator used to benchmark the multi-level correction quality)
- **Seurat3 CCA** (Single-batch-factor comparator used to benchmark the multi-level correction quality)

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)
```

## Evaluation signals

- scPhere's k-NN and logistic-regression cell-type classification accuracies on the UC colon embeddings are at least as high as Harmony, LIGER, and Seurat3 CCA (Fig. 3d-f, Supplementary Figs. 6-9)
- All four output files exist (model, latent TSV, log-likelihood TSV, training trace PNG) after the 250-epoch run
- The log-likelihood and KL-divergence traces converge monotonically over the 250 epochs
- Batch annotation codes are verified 0-based before being passed as batch_id to Trainer
- Latent coordinates lie on the unit 3D sphere (z_dim=2 vMF embedding), verifiable when visualized with the R PlotSphere function

## Limitations

- The demo configuration uses only ~2000 variable genes for efficiency rather than all genes
- Further studies are required to validate scPhere for imputing missing counts and removing ambient RNA contamination
- Estimating the uncertainty of aligning cells from batches requires future supervised information
- Downloading the UC data from Single Cell Portal SCP551 requires a Google account login

## Evidence

- [methods] Multi-level batch specification: "n_batch: the number of batches for each component of the batch vector. For this case, there are three batch vectors, and each with 30, 3, and 2 batches, respectively."
- [intro] Single-factor limitation of prior VAEs: "current applications of VAEs for scRNA-seq data can only handle a single-batch vector (factor), whereas biologically relevant datasets typically have multiple such factors"
- [discussion] Benchmark superiority: "scPhere performed better than state-of-the-art batch-correction methods such as Harmony, Seurat3 CCA, LIGER"
- [readme] Batch-invariant mapping option: [evidence span withheld — anchor status: not_found]
