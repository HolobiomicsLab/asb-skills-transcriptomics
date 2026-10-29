---
name: scphere-model-training-and-inference
description: Use when very large scRNA-seq UMI count matrices (e.g., 35,699-cell RGC
  or 599,926-cell HCL droplet data) need low-dimensional embeddings that preserve
  global hierarchical structure, or when mapping new batches/patients batch-invariantly.
when_to_use_negative:
- Data already consists of normalized embeddings or a feature table rather than raw
  UMI counts — scPhere models negative-binomial count observations directly
- The goal is local neighborhood visualization only, where t-SNE/UMAP already suffice
  and no batch correction or global hierarchical structure preservation is required
- Analyses requiring tools unavailable in hyperbolic space, such as efficient k-NN
  search, which the article notes are lacking
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scPhere
  role: Core VAE model (SCPHERE class) with vMF latent distribution and negative-binomial
    observation distribution for hyperspherical embedding of scRNA-seq counts
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Deep-learning backend (v1.14) used to train the VAE via the Trainer class
- name: t-SNE
  role: Baseline 2D embedding method against which scPhere's global k-NN hierarchical-structure
    accuracy (47.06% RGC, 79.22% HCL) is compared
- name: UMAP
  role: Baseline 2D embedding method compared against scPhere (52.94% RGC, 83.12%
    HCL global k-NN accuracy)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:81d705ccaf77e642de538ef7beee1bff4115d88031ef0d9b4323265c5489313a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - TensorFlow
  - UMAP
  - numpy
  - pandas
  - scPhere
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# scphere-model-training-and-inference

## Summary

Train and apply scPhere, a deep generative VAE that embeds scRNA-seq UMI count matrices into low-dimensional hyperspherical (von Mises–Fisher) or hyperbolic latent spaces, avoiding the cell crowding of normal-prior VAEs and preserving hierarchical global structure better than t-SNE or UMAP.

## When to use

Use when very large scRNA-seq UMI count matrices (e.g., 35,699-cell RGC or 599,926-cell HCL droplet data) need low-dimensional embeddings that preserve global hierarchical structure, or when mapping new batches/patients batch-invariantly.

## When NOT to use

- Data already consists of normalized embeddings or a feature table rather than raw UMI counts — scPhere models negative-binomial count observations directly
- The goal is local neighborhood visualization only, where t-SNE/UMAP already suffice and no batch correction or global hierarchical structure preservation is required
- Analyses requiring tools unavailable in hyperbolic space, such as efficient k-NN search, which the article notes are lacking

## Inputs

- Sparse scRNA-seq UMI count matrix (.mtx, gene-by-cell, transposed to cell-by-gene via read_mtx)
- 0-based batch vector(s) as TSV files (e.g., patient, health status, location batch labels)
- Major cell-type label files (for downstream k-NN accuracy evaluation)
- Installation deps: Python >=3.6, tensorflow==1.14, tensorflow-probability==0.7.0, numpy>=1.16.4, scipy>=1.3.0, pandas>=0.21.0

## Outputs

- Latent embeddings file (*_latent_250epoch.tsv of z_mean coordinates on the hypersphere)
- Per-cell log-likelihoods (*_ll_250epoch.tsv)
- Training trace PNG (log-likelihood and KL-divergence per iteration)
- Trained scPhere model checkpoint (*_model_250epoch*) reusable for encoding new data

## How to apply

Load the sparse UMI count matrix with scPhere's read_mtx and transpose it to cell-by-gene form, optionally subsetting to ~2000 variable genes for efficiency; build a 0-based batch vector (concatenating multiple batch factors such as patient/health/location, or n_batch=0 with an artificial all -1 vector). Instantiate SCPHERE with latent_dist='vmf' for spherical embeddings (or 'wn' for hyperbolic), observation_dist='nb', seed=0, and z_dim (2 for visualization; other dims for the RGC/HCL k-NN benchmarks), then train with Trainer at max_epoch=250, mb_size=128, learning_rate=0.001. Save the session with model.save_sess, embed all cells with model.encode(x, batch) writing z_mean to *_latent_250epoch.tsv, and optionally extract log-likelihoods via model.get_log_likelihood for quality assessment. Set batch_invariant=True when the trained model must map unseen individuals (e.g., new patients), since it learns embeddings from gene expression vectors alone.

## Related tools

- **scPhere** (Core VAE model (SCPHERE class) with vMF latent distribution and negative-binomial observation distribution for hyperspherical embedding of scRNA-seq counts) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Deep-learning backend (v1.14) used to train the VAE via the Trainer class)
- **t-SNE** (Baseline 2D embedding method against which scPhere's global k-NN hierarchical-structure accuracy (47.06% RGC, 79.22% HCL) is compared)
- **UMAP** (Baseline 2D embedding method compared against scPhere (52.94% RGC, 83.12% HCL global k-NN accuracy))

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=0, z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0); trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001); trainer.train(); z_mean = model.encode(x, batch)
```

## Evaluation signals

- Global k-NN classification accuracy (nearest cell-type median center) should match reported values within ~±0.5 percentage points: scPhere 73.53% (RGC) / 92.21% (HCL), exceeding t-SNE (47.06%/79.22%) and UMAP (52.94%/83.12%)
- Embedding coordinates in *_latent_250epoch.tsv should lie on the unit hypersphere (unit L2 norm per cell) for latent_dist='vmf'
- Training trace should show increasing log-likelihood and stabilizing KL-divergence over the 250 epochs
- Demo script completes in ~2 minutes and produces the four expected output files (latent tsv, ll tsv, train png, model checkpoint)
- Log-likelihood via model.get_log_likelihood should be computable for held-out cells without error, indicating a valid trained session

## Limitations

- Cell crowding can occur with standard multidimensional-normal-prior VAEs; scPhere mitigates this via the vMF/uniform hypersphere prior, but embeddings on a 3D sphere may need spherical-coordinate conversion (car2sph) for 2D display
- Further studies required to validate scPhere for imputing missing counts and removing ambient RNA contamination
- There are not yet many tools to process data distributed in hyperbolic space, such as efficient k-NN search tools
- Hyperbolic embedding ('wn' latent_dist) is appropriate for hierarchical, branched trajectories, not general-purpose spherical visualization
- TensorFlow 1.14 / tensorflow-probability 0.7.0 pins; Python 3.6+ only

## Evidence

- [intro] we minimize the distortion by embedding cells to a lower-dimensional hypersphere instead of a low-dimensional Euclidean space: "we minimize the distortion by embedding cells to a lower-dimensional hypersphere instead of a low-dimensional Euclidean space"
- [readme] build the model with latent_dist: "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2],
                z_dim=2, latent_dist='vmf', batch_invariant=False, 
                observation_dist='nb', seed=0)"
- [readme] Trainer hyperparameters: "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250,
                  mb_size=128, learning_rate=0.001)"
- [readme] batch-invariant model maps new patients: "The trained model can be used to map new data,
                  e.g., from new patients that have not been seen during training scPhere"
- [readme] normalize k-NN centers on the sphere: "k.centers = k.centers / sqrt(rowSums(k.centers^2)) * 1.15"
