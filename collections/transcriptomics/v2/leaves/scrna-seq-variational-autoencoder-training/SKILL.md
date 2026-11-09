---
name: scrna-seq-variational-autoencoder-training
description: Use when a UMI count matrix (cells x genes) needs low-dimensional embedding
  for visualization, batch correction across multiple batch factors (e.g., 30 patients
  x 3 health statuses x 2 locations), or mapping of held-out individuals.
when_to_use_negative:
- The goal is imputing missing counts or removing ambient RNA contamination — the
  article states further studies are required to validate scPhere for these tasks
- The data are already embedded or reduced (e.g., a precomputed embedding or feature
  table) rather than raw UMI counts
- An efficient k-NN search in hyperbolic space is required as the primary deliverable
  — the article notes there are not yet many tools to process data distributed in
  the hyperbolic space, such as efficient k-NN search tools
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scPhere
  role: Deep generative model (VAE) providing SCPHERE, Trainer, read_mtx, encode,
    get_log_likelihood for spherical/hyperbolic embedding
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend (v1.14.0) for VAE training via minibatch gradient descent
- name: tensorflow-probability
  role: v0.7.0; supplies vMF and wrapped-normal latent distributions and negative
    binomial observation distribution
- name: Harmony
  role: Comparator batch-correction method for k-NN/logistic-regression accuracy benchmarking
- name: LIGER
  role: Comparator batch-correction method in the UC dataset benchmark
- name: Seurat3 CCA
  role: Comparator batch-correction method in the UC dataset benchmark
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:f16602083c99677b3add1bc8f066a1a4517744e83e1a8ac3e583047e3389f6f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - Harmony
  - LIGER
  - Seurat3 CCA
  - TensorFlow
  - matplotlib
  - numpy
  - pandas
  - scPhere
  - scipy
  - tensorflow-probability
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# scrna-seq-variational-autoencoder-training

## Summary

Train the scPhere variational autoencoder to embed scRNA-seq UMI count matrices into hyperspherical (von Mises–Fisher) or hyperbolic (wrapped normal) latent spaces, handling multilevel batch vectors and optional batch-invariant mapping of new patients.

## When to use

Use when a UMI count matrix (cells x genes) needs low-dimensional embedding for visualization, batch correction across multiple batch factors (e.g., 30 patients x 3 health statuses x 2 locations), or mapping of held-out individuals.

## When NOT to use

- The goal is imputing missing counts or removing ambient RNA contamination — the article states further studies are required to validate scPhere for these tasks
- The data are already embedded or reduced (e.g., a precomputed embedding or feature table) rather than raw UMI counts
- An efficient k-NN search in hyperbolic space is required as the primary deliverable — the article notes there are not yet many tools to process data distributed in the hyperbolic space, such as efficient k-NN search tools

## Inputs

- Sparse UMI count matrix (.mtx, genes x cells, e.g., uc_epi.mtx), transposed to cells x genes
- Batch annotation TSVs with 0-based integer factors (uc_epi_batch_patient.tsv, uc_epi_batch_health.tsv, uc_epi_batch_location.tsv)
- Cell-type label TSV (e.g., uc_epi_celltype.tsv) for downstream classification evaluation
- GSE126954 C. elegans embryonic UMI count matrix for hyperbolic trajectory embedding

## Outputs

- Trained scPhere model checkpoint (e.g., uc_epi_model_250epoch via model.save_sess)
- Latent embedding TSV of cell coordinates on the sphere/Poincaré disk (e.g., uc_epi_latent_250epoch.tsv)
- Per-cell log-likelihood TSV (e.g., uc_epi_ll_250epoch.tsv)
- Training trace PNG of log-likelihood and KL-divergence per iteration
- k-NN / logistic-regression classification accuracy comparisons against t-SNE, UMAP, PHATE, Harmony, LIGER, Seurat3 CCA

## How to apply

