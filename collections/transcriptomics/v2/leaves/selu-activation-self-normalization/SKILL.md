---
name: selu-activation-self-normalization
description: Use when implementing or re-implementing the scDHA second module (stacked
  Bayesian autoencoder) or a modified VAE that projects filtered single-cell data
  onto a 15-dimensional latent space and needs self-normalizing activations for stable
  AdamW training.
when_to_use_negative:
- When the goal is the first scDHA module — the non-negative kernel autoencoder —
  rather than the low-dimensional compressed projection of the second module
- When only visualization or pseudo-time on an existing low-dimensional embedding
  is needed (use scDHA.vis or scDHA.pt on already-computed dimension reduction output
  instead of retraining the VAE)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDHA
  role: R/Torch package whose second module (Stacked Bayesian Self-learning Network,
    a modified VAE) uses SELU activations to project data onto a 15-dimensional compressed
    space
  repo: https://github.com/duct317/scDHA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:d0642ace36dc3f2aef4ceddec7e328de7059c0d9deea37b09331e7d1571f1858
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scDHA
  tools:
  - scDHA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# selu-activation-self-normalization

## Summary

A design pattern from the scDHA Stacked Bayesian Self-learning Network in which SELU (scaled exponential linear unit) activations are used inside a modified Variational Autoencoder so the low-dimensional latent space self-normalizes across layers, supporting stable training of the 15-dimensional compressed representation without explicit batch normalization.

## When to use

Use when implementing or re-implementing the scDHA second module (stacked Bayesian autoencoder) or a modified VAE that projects filtered single-cell data onto a 15-dimensional latent space and needs self-normalizing activations for stable AdamW training.

## When NOT to use

- When the goal is the first scDHA module — the non-negative kernel autoencoder — rather than the low-dimensional compressed projection of the second module
- When only visualization or pseudo-time on an existing low-dimensional embedding is needed (use scDHA.vis or scDHA.pt on already-computed dimension reduction output instead of retraining the VAE)

## Inputs

- Filtered, non-negative representation from the scDHA non-negative kernel autoencoder (rows = cells, columns = components/genes)
- Log-transformed scRNA-seq count matrix (e.g. log2(data + 1) as in the Goolam example)

## Outputs

- 15-dimensional latent embedding per cell (written to scdha_latent_embeddings.csv in the reference workflow)
- Result object from scDHA() containing the compressed representation and cluster assignments

## How to apply

Take the non-negative, filtered representation produced by scDHA's first module (the non-negative kernel autoencoder) and build a stacked Bayesian Self-learning Network: a modified VAE with a default 15-dimensional latent space that draws multiple realizations of z via the re-parameterization trick and uses SELU activations. Train with AdamW in two stages — a warm-up phase emphasizing reconstruction loss, then a phase adding the KL-divergence loss — relying on SELU self-normalization to keep activations centered across repeated realizations of the latent space. Repeatedly perturb the compressed space to learn a more generalized representation and reduce neural-network overfitting. In the released scDHA package the underlying ML framework is Torch (callable directly from R; no Python environment required), so the activation choice is realized inside the Torch model rather than a Tensorflow implementation.

## Related tools

- **scDHA** (R/Torch package whose second module (Stacked Bayesian Self-learning Network, a modified VAE) uses SELU activations to project data onto a 15-dimensional compressed space) — https://github.com/duct317/scDHA

## Examples

```
library(scDHA); data <- log2(data + 1); result <- scDHA(data, seed = 1)
```

## Evaluation signals

- Output embeddings table contains one row per input cell with exactly 15 latent dimensions
- Compressed space qualitatively preserves the cell-type structure of the input data (e.g. adjusted Rand Index of result$cluster vs labels, as in the Goolam example)
- Training proceeded through the two-stage AdamW schedule (reconstruction-only warm-up, then reconstruction + KL divergence) without divergence
- Latent embeddings are reproducible when the seed is fixed (the scDHA functions are reproducible by setting seed)
- Latent values remain normalized (self-normalizing property of SELU) across repeated perturbations of the compressed space

## Limitations

- Accuracy of the modified VAE second module is stated as an expectation ('expected to be more efficient than other autoencoders') rather than demonstrated in the main discussion
- The comparative claim against scVI for the second module is deferred to supplementary materials (Supplementary Section 6 and Fig. 25–32)
- The package requires libtorch; it must be installed via torch::install_torch() when the library is loaded

## Evidence

- [abstract] The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed).: "The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed)."
- [other] a modified VAE with a default 15-dimensional latent space that draws multiple realizations of z via the re-parameterization trick and uses SELU activations: [evidence span withheld — anchor status: not_found]
- [abstract] we repeatedly perturb the compressed space to learn a more generalized representation of the data: "we repeatedly perturb the compressed space to learn a more generalized representation of the data"
- [readme] The machine learning framework is changed from Tensorflow to Torch. Torch can be called directly from R. Python environment is no longer required.: "The machine learning framework is changed from Tensorflow to Torch. Torch can be called directly from R."
