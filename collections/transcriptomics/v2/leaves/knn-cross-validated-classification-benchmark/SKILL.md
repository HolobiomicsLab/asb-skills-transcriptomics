---
name: knn-cross-validated-classification-benchmark
description: Use when you have a low-dimensional cell embedding (e.g., scPhere 'wn'
  latent coordinates) plus per-cell labels such as embryo collection time, and need
  a quantitative time-continuity comparison across embedding methods.
when_to_use_negative:
- Input is raw UMI counts or a feature table rather than an existing cell embedding
  — scPhere must be trained first
- The dataset has real batch vectors (e.g., patient/health/location like the UC epi
  example with n_batch=[30, 3, 2]) — use batch-aware scPhere, not the n_batch=0 artificial-batch
  setup
- Labels are discrete cell types on a sphere ('vmf') rather than a continuous time
  course suited to the hyperbolic 'wn' latent space
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scPhere
  role: Trains the z_dim=2 hyperbolic ('wn' latent_dist) VAE whose embedding is benchmarked;
    encodes cells via model.encode(x, batch)
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend dependency for scPhere training (tensorflow v1.14.0 with tensorflow-probability
    v0.7.0)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:a5eb2c358ce17a9f3f0c2683e75626d075db7d304377bb6dcaf127982a1d0300
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - TensorFlow
  - scPhere
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# knn-cross-validated-classification-benchmark

## Summary

Tenfold-cross-validated k-nearest-neighbor classification of embryo time points on a scPhere hyperbolic (Poincaré disk) embedding, benchmarked against t-SNE, UMAP, and PHATE, to test whether the embedding preserves time-continuity structure.

## When to use

Use when you have a low-dimensional cell embedding (e.g., scPhere 'wn' latent coordinates) plus per-cell labels such as embryo collection time, and need a quantitative time-continuity comparison across embedding methods.

## When NOT to use

- Input is raw UMI counts or a feature table rather than an existing cell embedding — scPhere must be trained first
- The dataset has real batch vectors (e.g., patient/health/location like the UC epi example with n_batch=[30, 3, 2]) — use batch-aware scPhere, not the n_batch=0 artificial-batch setup
- Labels are discrete cell types on a sphere ('vmf') rather than a continuous time course suited to the hyperbolic 'wn' latent space

## Inputs

- 86,024-cell C. elegans embryonic UMI count matrix (GSE126954), cells x variable genes, dense
- Per-cell embryo collection time labels (classification targets)
- scPhere hyperbolic latent coordinates (z_dim=2, latent_dist='wn')
- Comparison embeddings of the same cells: t-SNE, UMAP, PHATE

## Outputs

- worm_embryo_hyperbolic_latent_250epoch.tsv (model.encode latent coordinates via np.savetxt)
- Pearson correlation between distance-to-Poincaré-origin and embryo time (target r = 0.55)
- knn_accuracy_comparison.csv (tenfold-CV mean k-NN accuracies for scPhere hyperbolic vs t-SNE, UMAP, PHATE)

## How to apply

First train scPhere on the 86,024-cell GSE126954 C. elegans embryonic UMI matrix with SCPHERE(n_gene=x.shape[1], n_batch=0, z_dim=2, latent_dist='wn', observation_dist='nb', seed=0), using the artificial batch vector batch = np.zeros(x.shape[0]) * -1 since the dataset has no batch factors. Train with Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001), then embed via z_mean = model.encode(x, batch) and save with np.savetxt to worm_embryo_hyperbolic_latent_250epoch.tsv. Compute each cell's distance to the Poincaré disk origin and its Pearson correlation with embryo time (expected r = 0.55) as a pseudotime check before benchmarking. Run tenfold-cross-validated k-NN classification of embryo time points on the hyperbolic embedding and on t-SNE, UMAP, and PHATE embeddings of the same cells, recording mean accuracies to knn_accuracy_comparison.csv; a caveat is that efficient k-NN tools for hyperbolic space are limited, so metric choice must be handled explicitly.

## Related tools

- **scPhere** (Trains the z_dim=2 hyperbolic ('wn' latent_dist) VAE whose embedding is benchmarked; encodes cells via model.encode(x, batch)) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend dependency for scPhere training (tensorflow v1.14.0 with tensorflow-probability v0.7.0))

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=0, z_dim=2, latent_dist='wn', observation_dist='nb', seed=0); trainer = Trainer(model=model, x=x, batch_id=np.zeros(x.shape[0])*-1, max_epoch=250, mb_size=128, learning_rate=0.001); trainer.train(); z_mean = model.encode(x, batch)
```

## Evaluation signals

- Pearson correlation of distance-to-Poincaré-disk-origin vs embryo collection time approximates the reported r = 0.55
- Latent file worm_embryo_hyperbolic_latent_250epoch.tsv has one row per cell (86,024 rows) with 2 coordinate columns
- Mean tenfold-CV k-NN accuracies for the hyperbolic embedding are comparable to Supplementary Fig. 18c
- Training trace (plot_trace of log_likelihood and kl_divergence) converges over 250 epochs without divergence
- scPhere k-NN accuracy is not degraded relative to t-SNE, UMAP, and PHATE baselines on the same cells

## Limitations

- There are not yet many tools to process data distributed in the hyperbolic space, such as efficient k-NN search tools
- Dataset without batch vectors requires an artificial batch = np.zeros(x.shape[0]) * -1 vector, which does not influence results but is needed to run scPhere
- Training used TensorFlow v1.14.0 / tensorflow-probability v0.7.0 on Python 3.6, so newer TF versions may be incompatible

## Evidence

- [readme] latent_dist: 'vmf' for spherical latent spaces, and 'wn' for hyperbolic latent spaces: "latent_dist: 'vmf' for spherical latent spaces, and 'wn' for hyperbolic latent spaces"
- [readme] trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001): "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250,
                  mb_size=128, learning_rate=0.001)"
- [readme] For the cases there are no batch vectors, we can set n_batch=0: "# For the cases there are no batch vectors, we can set n_batch=0, 
# and create an artificial batch vector (just for running scPhere, 
# the batch vector will not influence the results)"
- [methods] z_mean = model.encode(x, batch): "z_mean = model.encode(x, batch)"
- [discussion] there are not yet many tools to process data distributed in the hyperbolic space, such as efficient k-NN search tools: "there are not yet many tools to process data distributed in the hyperbolic space, such as efficient k-NN search tools"