Load the sparse count matrix with scPhere's read_mtx and transpose to cells x genes (e.g., the UC colon mucosa epithelial matrix uc_epi.mtx from Single Cell Portal SCP551), then build a SCPHERE model with n_gene equal to the gene count, n_batch=[30, 3, 2] for the three 0-based batch vectors (patient, health, location), z_dim=2, latent_dist='vmf' (spherical) or 'wn' (hyperbolic), observation_dist='nb', and seed=0; for datasets without batch factors set n_batch=0 with an artificial all -1 batch vector. Train with Trainer using max_epoch=250, mb_size=128, learning_rate=0.001, monitoring log-likelihood and KL-divergence traces via plot_trace. Set batch_invariant=True so only gene expression feeds the encoder, enabling mapping of new (held-out) patients unseen during training. Save the model with model.save_sess, embed cells with model.encode(x, batch), and compute per-cell log-likelihoods with model.get_log_likelihood. Benchmark against t-SNE/UMAP/PHATE embeddings or batch-correction methods (Harmony, LIGER, Seurat3 CCA) using k-NN and logistic-regression cell-type classification accuracy.

## Related tools

- **scPhere** (Deep generative model (VAE) providing SCPHERE, Trainer, read_mtx, encode, get_log_likelihood for spherical/hyperbolic embedding) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend (v1.14.0) for VAE training via minibatch gradient descent)
- **tensorflow-probability** (v0.7.0; supplies vMF and wrapped-normal latent distributions and negative binomial observation distribution)
- **Harmony** (Comparator batch-correction method for k-NN/logistic-regression accuracy benchmarking)
- **LIGER** (Comparator batch-correction method in the UC dataset benchmark)
- **Seurat3 CCA** (Comparator batch-correction method in the UC dataset benchmark)

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0); Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001).train()
```

## Evaluation signals

- All four scPhere output files exist: trained model, latent TSV, log-likelihood TSV, and training-trace PNG in the output directory
- On the UC dataset, scPhere's k-NN and logistic-regression cell-type classification accuracies are at least as high as Harmony, LIGER, and Seurat3 CCA (Fig. 3d–f)
- Batch-invariant model trained on 18 UC patients reaches k-NN (k=25) test accuracies near 0.79/0.83/0.82 for stromal/epithelial/immune cells, matching the all-patient model's 0.80/0.87/0.80
- C. elegans hyperbolic embedding: Pearson correlation between Poincaré-disk distance-to-origin and embryo collection time approximates the reported r = 0.55
- vMF spherical embeddings show no central cell crowding, unlike normal-prior VAE embeddings, with Silhouette-score rankings matching Supplementary Figs. 1–3

## Limitations

- Further studies are required to explore scPhere's abilities in imputing missing counts and removing ambient RNA contamination
- There are not yet many tools to process data distributed in hyperbolic space, such as efficient k-NN search tools
- Estimating the uncertainty of aligning cells from batches requires future supervised information; multimodal integration, perturbation modeling, and semi-supervised cell-type annotation are unimplemented extensions
- TensorFlow v1.14.0 and tensorflow-probability v0.7.0 are required; scPhere was tested on Python 3.6.7 on Linux and macOS

## Evidence

- [methods] Model construction with multilevel batch vectors and key parameters: "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)"
- [methods] Training hyperparameters: "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001)"
- [readme] README usage and data source: "A demo script (`example/script/demo.py`) shows how to run scPhere using data in the folder `example/data`."
- [readme] No-batch-vector fallback: "batch = np.zeros(x.shape[0]) * -1"
- [intro] Motivation for spherical latent space: "they assume a multidimensional normal prior for the low-dimensional latent variables, which unfortunately encourages the low-dimensional representations of all cells to the group in the center of the"
- [discussion] Benchmark advantage over competing batch-correction methods: "scPhere performed better than state-of-the-art batch-correction methods such as Harmony, Seurat3 CCA, LIGER"
- [abstract] Batch-invariant mapping capability: "generating batch-invariant embeddings to map data from new individuals, identifies cell types affected by biological variables, infers cells’ spatial positions in pre-defined biological specimens,"
