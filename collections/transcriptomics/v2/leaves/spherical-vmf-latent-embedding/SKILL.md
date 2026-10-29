---
name: spherical-vmf-latent-embedding
description: Use when scRNA-seq droplet UMI count matrices need a 2D latent embedding
  where cells of distinct types should not overlap at the origin, or when sparse data
  effectively lies on a hypersphere under cosine distance.
when_to_use_negative:
- Data are sparse droplet-based scRNA-seq that you want to represent as hyperbolic
  (branched hierarchical) trajectories — use the 'wn' latent_dist hyperbolic Lorentz/Poincaré
  setting instead
- The goal is batch correction/mapping of new individuals requiring batch-invariant
  embeddings — configure batch_invariant=True rather than the plain vMF model
- Input is already a low-dimensional embedding or a non-count (e.g. TPM/log-normalized)
  matrix — the model expects a raw UMI count matrix with observation_dist='nb'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPhere
  role: Core deep-generative model (SCPHERE class with latent_dist='vmf', observation_dist='nb')
    that embeds cells on a unit hypersphere; includes read_mtx, Trainer, encode, save_sess
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend (v1.14.0) on which the vMF-VAE is trained via gradient descent
- name: tensorflow-probability
  role: v0.7.0; provides von Mises–Fisher distribution support for the spherical latent
    posterior
- name: t-SNE
  role: Euclidean baseline embedding of the same count matrices for the crowding comparison
- name: UMAP
  role: Euclidean baseline embedding of the same count matrices for the crowding comparison
- name: PHATE
  role: Euclidean baseline embedding of the same count matrices for the crowding comparison
- name: densitycut / PlotSphere (R)
  role: 'R visualization: rgl-based PlotSphere 3D sphere rendering and car2sph Cartesian-to-spherical
    2D projection of the latent embeddings'
  repo: https://bitbucket.org/jerry00/densitycut_dev
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:540eb5875abe86de8dc98a60ad4af564e980ee3f9dfd8ffa02f9afe56a4275e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - PHATE
  - TensorFlow
  - UMAP
  - densitycut / PlotSphere (R)
  - matplotlib
  - numpy
  - pandas
  - scPhere
  - scipy
  - t-SNE
  - tensorflow-probability
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# spherical-vmf-latent-embedding

## Summary

Embed UMI count matrices of single-cell RNA-seq profiles into a low-dimensional unit hypersphere using scPhere's vMF-latent VAE, avoiding the central cell-crowding that a standard multivariate-normal (Euclidean) prior induces.

## When to use

Use when scRNA-seq droplet UMI count matrices need a 2D latent embedding where cells of distinct types should not overlap at the origin, or when sparse data effectively lies on a hypersphere under cosine distance.

## When NOT to use

- Data are sparse droplet-based scRNA-seq that you want to represent as hyperbolic (branched hierarchical) trajectories — use the 'wn' latent_dist hyperbolic Lorentz/Poincaré setting instead
- The goal is batch correction/mapping of new individuals requiring batch-invariant embeddings — configure batch_invariant=True rather than the plain vMF model
- Input is already a low-dimensional embedding or a non-count (e.g. TPM/log-normalized) matrix — the model expects a raw UMI count matrix with observation_dist='nb'

## Inputs

- UMI count matrix in .mtx (MatrixMarket sparse) format, loaded via scPhere's read_mtx and transposed to cells x genes
- Optional batch vectors (0-based TSV files) for multi-level batch factors; an artificial all -1 batch vector when no batches exist
- Cell-type label annotations for Silhouette-score evaluation
- Optional baseline count-matrix inputs for t-SNE, UMAP, PHATE comparison

## Outputs

- 2D latent spherical coordinates (z_mean) per cell, written with np.savetxt as a TSV, viewable on a unit sphere
- Per-cell log-likelihoods from model.get_log_likelihood
- Training trace plot of log-likelihood and KL-divergence per iteration
- Saved trained model checkpoint via model.save_sess
- Silhouette scores per method for the crowding comparison

## How to apply

