---
name: scrna-dimensionality-reduction-benchmarking
description: Use when benchmarking dimensionality-reduction embeddings of small UMI
  count matrices to test whether a vMF/hyperspherical latent space avoids the origin-crowding
  seen with normal-prior VAEs and t-SNE/UMAP/PHATE.
when_to_use_negative:
- Datasets are large with real multi-level batch factors (patient/disease/location)
  — use batch-aware scPhere with n_batch lists instead of the no-batch crowding benchmark
- The goal is trajectory/developmental analysis — use scPhere's hyperbolic (Lorentz/Poincaré,
  latent_dist='wn') mode rather than this spherical-vs-Euclidean crowding comparison
- Input is already an embedding or a feature table rather than a raw UMI count matrix
  — scPhere models negative-binomial gene expression counts directly
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0091
tools:
- name: scPhere
  role: Deep generative model providing SCPHERE (vMF spherical) and normal-prior (Euclidean)
    VAE variants, read_mtx loading, Trainer, encode, and log-likelihood utilities
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend for training the VAE models (v1.14.0 used with tensorflow-probability
    v0.7.0)
- name: tensorflow-probability
  role: Probability distributions (vMF, negative binomial) underlying the vMF/normal
    latent and NB observation models
- name: t-SNE
  role: Baseline dimensionality-reduction embedding of the same count matrices for
    comparison
- name: UMAP
  role: Baseline dimensionality-reduction embedding of the same count matrices for
    comparison
- name: PHATE
  role: Baseline dimensionality-reduction embedding of the same count matrices for
    comparison
- name: matplotlib
  role: Rendering trace plots and 2D embedding comparison figures
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:94fd8ff421a0ffa137460ed90b608d17af5e5b2eed722cc4cbd7fa545828d5d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - PHATE
  - TensorFlow
  - UMAP
  - matplotlib
  - numpy
  - pandas
  - scPhere
  - scipy
  - t-SNE
  - tensorflow-probability
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# scrna-dimensionality-reduction-benchmarking

## Summary

Benchmark workflow comparing scPhere's hyperspherical (von Mises–Fisher latent) VAE embeddings against a standard normal-prior (Euclidean) VAE and classical tools (t-SNE, UMAP, PHATE) on small scRNA-seq datasets, to test whether a spherical latent space resolves the cell-crowding problem at the origin.

## When to use

Use when benchmarking dimensionality-reduction embeddings of small UMI count matrices to test whether a vMF/hyperspherical latent space avoids the origin-crowding seen with normal-prior VAEs and t-SNE/UMAP/PHATE.

## When NOT to use

- Datasets are large with real multi-level batch factors (patient/disease/location) — use batch-aware scPhere with n_batch lists instead of the no-batch crowding benchmark
- The goal is trajectory/developmental analysis — use scPhere's hyperbolic (Lorentz/Poincaré, latent_dist='wn') mode rather than this spherical-vs-Euclidean crowding comparison
- Input is already an embedding or a feature table rather than a raw UMI count matrix — scPhere models negative-binomial gene expression counts directly

## Inputs

- Small scRNA-seq UMI count matrices in .mtx (MatrixMarket) format, plus the bundled example/data CD14+ monocyte/erythroid demo data
- Cell-type/cluster label vectors per dataset (for Silhouette scoring)
- Artificial 0-based batch vector (all -1) when datasets have no batch factors

## Outputs

- z_dim=2 latent embeddings (unit-hypersphere points for vMF, origin-centered for normal prior) per method and dataset
- t-SNE, UMAP, and PHATE baseline embeddings of the same count matrices
- Silhouette scores of cell-type labels per method, aggregated across the four small datasets
- Crowding-visualization figures (3D sphere renders or 2D spherical-coordinate projections) in the Supplementary Figs. 1-3 style

## How to apply

Load each small UMI count matrix with scPhere's read_mtx, transpose to cells x genes, and densify. Build two SCPHERE models per dataset with z_dim=2, observation_dist='nb', seed=0: one with latent_dist='vmf' (spherical) and one with a normal (Euclidean) latent prior; since these small datasets have no batch factors, set n_batch=0 and use an artificial all -1 batch vector, which per the README does not influence results. Train each with Trainer(max_epoch=250, mb_size=128, learning_rate=0.001), then embed all cells via model.encode and compute baseline t-SNE, UMAP, and PHATE embeddings on the same matrices. Visualize the spherical embeddings (3D PlotSphere-style rendering or car2sph 2D spherical-coordinate projection) and compute Silhouette scores of cell-type labels for each method; aggregate across the four datasets and compare against the pattern in the paper's Supplementary Figs. 1-3.

## Related tools

- **scPhere** (Deep generative model providing SCPHERE (vMF spherical) and normal-prior (Euclidean) VAE variants, read_mtx loading, Trainer, encode, and log-likelihood utilities) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend for training the VAE models (v1.14.0 used with tensorflow-probability v0.7.0))
- **tensorflow-probability** (Probability distributions (vMF, negative binomial) underlying the vMF/normal latent and NB observation models)
- **t-SNE** (Baseline dimensionality-reduction embedding of the same count matrices for comparison)
- **UMAP** (Baseline dimensionality-reduction embedding of the same count matrices for comparison)
- **PHATE** (Baseline dimensionality-reduction embedding of the same count matrices for comparison)
- **matplotlib** (Rendering trace plots and 2D embedding comparison figures)

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=0, z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0); batch = np.zeros(x.shape[0]) * -1; trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001); z_mean = model.encode(x, batch)
```

## Evaluation signals

- The vMF spherical embeddings show no central crowding — cell posterior means do not overlap at the origin, unlike the normal-prior VAE embeddings
- Silhouette scores of cell-type labels rank methods qualitatively as reported in Supplementary Figs. 1-3 across the four small datasets
- Training completes 250 epochs with logged log-likelihood and KL-divergence traces (plot_trace output) showing convergence
- Encoded spherical embeddings lie on the unit hypersphere and render correctly within PlotSphere's [-1,1] axis limits
- Expert visual check of figures against the paper's Supplementary Figs. 1-3 pattern

## Limitations

- Normal-prior VAEs encourage low-dimensional representations of all cells to group in the center of the latent space, causing cell crowding even for distinct cell types — this is the artifact being benchmarked, not a scPhere failure
- Embedding data effectively distributed on a hypersphere (sparse droplet data under cosine distance) into Euclidean space introduces significant distortion, so Euclidean baselines may be structurally disadvantaged
- Demo/training runtime: the README notes the demo script takes about 2 minutes; TF1.14 / tensorflow-probability 0.7.0 pin old versions

## Evidence

- [intro] small-dataset crowding contrast: "they assume a multidimensional normal prior for the low-dimensional latent variables, which unfortunately encourages the low-dimensional representations of all cells to the group in the center of the"
- [intro] distortion rationale for spherical embedding: "Embedding data distributed on a hypersphere to a Euclidean space introduces significant distortion for commonly used dimensionality reduction tools"
- [methods] model construction: "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)"
- [methods] training hyperparameters: "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001)"
- [readme] no-batch setup: "batch = np.zeros(x.shape[0]) * -1"
- [readme] latent_dist semantics: "latent_dist: 'vmf' for spherical latent spaces, and 'wn' for hyperbolic latent spaces"
- [readme] 2D projection of sphere: "We can simply convert the 3D cartesian coordinates to spherical coordinates using the car2sph function."
