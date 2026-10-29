---
name: von-mises-fisher-spherical-latent-modeling
description: Use when scRNA-seq count matrices must be embedded on a hypersphere to
  avoid Gaussian-VAE cell crowding, corrected for multilevel batch factors (e.g.,
  30 patients x 3 disease statuses x 2 locations), or mapped from unseen individuals.
when_to_use_negative:
- Data lie on hierarchical, branched developmental trajectories better suited to hyperbolic
  space — use latent_dist='wn' (wrapped normal, Poincaré disk) instead of vMF
- The goal is imputing missing counts or removing ambient RNA contamination — the
  article states further studies are required to validate scPhere for these tasks
- Only a Euclidean 2D plot is needed without batch modeling; simpler dimensionality
  reduction avoids VAE training overhead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scPhere
  role: Core VAE implementation with vMF latent posteriors, multilevel n_batch specification,
    batch_invariant mode, Trainer, encode, and get_log_likelihood APIs
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend (v1.14.0) used to train the scPhere model; installed via pip install
    tensorflow==1.14
- name: Harmony
  role: Comparator batch-correction method whose embeddings are benchmarked against
    scPhere with k-NN and logistic-regression cell-type accuracy
- name: LIGER
  role: Comparator batch-correction method for the UC colon benchmark (Fig. 3d-f)
- name: Seurat3 CCA
  role: Comparator batch-correction method for the UC colon benchmark (Fig. 3d-f)
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:ffe3be0d77b950c09b2ae13233b73966f238500d2d15df1bd79ce002a87b1481
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - Harmony
  - LIGER
  - Seurat3 CCA
  - TensorFlow
  - numpy
  - pandas
  - scPhere
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# von-mises-fisher-spherical-latent-modeling

## Summary

Train scPhere, a deep generative VAE whose latent variables follow von Mises–Fisher (vMF) distributions on a unit hypersphere, to embed scRNA-seq cells without the central crowding of Gaussian-latent VAEs and to support multilevel batch correction and batch-invariant mapping of new patients.

## When to use

Use when scRNA-seq count matrices must be embedded on a hypersphere to avoid Gaussian-VAE cell crowding, corrected for multilevel batch factors (e.g., 30 patients x 3 disease statuses x 2 locations), or mapped from unseen individuals.

## When NOT to use

- Data lie on hierarchical, branched developmental trajectories better suited to hyperbolic space — use latent_dist='wn' (wrapped normal, Poincaré disk) instead of vMF
- The goal is imputing missing counts or removing ambient RNA contamination — the article states further studies are required to validate scPhere for these tasks
- Only a Euclidean 2D plot is needed without batch modeling; simpler dimensionality reduction avoids VAE training overhead

## Inputs

- Sparse UMI count matrix (.mtx, e.g., uc_epi.mtx) with ~2000 variable genes
- 0-based batch annotation TSVs (uc_epi_batch_patient.tsv, uc_epi_batch_health.tsv, uc_epi_batch_location.tsv)
- Cell-type label TSV (uc_epi_celltype.tsv) for downstream classification benchmarks

## Outputs

- Trained scPhere model session (e.g., uc_epi_model_250epoch)
- Latent embedding TSV of unit-sphere coordinates (e.g., uc_epi_latent_250epoch.tsv)
- Per-cell log-likelihood TSV (e.g., uc_epi_ll_250epoch.tsv)
- Training trace PNG of log-likelihood and KL-divergence per iteration
- k-NN / logistic-regression classification accuracies per cell type

## How to apply

Load a sparse UMI count matrix (e.g., uc_epi.mtx from Single Cell Portal SCP551) with scPhere's read_mtx utility and transpose it, then assemble a 0-based batch vector concatenating patient, health, and location annotations. Build a SCPHERE model with n_gene equal to the gene count, n_batch=[30, 3, 2] (or a single vector), z_dim=2, latent_dist='vmf', observation_dist='nb', and seed=0; set batch_invariant=True (feeding only gene expression to the encoder) when the goal is mapping cells from held-out patients. Train with the Trainer class using max_epoch=250, mb_size=128, and learning_rate=0.001, monitoring the log-likelihood and KL-divergence trace, then save the session with model.save_sess. Embed cells with model.encode (unit-sphere coordinates visualizable in R via PlotSphere/rgl) and compute per-cell log-likelihoods with model.get_log_likelihood; optionally train k-NN classifiers (k=25) on latent coordinates to benchmark batch correction or classify test-patient cells.

## Related tools

- **scPhere** (Core VAE implementation with vMF latent posteriors, multilevel n_batch specification, batch_invariant mode, Trainer, encode, and get_log_likelihood APIs) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend (v1.14.0) used to train the scPhere model; installed via pip install tensorflow==1.14)
- **Harmony** (Comparator batch-correction method whose embeddings are benchmarked against scPhere with k-NN and logistic-regression cell-type accuracy)
- **LIGER** (Comparator batch-correction method for the UC colon benchmark (Fig. 3d-f))
- **Seurat3 CCA** (Comparator batch-correction method for the UC colon benchmark (Fig. 3d-f))

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0); Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001).train()
```

## Evaluation signals

- Training trace shows increasing log-likelihood and tracked KL-divergence over 250 epochs (plotted with plot_trace)
- Encoded cell coordinates lie on the unit sphere (|z| ≈ 1; R PlotSphere renders with xlim/ylim/zlim c(-1, 1))
- k-NN (k=25) and logistic-regression cell-type classification accuracies on scPhere embeddings meet or exceed Harmony, LIGER, and Seurat3 CCA (Fig. 3d-f)
- Batch-invariant model trained on 18 UC patients classifies 12 held-out test patients with accuracies ~0.79 (stromal), 0.83 (epithelial), 0.82 (immune), close to the all-patient model's 0.80/0.87/0.80 (Fig. 4d-g)
- All four expected output files exist: *_model_250epoch, *_latent_250epoch.tsv, *_ll_250epoch.tsv, *_train.png

## Limitations

- scPhere's abilities for imputing missing counts and removing ambient RNA contamination are unvalidated and require further study
- Estimating uncertainty of aligning cells from batches requires future supervised information
- Efficient k-NN search tools for hyperbolic embeddings are lacking; semi-supervised annotation, perturbation modeling, and multimodal integration are future extensions, not implemented

## Evidence

- [intro] vMF latent with uniform prior avoids center crowding: "we minimize the distortion by embedding cells to a lower-dimensional hypersphere instead of a low-dimensional Euclidean space"
- [intro] multilevel batch factors motivation: "current applications of VAEs for scRNA-seq data can only handle a single-batch vector (factor), whereas biologically relevant datasets typically have multiple such factors"
- [methods] model construction call: "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)"
- [methods] training hyperparameters: "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001)"
- [readme] batch-invariant mode enables mapping new patients: [evidence span withheld — anchor status: not_found]
- [discussion] outperforms comparator batch-correction methods: "scPhere performed better than state-of-the-art batch-correction methods such as Harmony, Seurat3 CCA, LIGER"
