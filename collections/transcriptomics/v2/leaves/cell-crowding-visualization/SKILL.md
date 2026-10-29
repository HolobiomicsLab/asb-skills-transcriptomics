---
name: cell-crowding-visualization
description: Use when evaluating whether a low-dimensional embedding of small UMI
  count matrices suffers from cell crowding at the latent origin, comparing vMF spherical
  vs Euclidean VAE latents or t-SNE/UMAP/PHATE baselines.
when_to_use_negative:
- Datasets with complex multi-level batch factors (e.g. 30 patients x 3 disease statuses
  x 2 locations as in the IBD colon epithelial data) — that is the batch-correction
  use case with batch_invariant=False and n_batch=[30, 3, 2], not the crowding demo
- Hyperbolic trajectory embedding — use latent_dist='wn' (wrapped normal, Lorentz
  model / Poincaré disk), not the vMF spherical setup compared here
- Large datasets where crowding is not the question and the goal is reference mapping
  or batch-invariant embeddings of new individuals
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scPhere
  role: Provides SCPHERE model (latent_dist='vmf' vs normal prior), read_mtx, Trainer,
    and model.encode used to produce crowding vs non-crowding latents
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Deep-learning backend (v1.14.0) on which scPhere's VAE is trained
- name: tensorflow-probability
  role: Probabilistic distributions (vMF, normal, negative binomial) backing the latent
    and observation models
- name: t-SNE
  role: Baseline dimensionality-reduction embedding of the same count matrices for
    the crowding comparison
- name: UMAP
  role: Baseline dimensionality-reduction embedding of the same count matrices for
    the crowding comparison
- name: PHATE
  role: Baseline dimensionality-reduction embedding of the same count matrices for
    the crowding comparison
- name: PlotSphere (R, rgl)
  role: 3D interactive rendering of scPhere spherical embeddings on the unit hypersphere,
    with car2sph 2D projection option
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:6afc563f78cbefe5daecb992f93e2ffb2434f445b9ba3675eb9db79ed3705078
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - PHATE
  - PlotSphere (R, rgl)
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# cell-crowding-visualization

## Summary

Reproduces the scPhere demonstration that a hyperspherical (von Mises–Fisher prior) VAE latent space resolves the cell-crowding problem that afflicts standard normal-prior (Euclidean) VAEs and tools like t-SNE/UMAP/PHATE on small scRNA-seq datasets.

## When to use

Use when evaluating whether a low-dimensional embedding of small UMI count matrices suffers from cell crowding at the latent origin, comparing vMF spherical vs Euclidean VAE latents or t-SNE/UMAP/PHATE baselines.

## When NOT to use

- Datasets with complex multi-level batch factors (e.g. 30 patients x 3 disease statuses x 2 locations as in the IBD colon epithelial data) — that is the batch-correction use case with batch_invariant=False and n_batch=[30, 3, 2], not the crowding demo
- Hyperbolic trajectory embedding — use latent_dist='wn' (wrapped normal, Lorentz model / Poincaré disk), not the vMF spherical setup compared here
- Large datasets where crowding is not the question and the goal is reference mapping or batch-invariant embeddings of new individuals

## Inputs

- Small scRNA-seq UMI count matrices in Matrix Market (.mtx) format, e.g. the bundled example/data CD14+ monocyte/erythroid demo data
- Cell-type label vector per dataset for Silhouette scoring
- Count matrices for three external small scRNA-seq datasets used in the paper's comparison

## Outputs

- 2D/3D scPhere embeddings on the unit hypersphere (e.g. cd14_mono_eryth_latent_250epoch.tsv), visualizable with PlotSphere in R
- Euclidean (normal-prior) VAE embeddings showing central crowding
- t-SNE, UMAP, and PHATE baseline embeddings of the same matrices
- Silhouette scores of cell-type labels per method, aggregated across the four datasets, as a Supplementary Figs. 1–3 style figure

## How to apply

