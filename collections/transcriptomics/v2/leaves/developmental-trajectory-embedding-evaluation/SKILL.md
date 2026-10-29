---
name: developmental-trajectory-embedding-evaluation
description: Use when a single-cell RNA-seq dataset represents a time-course (e.g.,
  86,024-cell C. elegans embryo GSE126954) and you need to test whether a hyperbolic
  embedding's radial coordinate (distance-to-origin) recapitulates developmental time
  as pseudotime.
when_to_use_negative:
- The dataset has multiple biological/technical batch vectors that should be modeled
  — then use n_batch matching the batch vectors (e.g. [30, 3, 2]) or batch-invariant
  scPhere, not the n_batch=0 single-sample setup
- The goal is a spherical (vMF) embedding for cell-type separation rather than a temporal/hierarchical
  trajectory, where latent_dist='vmf' is appropriate
- You need imputation of missing counts or ambient RNA removal — the article states
  these are unvalidated in scPhere
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPhere
  role: Trains the hyperbolic (latent_dist='wn') VAE and encodes cells into the Poincaré
    disk whose radial coordinate is evaluated as pseudotime
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend for the scPhere deep generative model (tensorflow==1.14 with tensorflow-probability
    0.7.0 per the README)
  repo: https://github.com/klarman-cell-observatory/scPhere
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:2a927f3aa67c506a77dede4d5ef9be086b31b018c64ed71b9c604ee01ff33cc2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - TensorFlow
  - scPhere
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# developmental-trajectory-embedding-evaluation

## Summary

Evaluates whether a scPhere hyperbolic (Poincaré disk) embedding captures developmental time by correlating cell distances to the latent origin with collection/embryonic time, and by benchmarking time-continuity via tenfold-cross-validated k-NN against t-SNE, UMAP, and PHATE.

## When to use

Use when a single-cell RNA-seq dataset represents a time-course (e.g., 86,024-cell C. elegans embryo GSE126954) and you need to test whether a hyperbolic embedding's radial coordinate (distance-to-origin) recapitulates developmental time as pseudotime.

## When NOT to use

- The dataset has multiple biological/technical batch vectors that should be modeled — then use n_batch matching the batch vectors (e.g. [30, 3, 2]) or batch-invariant scPhere, not the n_batch=0 single-sample setup
- The goal is a spherical (vMF) embedding for cell-type separation rather than a temporal/hierarchical trajectory, where latent_dist='vmf' is appropriate
- You need imputation of missing counts or ambient RNA removal — the article states these are unvalidated in scPhere

## Inputs

- GSE126954 C. elegans embryonic scRNA-seq UMI count matrix (86,024 cells x variable genes), e.g. .mtx read via scphere.util.util.read_mtx
- Vector of embryo collection times per cell for correlation/k-NN evaluation
- Pre-trained scPhere hyperbolic model or per-cell latent coordinates TSV (e.g. worm_embryo_hyperbolic_latent_250epoch.tsv)

## Outputs

- Per-cell latent coordinates in the Poincaré disk (2D, saved as .tsv via np.savetxt)
- Pearson correlation between distance-to-origin and embryo time (expected ~0.55)
- k-NN accuracy comparison CSV (scPhere hyperbolic vs t-SNE, UMAP, PHATE) for the tenfold cross-validation

## How to apply

Train a scPhere SCPHERE model on the UMI count matrix with n_batch=0 and an artificial batch vector (batch = np.zeros(x.shape[0]) * -1), z_dim=2, latent_dist='wn' for the hyperbolic (wrapped normal) latent space, observation_dist='nb', and seed=0, then fit with the Trainer using max_epoch=250, mb_size=128, learning_rate=0.001. Embed all cells via model.encode(x, batch) and save the latent coordinates with np.savetxt. Compute each cell's Euclidean distance to the Poincaré disk origin (the hyperbolic radial coordinate) and correlate it with embryo collection time using Pearson correlation. As an independent check, run tenfold-cross-validated k-nearest-neighbor classification of embryo time points on the hyperbolic embedding and on t-SNE, UMAP, and PHATE embeddings, recording mean accuracies. The rationale, from Ding & Regev (2021), is that hierarchical/branched developmental trajectories are better represented in Lorentz hyperbolic space than in Euclidean space, and the radial coordinate of the Poincaré disk acts as a pseudotime signal.

## Related tools

- **scPhere** (Trains the hyperbolic (latent_dist='wn') VAE and encodes cells into the Poincaré disk whose radial coordinate is evaluated as pseudotime) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend for the scPhere deep generative model (tensorflow==1.14 with tensorflow-probability 0.7.0 per the README)) — https://github.com/klarman-cell-observatory/scPhere

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=0, z_dim=2, latent_dist='wn', observation_dist='nb', seed=0); trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001); trainer.train(); z_mean = model.encode(x, batch)
```

## Evaluation signals

- Pearson correlation between cells' distance to the Poincaré disk origin and embryo collection time should approximate the reported r = 0.55
- Latent coordinates TSV has shape (number_of_cells x 2) and finite values strictly inside the disk for z_dim=2 'wn' embeddings
- Tenfold-cross-validated k-NN accuracies on the hyperbolic embedding should be comparable to those reported in Supplementary Fig. 18c, relative to t-SNE, UMAP, and PHATE
- Training trace (log-likelihood and KL-divergence plots) converges over the 250 epochs at mb_size=128, learning_rate=0.001
- seed=0 reproducibility: re-running the pipeline yields the same latent coordinates and correlation value

## Limitations

- Lack of tools for processing data in hyperbolic space, e.g. efficient k-NN search tools, so k-NN comparison is done via cross-validated classification rather than native hyperbolic-geodesic neighbors
- Results depend on hyperparameters (max_epoch=250, mb_size=128, learning_rate=0.001, seed=0); scPhere was tested with Tensorflow v1.14.0 and tensorflow-probability v0.7.0 on Python 3.6
- Interpreting distance-to-origin as pseudotime is an empirical correlation (r = 0.55), not a calibrated time estimate

## Evidence

- [intro] embed cells to the hyperbolic space of the Lorentz model and visualize the embedding in a Poincaré disk: "we embed cells to the hyperbolic space of the Lorentz model and visualize the embedding in a Poincaré disk"
- [methods] trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001): "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250,
                  mb_size=128, learning_rate=0.001)"
- [supplementary] distances to the Poincaré disk origin correlate with embryonic time (r = 0.55), supporting distance-to-origin as pseudotime: [evidence span withheld — anchor status: not_found]
- [readme] no batch vectors: n_batch=0 with artificial batch vector: "For the cases there are no batch vectors, we can set n_batch=0, 
# and create an artificial batch vector"
- [discussion] few tools exist for hyperbolic-space data such as efficient k-NN search: "there are not yet many tools to process data distributed in the hyperbolic space, such as efficient k-NN search tools"