Load the UMI count matrix with scPhere's read_mtx, transpose to cells-by-genes and densify, and restrict to ~2000 variable genes for efficiency. Build a SCPHERE model with z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0 for reproducibility; if the dataset has no batch factors, set n_batch=0 and supply an artificial all -1 batch vector (batch = np.zeros(x.shape[0]) * -1), which does not influence results. Train with Trainer (max_epoch=250, mb_size=128, learning_rate=0.001), then obtain the spherical coordinates with model.encode(x, batch). To demonstrate the crowding problem, train a parallel variant with a normal (Euclidean) latent prior and optionally run t-SNE, UMAP and PHATE on the same matrices as baselines; quantify separation with Silhouette scores of cell-type labels and visualize on the sphere (PlotSphere-style 3D or car2sph 2D spherical projection). The vMF latent space uses a uniform prior on the unit hypersphere, so posterior means are not forced to cluster in the center.

## Related tools

- **scPhere** (Core deep-generative model (SCPHERE class with latent_dist='vmf', observation_dist='nb') that embeds cells on a unit hypersphere; includes read_mtx, Trainer, encode, save_sess) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend (v1.14.0) on which the vMF-VAE is trained via gradient descent)
- **tensorflow-probability** (v0.7.0; provides von Mises–Fisher distribution support for the spherical latent posterior)
- **t-SNE** (Euclidean baseline embedding of the same count matrices for the crowding comparison)
- **UMAP** (Euclidean baseline embedding of the same count matrices for the crowding comparison)
- **PHATE** (Euclidean baseline embedding of the same count matrices for the crowding comparison)
- **densitycut / PlotSphere (R)** (R visualization: rgl-based PlotSphere 3D sphere rendering and car2sph Cartesian-to-spherical 2D projection of the latent embeddings) — https://bitbucket.org/jerry00/densitycut_dev

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=0, z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0); Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001).train(); z_mean = model.encode(x, batch)
```

## Evaluation signals

- vMF spherical embeddings must show no central crowding — cell subsets of distinct types should be separated on the sphere without overlapping posterior means, unlike the normal-prior variant
- Silhouette scores of cell-type labels for the vMF embedding must qualitatively match the ranking reported in Supplementary Figs. 1–3 across the four small datasets
- Training trace should show decreasing KL-divergence and increasing log-likelihood over the 250 epochs (plot_trace output)
- Embedded 3D Cartesian coordinates should lie near the unit sphere (PlotSphere uses xlim/ylim/zlim=c(-1,1) and a radius-0.99 sphere), and car2sph conversion should yield valid spherical coordinates
- The normal-prior (Euclidean VAE) variant should reproduce the crowding artifact — latent means centered at the origin — confirming the contrast is due to the prior

## Limitations

- Standard Euclidean baselines (t-SNE/UMAP/PHATE) distort data that effectively lies on a hypersphere under cosine distance, so comparisons with them are asymmetric by design
- Hyperbolic-space tooling is immature — 'there are not yet many tools to process data distributed in the hyperbolic space, such as efficient k-NN search tools'
- ScPhere's imputation of missing counts and ambient-RNA removal remain unvalidated, so embeddings should not be relied on for those tasks
- A normal (multivariate) prior encourages all cells to group in the center of the latent space, causing crowding even for data with distinct cell types — the core failure mode this skill avoids

## Evidence

- [intro] Uniform prior prevents central crowding: "we minimize the distortion by embedding cells to a lower-dimensional hypersphere instead of a low-dimensional Euclidean space"
- [intro] Normal prior causes crowding: "they assume a multidimensional normal prior for the low-dimensional latent variables, which unfortunately encourages the low-dimensional representations of all cells to the group in the center of the"
- [methods] Model construction with vMF latent and NB observation: "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)"
- [methods] Training hyperparameters: "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001)"
- [readme] Artificial batch vector when no batches exist: "batch = np.zeros(x.shape[0]) * -1"
- [readme] Sparse matrix preparation: "Preparing a sparse matrix and using ~2000 variable genes for efficiency."
- [readme] 3D sphere visualization: "cd14_mono_eryth_latent_250epoch.tsv (the embeddings in a sphere, which can be visulzied by the PlotSphere R function"
- [readme] 2D spherical-coordinate projection: "We can simply convert the 3D cartesian coordinates to spherical coordinates using the car2sph function."