Load each small UMI count matrix with scPhere's read_mtx (transposing to cells x genes and densifying), including the bundled example/data CD14+ monocyte/erythroid demo data. Build two SCPHERE models with z_dim=2, observation_dist='nb', seed=0: one with latent_dist='vmf' (spherical) and a variant with a normal (Euclidean) latent prior; since the small datasets have no batch factors, set n_batch=0 and create an artificial all -1 batch vector (batch = np.zeros(x.shape[0]) * -1). Train both with Trainer(max_epoch=250, mb_size=128, learning_rate=0.001) and embed all cells via model.encode(x, batch). Compute t-SNE, UMAP, and PHATE embeddings of the same count matrices as baselines, then visualize the spherical embeddings with a PlotSphere-style 3D rendering or a 2D car2sph spherical-coordinate projection. The normal-prior latents and standard tools should show posterior means crowding at the origin, while the vMF prior spreads cell subsets over the unit hypersphere; quantify this with Silhouette scores of cell-type labels per method, aggregated across the four datasets, rendered as Supplementary Figs. 1–3 style comparisons.

## Related tools

- **scPhere** (Provides SCPHERE model (latent_dist='vmf' vs normal prior), read_mtx, Trainer, and model.encode used to produce crowding vs non-crowding latents) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Deep-learning backend (v1.14.0) on which scPhere's VAE is trained)
- **tensorflow-probability** (Probabilistic distributions (vMF, normal, negative binomial) backing the latent and observation models)
- **t-SNE** (Baseline dimensionality-reduction embedding of the same count matrices for the crowding comparison)
- **UMAP** (Baseline dimensionality-reduction embedding of the same count matrices for the crowding comparison)
- **PHATE** (Baseline dimensionality-reduction embedding of the same count matrices for the crowding comparison)
- **PlotSphere (R, rgl)** (3D interactive rendering of scPhere spherical embeddings on the unit hypersphere, with car2sph 2D projection option)

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=0, z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0); trainer = Trainer(model=model, x=x, batch_id=np.zeros(x.shape[0])*-1, max_epoch=250, mb_size=128, learning_rate=0.001); trainer.train(); z_mean = model.encode(x, batch)
```

## Evaluation signals

- vMF spherical embeddings show cell subsets distinguished on the unit hypersphere with no overlapping posterior means at the center
- Normal-prior (Euclidean) VAE latents visibly group near the origin, demonstrating the crowding pathology
- Silhouette-score ranking across scPhere-vMF, normal VAE, t-SNE, UMAP, and PHATE qualitatively matches Supplementary Figs. 1–3 (expert visual check)
- Demo run reproduces example/demo-out outputs: cd14_mono_eryth_latent_250epoch.tsv, cd14_mono_eryth_ll_250epoch.tsv, training trace png, and saved model in ~2 minutes
- Log-likelihood and KL-divergence training traces decrease smoothly over the 250 epochs

## Limitations

- Standard VAEs with a multidimensional normal prior encourage all cells' low-dimensional representations to group in the center of the latent space, causing crowding even for distinct cell types
- scPhere has been tested only on Python 3.6.7 with TensorFlow 1.14.0 and tensorflow-probability 0.7.0 on Linux and macOS
- Validation of the crowding comparison relies on expert visual comparison against the supplementary figures rather than an automated metric threshold

## Evidence

- [intro] Normal-prior crowding rationale: "they assume a multidimensional normal prior for the low-dimensional latent variables, which unfortunately encourages the low-dimensional representations of all cells to the group in the center of the"
- [methods] Model construction parameters: "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)"
- [methods] Trainer hyperparameters: "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001)"
- [readme] No-batch-vector variant from README: "For the cases there are no batch vectors, we can set n_batch=0, and create an artificial batch vector (just for running scPhere"
- [abstract] Crowding resolution claim: "ScPhere addresses multi-level, complex batch factors, facilitates the interactive visualization of large datasets, resolves cell crowding, and uncovers temporal trajectories."
