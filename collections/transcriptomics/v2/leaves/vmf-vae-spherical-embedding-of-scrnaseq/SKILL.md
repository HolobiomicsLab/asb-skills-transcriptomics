---
name: vmf-vae-spherical-embedding-of-scrnaseq
description: Use when embedding very large, sparse droplet scRNA-seq UMI count matrices
  (e.g., RGC ~35,699 cells or HCL ~599,926 cells) where a normal-prior VAE or 2D t-SNE/UMAP
  crowds cells and distorts hierarchical global structure.
when_to_use_negative:
- The goal is batch correction/integration that must preserve biological variation
  rather than a geometric embedding on a sphere — scVI, Harmony, Seurat3 CCA, LIGER,
  fastMNN, Scanorama, or Conos are the comparison methods there
- Data or the target analysis lives naturally in hyperbolic space (hierarchical, branched
  developmental trajectories), for which the 'wn' hyperbolic Lorentz/Poincaré-disk
  variant is the article's intended choice
- Only imputation of missing counts or ambient RNA removal is needed — the article
  explicitly says further studies are required for these abilities
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scPhere
  role: Deep generative model providing SCPHERE (latent_dist='vmf'), Trainer, encode,
    get_log_likelihood, and save_sess for hyperspherical embedding
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend (v1.14.0) with tensorflow-probability v0.7.0 on which the vMF/negative-binomial
    VAE is trained
- name: numpy
  role: Matrix handling (>=1.16.4), transposing read_mtx output, and writing embeddings
    with np.savetxt
- name: pandas
  role: Loading and concatenating the multilevel batch vectors (>=0.21.0)
- name: densitycut
  role: R library used with rgl/robustbase in the PlotSphere workflow to interactively
    visualize the spherical embeddings and cell-type median centers
  repo: https://bitbucket.org/jerry00/densitycut_dev
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:fb1e75c34d5fdbb42da63ab15bca67eb6204316b798c8063b817b63f8b9c79dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - TensorFlow
  - densitycut
  - numpy
  - pandas
  - scPhere
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# vmf-vae-spherical-embedding-of-scrnaseq

## Summary

Train a scPhere variational autoencoder that embeds UMI count matrices of single-cell RNA-seq profiles onto a low-dimensional unit hypersphere using von Mises–Fisher (vMF) latent distributions, avoiding the cell crowding and distortion caused by multidimensional normal priors in Euclidean-space VAEs.

## When to use

Use when embedding very large, sparse droplet scRNA-seq UMI count matrices (e.g., RGC ~35,699 cells or HCL ~599,926 cells) where a normal-prior VAE or 2D t-SNE/UMAP crowds cells and distorts hierarchical global structure.

## When NOT to use

- The goal is batch correction/integration that must preserve biological variation rather than a geometric embedding on a sphere — scVI, Harmony, Seurat3 CCA, LIGER, fastMNN, Scanorama, or Conos are the comparison methods there
- Data or the target analysis lives naturally in hyperbolic space (hierarchical, branched developmental trajectories), for which the 'wn' hyperbolic Lorentz/Poincaré-disk variant is the article's intended choice
- Only imputation of missing counts or ambient RNA removal is needed — the article explicitly says further studies are required for these abilities

## Inputs

- Sparse UMI count matrix (.mtx) from droplet scRNA-seq, transposed to cell-by-gene dense form
- 0-based batch vector(s) as TSV (e.g., uc_epi_batch_patient.tsv, uc_epi_batch_health.tsv, uc_epi_batch_location.tsv)
- Cell-type label file for downstream k-NN accuracy evaluation (e.g., uc_epi_celltype.tsv)

## Outputs

- *_latent_250epoch.tsv containing vMF z_mean embeddings (3D cartesian coordinates on the unit hypersphere for z_dim=2)
- *_ll_250epoch.tsv log-likelihoods per cell
- Trained scPhere model checkpoint (*_model_250epoch*) reloadable via model.save_sess
- Training trace PNG of log-likelihood and KL-divergence per iteration

## How to apply

