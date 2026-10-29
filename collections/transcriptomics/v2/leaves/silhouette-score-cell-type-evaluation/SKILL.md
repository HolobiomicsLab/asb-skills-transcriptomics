---
name: silhouette-score-cell-type-evaluation
description: Use when comparing dimensionality-reduction or VAE embeddings of scRNA-seq
  data with known cell-type labels and you need a quantitative, per-method separation
  metric to supplement visual checks for crowding or overlap.
when_to_use_negative:
- Cell-type labels are unavailable or unreliable — Silhouette scores require ground-truth
  labels and are meaningless on unlabeled embeddings
- Datasets are the large multi-patient IBD/colon mucosa benchmarks where the paper's
  evaluation focus is batch correction and reference mapping, not the small-dataset
  crowding benchmark
- You need a hyperbolic (wn latent_dist) embedding evaluation for developmental trajectory
  structure rather than discrete cell-type separation
edam_operation: http://edamontology.org/operation_0320
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scPhere
  role: Produces the vMF (hyperspherical) and normal-prior (Euclidean) embeddings
    whose cell-type separation is scored with Silhouettes; provides read_mtx, SCPHERE,
    Trainer, model.encode
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: t-SNE
  role: Baseline dimensionality-reduction embedding of the same count matrices included
    in the Silhouette-score comparison
- name: UMAP
  role: Baseline dimensionality-reduction embedding of the same count matrices included
    in the Silhouette-score comparison
- name: PHATE
  role: Baseline dimensionality-reduction embedding of the same count matrices included
    in the Silhouette-score comparison
- name: TensorFlow
  role: Backend (v1.14.0) on which the scPhere VAE models are trained and encoded
- name: matplotlib
  role: Renders the per-method Silhouette-score comparison figure in the style of
    Supplementary Figs. 1–3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:0cf7c9f11d9623fed09381e3de090ced73e9361d9dbf8781cc3a7838ae7e4d84
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
  edam_operation: http://edamontology.org/operation_0320
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# silhouette-score-cell-type-evaluation

## Summary

Quantitatively evaluates how well a low-dimensional embedding (e.g., scPhere vMF spherical vs. normal-prior Euclidean VAE vs. t-SNE/UMAP/PHATE) separates known cell types by computing Silhouette scores of cell-type labels on each embedding, as done in the scPhere paper's Supplementary Figs. 1–3 benchmark on small scRNA-seq datasets.

## When to use

Use when comparing dimensionality-reduction or VAE embeddings of scRNA-seq data with known cell-type labels and you need a quantitative, per-method separation metric to supplement visual checks for crowding or overlap.

## When NOT to use

- Cell-type labels are unavailable or unreliable — Silhouette scores require ground-truth labels and are meaningless on unlabeled embeddings
- Datasets are the large multi-patient IBD/colon mucosa benchmarks where the paper's evaluation focus is batch correction and reference mapping, not the small-dataset crowding benchmark
- You need a hyperbolic (wn latent_dist) embedding evaluation for developmental trajectory structure rather than discrete cell-type separation

## Inputs

- scRNA-seq UMI count matrix (cells x genes, e.g., .mtx read via scPhere's read_mtx and transposed/densified)
- Cell-type label vector for each cell
- Embedding coordinates per method (scPhere vMF z_mean, scPhere normal-prior z_mean, t-SNE, UMAP, PHATE coordinates)

## Outputs

- Per-method, per-dataset Silhouette scores of cell-type labels
- Aggregated Silhouette-score comparison across the four small datasets
- Supplementary-Figs-1–3-style comparison figure (e.g., via matplotlib)

## How to apply

First produce each method's embedding of the same cell-by-gene UMI count matrix: a scPhere model with z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0, trained with Trainer (max_epoch=250, mb_size=128, learning_rate=0.001); a variant with a normal (Euclidean) latent prior; and baselines (t-SNE, UMAP, PHATE). Obtain latent coordinates via model.encode(x, batch) — for datasets with no batch factors use n_batch=0 and an artificial all -1 batch vector (batch = np.zeros(x.shape[0]) * -1). For each embedding, compute the Silhouette score using the provided cell-type labels, then aggregate the scores across the benchmark datasets (the paper's four small UMI datasets) per method. The rationale is that the multidimensional normal prior of standard VAEs encourages all latent representations to group near the origin (cell crowding), which should depress label separation relative to the vMF uniform-prior-on-hypersphere embeddings. Judge success by rendering the per-dataset and aggregated Silhouette-score comparison as a Supplementary-Figs-1–3-style figure and checking the ranking qualitatively matches the reported pattern.

## Related tools

- **scPhere** (Produces the vMF (hyperspherical) and normal-prior (Euclidean) embeddings whose cell-type separation is scored with Silhouettes; provides read_mtx, SCPHERE, Trainer, model.encode) — https://github.com/klarman-cell-observatory/scPhere
- **t-SNE** (Baseline dimensionality-reduction embedding of the same count matrices included in the Silhouette-score comparison)
- **UMAP** (Baseline dimensionality-reduction embedding of the same count matrices included in the Silhouette-score comparison)
- **PHATE** (Baseline dimensionality-reduction embedding of the same count matrices included in the Silhouette-score comparison)
- **TensorFlow** (Backend (v1.14.0) on which the scPhere VAE models are trained and encoded)
- **matplotlib** (Renders the per-method Silhouette-score comparison figure in the style of Supplementary Figs. 1–3)

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=0, z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0); trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001); trainer.train(); z_mean = model.encode(x, batch)
```

## Evaluation signals

- The vMF spherical embeddings show no central crowding: posterior means of distinct cell subsets do not overlap at the origin, unlike the normal-prior variant
- Silhouette-score ranking across methods (scPhere vMF vs. normal-prior VAE vs. t-SNE/UMAP/PHATE) qualitatively matches the pattern in Supplementary Figs. 1–3, verified by expert visual check
- All methods are scored on embeddings of the identical count matrices (same cells, same ~cell-by-gene input), so differences reflect embedding geometry, not data preprocessing
- Trained scPhere model uses the documented reproducibility settings: z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0, max_epoch=250, mb_size=128, learning_rate=0.001
- Training trace (log-likelihood and KL-divergence per iteration via plot_trace) converges over the 250 epochs before embeddings are extracted

## Limitations

- Silhouette-score agreement with the paper's figure is a qualitative (expert visual) check, not a strict numeric threshold
- The paper notes scPhere's ability to impute missing counts and remove ambient RNA contamination requires further validation, so embeddings of very noisy matrices may affect label-separation scores
- scPhere was tested on Python 3.6.7 with tensorflow v1.14.0 and tensorflow-probability v0.7.0; newer TensorFlow 2.x environments may break the model/trainer code

## Evidence

- [intro] Normal-prior crowding motivation: "they assume a multidimensional normal prior for the low-dimensional latent variables, which unfortunately encourages the low-dimensional representations of all cells to the group in the center of the"
- [readme] Training configuration: "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001)"
- [methods] Embedding extraction: "z_mean = model.encode(x, batch)"
- [readme] No-batch-vector handling: "For the cases there are no batch vectors, we can set n_batch=0, and create an artificial batch vector (just for running scPhere"
- [abstract] Crowding claim: "ScPhere addresses multi-level, complex batch factors, facilitates the interactive visualization of large datasets, resolves cell crowding, and uncovers temporal trajectories."
