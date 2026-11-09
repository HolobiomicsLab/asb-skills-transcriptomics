---
name: reparameterization-trick-sampling
description: Use when building or re-running the scDHA Stacked Bayesian Self-learning
  Network stage that converts the filtered non-negative kernel autoencoder representation
  into a low-dimensional latent embedding.
when_to_use_negative:
- Input is raw unfiltered scRNA-seq counts — the re-parameterized Bayesian stage expects
  the non-negative, filtered representation produced by the first-module kernel autoencoder
- The goal is only gene/component filtering; insignificant genes or components are
  removed by the first module's encoder weight distribution, not by latent perturbation
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDHA
  role: R package whose scDHA function runs the two-module hierarchical autoencoder,
    including the modified VAE second module that draws multiple latent realizations
    via the re-parameterization trick; implemented on Torch
  repo: github.com/duct317/scDHA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:484bc0bbb01116e97f73d4376c85c90173656d270e56f341c88c537062d38b3a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scDHA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# reparameterization-trick-sampling

## Summary

Samples multiple realizations of the latent vector z via the re-parameterization trick inside scDHA's second module (a stacked Bayesian autoencoder built on the VAE), so the compressed 15-dimensional space is repeatedly perturbed to learn a more generalized representation and reduce neural-network overfitting.

## When to use

Use when building or re-running the scDHA Stacked Bayesian Self-learning Network stage that converts the filtered non-negative kernel autoencoder representation into a low-dimensional latent embedding.

## When NOT to use

- Input is raw unfiltered scRNA-seq counts — the re-parameterized Bayesian stage expects the non-negative, filtered representation produced by the first-module kernel autoencoder
- The goal is only gene/component filtering; insignificant genes or components are removed by the first module's encoder weight distribution, not by latent perturbation

## Inputs

- Filtered, non-negative cell-by-feature representation output from the scDHA non-negative kernel autoencoder (rows as cells/samples, columns as genes)
- Choice of latent dimensionality (default 15 dimensions)
- Random seed for reproducible sampling

## Outputs

- 15-dimensional latent embedding per cell (e.g. written to scdha_latent_embeddings.csv, one row per input cell)
- Perturbation-robust compressed representation usable for clustering, visualization, classification, and pseudo-time inference

## How to apply

Load the filtered, non-negative representation produced by the scDHA non-negative kernel autoencoder (first module) and feed it into the second module, a stacked Bayesian autoencoder implemented as a modified Variational Autoencoder with a default 15-dimensional latent space. Instead of a single point estimate of z, draw multiple realizations of z via the re-parameterization trick and apply SELU activations throughout the network. Train with AdamW in two stages: a warm-up phase emphasizing the reconstruction loss, followed by a phase that adds the KL-divergence loss. Repeatedly perturb the compressed latent space during training to learn a more generalized representation of the data and reduce overfitting. The scDHA R package implements this on the Torch backend, so no separate Python/TensorFlow environment is required, and results are reproducible by fixing the seed. Finally extract one latent embedding per cell and confirm the 15-dimensional compressed space qualitatively preserves cell-type structure.

## Related tools

- **scDHA** (R package whose scDHA function runs the two-module hierarchical autoencoder, including the modified VAE second module that draws multiple latent realizations via the re-parameterization trick; implemented on Torch) — github.com/duct317/scDHA

## Examples

```
data <- log2(data + 1); result <- scDHA(data, seed = 1)
```

## Evaluation signals

- The output embeddings file exists and contains exactly one row per input cell with 15 latent dimensions
- The compressed space qualitatively preserves the cell-type structure of the input data (e.g. via downstream clustering vs. labels, such as adjusted Rand index on the Goolam dataset)
- Training followed the two-stage AdamW schedule (reconstruction loss warm-up, then KL-divergence term added) without NaNs or divergence
- Re-running with the same seed reproduces the same latent embeddings, as the scDHA functions are reproducible by setting the seed

## Limitations

- Accuracy claims for the modified VAE second module are stated as expectations ('are expected to be more efficient than other autoencoders') rather than demonstrated in the discussion
- The comparative claim against scVI is deferred to supplementary materials rather than presented in the main discussion
- Latent dimensionality defaults to 15; deviating from the validated default requires re-validating that cell-type structure is preserved

## Evidence

- [abstract] The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed).: "The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed)."
- [abstract] Repeated perturbation rationale: "we repeatedly perturb the compressed space to learn a more generalized representation of the data"
- [intro] First-module input representation: "The first module is a non-negative kernel autoencoder that provides a non-negative, part-based representation of the data."
- [readme] Torch backend and reproducibility: "The machine learning framework is changed from Tensorflow to Torch. Torch can be called directly from R. Python environment is no longer required."
- [readme] Seed-based reproducibility of scDHA functions: [evidence span withheld — anchor status: external_doc]
