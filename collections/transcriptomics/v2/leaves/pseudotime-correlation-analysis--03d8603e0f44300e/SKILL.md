---
name: pseudotime-correlation-analysis
description: Use when a hyperbolic scPhere embedding of a time-course single-cell
  dataset (e.g., C. elegans embryo GSE126954, 86,024 cells) must be checked for whether
  latent geometry encodes progression time as a pseudotime signal.
when_to_use_negative:
- The embedding was produced with latent_dist='vmf' on a hypersphere (e.g., the UC
  epithelium demo); distance-to-origin pseudotime logic is specific to the hyperbolic
  'wn' latent space.
- The dataset has no time or ordering metadata, so neither the Pearson correlation
  nor time-point k-NN classification can be computed.
- The goal is batch correction or mapping cells from new patients rather than validating
  temporal structure in a single developmental time course.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPhere
  role: Trains the deep generative model with hyperbolic 'wn' latent space, encodes
    cells to the Poincaré disk, and supplies the latent coordinates whose origin distances
    serve as the pseudotime signal.
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: TensorFlow
  role: Backend dependency for scPhere (tensorflow==1.14 with tensorflow-probability==0.7.0)
    used to train the VAE and run model.encode.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:0b93f55f533da8aba6b7304ae2b417c10c554d3aac48994456f0e3e037dfff79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - TensorFlow
  - scPhere
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pseudotime-correlation-analysis

## Summary

Validate that a scPhere hyperbolic (Poincaré disk) embedding captures developmental time by correlating each cell's distance to the latent-space origin with embryo collection time, and benchmarking the embedding against t-SNE, UMAP, and PHATE via cross-validated k-NN classification of time points.

## When to use

Use when a hyperbolic scPhere embedding of a time-course single-cell dataset (e.g., C. elegans embryo GSE126954, 86,024 cells) must be checked for whether latent geometry encodes progression time as a pseudotime signal.

## When NOT to use

- The embedding was produced with latent_dist='vmf' on a hypersphere (e.g., the UC epithelium demo); distance-to-origin pseudotime logic is specific to the hyperbolic 'wn' latent space.
- The dataset has no time or ordering metadata, so neither the Pearson correlation nor time-point k-NN classification can be computed.
- The goal is batch correction or mapping cells from new patients rather than validating temporal structure in a single developmental time course.

## Inputs

- GSE126954 C. elegans embryonic UMI count matrix (86,024 cells x variable genes, dense matrix)
- Per-cell embryo collection time labels (numeric time points for pseudotime correlation and k-NN targets)
- Trained scPhere SCPHERE model (n_batch=0, z_dim=2, latent_dist='wn', observation_dist='nb', seed=0)

## Outputs

- worm_embryo_hyperbolic_latent_250epoch.tsv (per-cell Poincaré disk latent coordinates)
- Pearson correlation coefficient between distance-to-hyperbolic-origin and embryo time (expected ~0.55)
- knn_accuracy_comparison.csv (mean tenfold-cross-validated k-NN accuracies for scPhere hyperbolic vs t-SNE, UMAP, PHATE embeddings)

## How to apply

Train a scPhere SCPHERE model on the GSE126954 C. elegans embryonic UMI count matrix with n_batch=0 (artificial batch = np.zeros(x.shape[0]) * -1), z_dim=2, latent_dist='wn' (wrapped-normal hyperbolic latent space), observation_dist='nb', and seed=0, then train with Trainer(max_epoch=250, mb_size=128, learning_rate=0.001). Embed all cells via model.encode(x, batch) and save to worm_embryo_hyperbolic_latent_250epoch.tsv. Compute each cell's distance to the Poincaré disk origin and its Pearson correlation with embryo collection time — the reported r = 0.55 supports distance-to-origin as a pseudotime measure. As an orthogonal check, run tenfold-cross-validated k-NN classification of embryo time points on the hyperbolic embedding and on t-SNE, UMAP, and PHATE embeddings, recording mean accuracies to knn_accuracy_comparison.csv. The rationale is that the Lorentz-model hyperbolic geometry is suited to hierarchical, branched developmental trajectories, so radial position in the Poincaré disk should track developmental progression.

## Related tools

- **scPhere** (Trains the deep generative model with hyperbolic 'wn' latent space, encodes cells to the Poincaré disk, and supplies the latent coordinates whose origin distances serve as the pseudotime signal.) — https://github.com/klarman-cell-observatory/scPhere
- **TensorFlow** (Backend dependency for scPhere (tensorflow==1.14 with tensorflow-probability==0.7.0) used to train the VAE and run model.encode.)

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=0, z_dim=2, latent_dist='wn', observation_dist='nb', seed=0); trainer = Trainer(model=model, x=x, batch_id=np.zeros(x.shape[0])*-1, max_epoch=250, mb_size=128, learning_rate=0.001); trainer.train(); z_mean = model.encode(x, batch)
```

## Evaluation signals

- Pearson correlation between per-cell distance to the Poincaré disk origin and embryo collection time approximates the reported r = 0.55.
- Tenfold-cross-validated k-NN accuracies on embryo time points are comparable to those in Supplementary Fig. 18c across scPhere, t-SNE, UMAP, and PHATE embeddings.
- Latent file worm_embryo_hyperbolic_latent_250epoch.tsv contains one 2D row per cell (86,024 rows), saved via np.savetxt as in the README workflow.
- Model configuration matches the recipe: n_batch=0 with batch = np.zeros(x.shape[0]) * -1, z_dim=2, latent_dist='wn', observation_dist='nb', seed=0, max_epoch=250, mb_size=128, learning_rate=0.001.
- Trainer training trace (log-likelihood and KL-divergence per iteration) converges over the 250 epochs, as plottable with plot_trace.

## Limitations

- There are not yet many tools to process data distributed in the hyperbolic space, such as efficient k-NN search tools, complicating downstream analyses in the Poincaré disk.
- The Pearson correlation of ~0.55 captures monotone radial ordering but not fine-grained per-cell time precision; it is an embedding-level rather than per-cell validation.
- Results depend on reproducibility settings (seed=0, TensorFlow v1.14.0, tensorflow-probability v0.7.0); scPhere was tested on Python 3.6.7 and alternative TF versions may not reproduce the embedding.
- Further studies are required to validate scPhere for imputing missing counts in scRNA-seq data and removing ambient RNA contamination, so the embedding should not be reused for imputation.

## Evidence

- [intro] we embed cells to the hyperbolic space of the Lorentz model and visualize the embedding in a Poincaré disk: "we embed cells to the hyperbolic space of the Lorentz model and visualize the embedding in a Poincaré disk"
- [readme] For the cases there are no batch vectors, we can set n_batch=0, and create an artificial batch vector: "For the cases there are no batch vectors, we can set n_batch=0, and create an artificial batch vector (just for running scPhere"
- [readme] latent_dist: 'vmf' for spherical latent spaces, and 'wn' for hyperbolic latent spaces: "latent_dist: 'vmf' for spherical latent spaces, and 'wn' for hyperbolic latent spaces"
- [methods] trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250, mb_size=128, learning_rate=0.001): "trainer = Trainer(model=model, x=x, batch_id=batch, max_epoch=250,
                  mb_size=128, learning_rate=0.001)"
- [discussion] there are not yet many tools to process data distributed in the hyperbolic space, such as efficient k-NN search tools: "there are not yet many tools to process data distributed in the hyperbolic space, such as efficient k-NN search tools"