Read the sparse count matrix with scPhere's read_mtx and transpose it to cell-by-gene form (optionally restricting to ~2000 variable genes for efficiency). Build an SCPHERE model with n_gene=x.shape[1], z_dim=2 (or higher for global-structure benchmarks), latent_dist='vmf' for spherical spaces ('wn' for hyperbolic), observation_dist='nb', and seed=0; supply n_batch lists for multilevel batch factors (e.g., [30, 3, 2] for patient/health/location), or n_batch=0 with an artificial all -1 batch vector when no batches exist. Train with Trainer(max_epoch=250, mb_size=128, learning_rate=0.001), save with model.save_sess, then embed all cells via z_mean = model.encode(x, batch) and write z_mean to a *_latent_250epoch.tsv file; set batch_invariant=True when the trained model must map new patients unseen during training. Evaluate by comparing log-likelihood/KL training traces and, for structure preservation, by global k-NN classification accuracy to major cell types using cell-type medians as cluster centers (scPhere reached 73.53%/92.21% on RGC/HCL vs 47.06%/79.22% t-SNE and 52.94%/83.12% UMAP).

## Related tools

- **scPhere** (Deep generative model providing SCPHERE (latent_dist='vmf'), Trainer, encode, get_log_likelihood, and save_sess for hyperspherical embedding) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend (v1.14.0) with tensorflow-probability v0.7.0 on which the vMF/negative-binomial VAE is trained)
- **numpy** (Matrix handling (>=1.16.4), transposing read_mtx output, and writing embeddings with np.savetxt)
- **pandas** (Loading and concatenating the multilevel batch vectors (>=0.21.0))
- **densitycut** (R library used with rgl/robustbase in the PlotSphere workflow to interactively visualize the spherical embeddings and cell-type median centers) — https://bitbucket.org/jerry00/densitycut_dev

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=0, z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0); trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001); trainer.train(); z_mean = model.encode(x, batch)
```

## Evaluation signals

- Training trace shows log-likelihood rising and KL-divergence stabilizing over 250 epochs (plot_trace of trainer.status['log_likelihood'] and 'kl_divergence')
- Embedding rows have unit norm (points lie on the unit hypersphere, visualized in PlotSphere with xlim/ylim/zlim = [-1, 1])
- Global k-NN classification accuracy to major cell types matches reported scPhere values within ~±0.5 pp (73.53% RGC, 92.21% HCL) and ranks above t-SNE (47.06%/79.22%) and UMAP (52.94%/83.12%)
- Embedding shows reduced central crowding of distinct cell types relative to a normal-prior VAE latent space
- Output files exist with expected schema: *_latent_250epoch.tsv (cells × 3), *_ll_250epoch.tsv (one log-likelihood per cell), and a reloadable *_model_250epoch* checkpoint

## Limitations

- Batch vector must be 0-based; incorrect indexing of n_batch (e.g., [30, 3, 2] for 30 patients, 3 health statuses, 2 locations) will break training
- Ability to impute missing counts and remove ambient RNA contamination is unvalidated — the article states further studies are required
- Embedding on a hyperbolic/spherical geometry lacks supporting tooling, e.g., efficient k-NN search tools in hyperbolic space
- Standard (non-batch-invariant) models require the batch vector at encode time; mapping new patients requires batch_invariant=True training

## Evidence

- [methods] model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0): "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)"
- [methods] trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001): "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001)"
- [methods] z_mean = model.encode(x, batch): "z_mean = model.encode(x, batch)"
- [intro] we minimize the distortion by embedding cells to a lower-dimensional hypersphere instead of a low-dimensional Euclidean space: "we minimize the distortion by embedding cells to a lower-dimensional hypersphere instead of a low-dimensional Euclidean space"
- [readme] Prepared sparse matrix and using ~2000 variable genes for efficiency.: "# Preparing a sparse matrix and using ~2000 variable genes for efficiency."
- [readme] latent_dist: 'vmf' for spherical latent spaces, and 'wn' for hyperbolic latent spaces: "# latent_dist: 'vmf' for spherical latent spaces, and 'wn' for hyperbolic latent spaces"
