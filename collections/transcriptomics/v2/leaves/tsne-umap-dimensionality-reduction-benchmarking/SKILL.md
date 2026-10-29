---
name: tsne-umap-dimensionality-reduction-benchmarking
description: Use when comparing dimensionality-reduction embeddings (scPhere vs t-SNE
  vs UMAP) on large scRNA-seq count matrices and you need a quantitative global-structure
  metric, e.g. k-NN accuracy to major cell types, rather than visual inspection.
when_to_use_negative:
- The goal is assessing local neighborhood preservation or fine-grained cluster separation
  rather than hierarchical global structure to major cell types
- The dataset is small (e.g. the demo colon mucosa subset) where the paper's large-dataset
  global-structure benchmarking rationale does not apply
- You need batch-invariant mapping of new samples rather than a global-structure accuracy
  comparison across embedding methods
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scPhere
  role: Trains the vMF (hyperspherical) VAE model and produces the z_mean latent embeddings
    being benchmarked; provides read_mtx for loading sparse count matrices
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: t-SNE
  role: Produces baseline 2D Euclidean embeddings of the same count matrices for accuracy
    comparison
- name: UMAP
  role: Produces baseline 2D Euclidean embeddings of the same count matrices for accuracy
    comparison
- name: TensorFlow
  role: Backend for training the SCPHERE model (tensorflow v1.14.0 with tensorflow-probability
    v0.7.0)
- name: numpy
  role: Loads count matrices and saves z_mean embeddings to *_latent_250epoch.tsv
    via np.savetxt
- name: pandas
  role: Reads batch vectors and cell-type label files
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:2337f7b81a8f2f489fd22877a5daf40073c8e233b6ff1713fb3fad23a0b741a8
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# tsne-umap-dimensionality-reduction-benchmarking

## Summary

Benchmarks t-SNE and UMAP 2D embeddings against scPhere spherical embeddings by computing global k-NN classification accuracies to major cell types on very large scRNA-seq datasets (RGC, 35,699 cells; Human Cell Landscape, 599,926 cells), quantifying how well each method preserves hierarchical global structure.

## When to use

Use when comparing dimensionality-reduction embeddings (scPhere vs t-SNE vs UMAP) on large scRNA-seq count matrices and you need a quantitative global-structure metric, e.g. k-NN accuracy to major cell types, rather than visual inspection.

## When NOT to use

- The goal is assessing local neighborhood preservation or fine-grained cluster separation rather than hierarchical global structure to major cell types
- The dataset is small (e.g. the demo colon mucosa subset) where the paper's large-dataset global-structure benchmarking rationale does not apply
- You need batch-invariant mapping of new samples rather than a global-structure accuracy comparison across embedding methods

## Inputs

- RGC dataset UMI count matrix (35,699 cells, sparse .mtx read via scPhere read_mtx, transposed to cell-by-gene)
- Human Cell Landscape (HCL) dataset UMI count matrix (599,926 cells, sparse .mtx, cell-by-gene)
- Major cell-type label files for RGC and HCL
- Trained scPhere model (SCPHERE, latent_dist='vmf', observation_dist='nb', seed=0)

## Outputs

- scPhere spherical latent embeddings written to *_latent_250epoch.tsv (z_mean from model.encode)
- t-SNE and UMAP 2D embeddings of the same count matrices
- knn_accuracy.csv containing six global k-NN classification accuracies (scPhere/t-SNE/UMAP × RGC/HCL)

## How to apply

Load the RGC and HCL UMI count matrices (sparse matrices read via scPhere's read_mtx and transposed to cell-by-gene form) with their major cell-type label files. Train an SCPHERE model with latent_dist='vmf', observation_dist='nb', seed=0 and an appropriate z_dim using Trainer with max_epoch=250, mb_size=128, learning_rate=0.001, then embed all cells with model.encode(x, batch) into the spherical latent space. Compute baseline 2D t-SNE and UMAP embeddings of the same count matrices. For each embedding, classify each cell to a major cell type by assigning the label of the nearest cell-type median (global k-NN over cell-type medians) and record the accuracy, writing the six values to knn_accuracy.csv. Reference values to match: scPhere 73.53%/92.21%, t-SNE 47.06%/79.22%, UMAP 52.94%/83.12% (RGC/HCL, Fig. 2 / Supplementary Fig. 3).

## Related tools

- **scPhere** (Trains the vMF (hyperspherical) VAE model and produces the z_mean latent embeddings being benchmarked; provides read_mtx for loading sparse count matrices) — https://github.com/klarman-cell-observatory/scPhere
- **t-SNE** (Produces baseline 2D Euclidean embeddings of the same count matrices for accuracy comparison)
- **UMAP** (Produces baseline 2D Euclidean embeddings of the same count matrices for accuracy comparison)
- **TensorFlow** (Backend for training the SCPHERE model (tensorflow v1.14.0 with tensorflow-probability v0.7.0))
- **numpy** (Loads count matrices and saves z_mean embeddings to *_latent_250epoch.tsv via np.savetxt)
- **pandas** (Reads batch vectors and cell-type label files)

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=0, z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0); trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001); trainer.train(); z_mean = model.encode(x, batch)
```

## Evaluation signals

- Each computed accuracy must match its reported reference within ±0.5 percentage points: scPhere 73.53% (RGC) / 92.21% (HCL), t-SNE 47.06% / 79.22%, UMAP 52.94% / 83.12%
- scPhere must rank above both t-SNE and UMAP on both the RGC and HCL datasets
- knn_accuracy.csv contains exactly six accuracy values covering all three methods on both datasets
- Training trace shows monotonically improving log-likelihood and decreasing KL-divergence over the 250 epochs (plot_trace output)
- Spherical embeddings are unit-norm 3D points suitable for PlotSphere visualization (coordinates within [-1, 1])

## Limitations

- scPhere accuracy depends on ~2000 variable genes being selected for efficiency before training on large matrices
- Hyperbolic-space analysis lacks efficient processing tools: 'there are not yet many tools to process data distributed in the hyperbolic space, such as efficient k-NN search tools'
- The demo workflow requires TensorFlow v1.14.0 and tensorflow-probability v0.7.0; Python 3.6+ environment compatibility must be verified

## Evidence

- [readme] Training parameters and model config: "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001)"
- [methods] Model construction with vmf latent space: "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)"
- [methods] Embedding step: "z_mean = model.encode(x, batch)"
- [readme] Sparse matrix loading: [evidence span withheld — anchor status: not_found]
- [intro] Rationale for spherical embedding: "Embedding data distributed on a hypersphere to a Euclidean space introduces significant distortion for commonly used dimensionality reduction tools"
- [results] Benchmark outcome: [evidence span withheld — anchor status: not_found]
