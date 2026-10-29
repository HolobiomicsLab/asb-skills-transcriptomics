---
name: two-stage-adamw-warmup-training
description: Use when transforming a filtered, non-negative single-cell representation
  (e.g., from the non-negative kernel autoencoder) into a low-dimensional compressed
  latent embedding for downstream clustering, visualization, classification, or pseudo-time
  inference.
when_to_use_negative:
- Input is raw scRNA-seq counts that have not been filtered and converted to a non-negative,
  part-based representation by the first scDHA module
- The goal is only clustering or visualization from an existing reduced representation
  — call the higher-level scDHA, scDHA.vis, or scDHA.pt functions instead of re-training
  the second module
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: scDHA
  role: Provides the stacked Bayesian autoencoder (second module) whose modified VAE
    is trained with the two-stage AdamW schedule and repeated latent perturbation
  repo: github.com/duct317/scDHA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:ab53334c8194f5eb22a1b776cd57788d00d2d5ff0a7d90de2fe01a02f5c66256
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scDHA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# two-stage-adamw-warmup-training

## Summary

Trains the Stacked Bayesian Self-learning Network (the second scDHA module) with a two-stage AdamW schedule: a warm-up phase emphasizing reconstruction loss, followed by a phase adding the KL-divergence loss, while repeatedly perturbing the 15-dimensional latent space to reduce overfitting.

## When to use

Use when transforming a filtered, non-negative single-cell representation (e.g., from the non-negative kernel autoencoder) into a low-dimensional compressed latent embedding for downstream clustering, visualization, classification, or pseudo-time inference.

## When NOT to use

- Input is raw scRNA-seq counts that have not been filtered and converted to a non-negative, part-based representation by the first scDHA module
- The goal is only clustering or visualization from an existing reduced representation — call the higher-level scDHA, scDHA.vis, or scDHA.pt functions instead of re-training the second module

## Inputs

- Filtered, non-negative representation (cells x filtered features) from the scDHA non-negative kernel autoencoder
- Latent-space dimensionality setting (default 15 dimensions)

## Outputs

- Latent embedding table with one row per cell and 15 latent dimensions (scdha_latent_embeddings.csv)
- Trained stacked Bayesian Self-learning Network model

## How to apply

Load the filtered, non-negative representation produced by the non-negative kernel autoencoder (the first scDHA module). Build the stacked Bayesian Self-learning Network, a modified Variational Autoencoder with a default 15-dimensional latent space that draws multiple realizations of the latent z via the re-parameterization trick and uses SELU activations. Train with AdamW in two stages: first a warm-up phase emphasizing the reconstruction loss, then a phase that adds the KL-divergence loss so the latent distribution is regularized after reconstruction is stabilized. Repeatedly perturb the compressed latent space during training to learn a more generalized representation of the data and reduce neural-network overfitting. Extract the latent embedding for every cell and write the embeddings table to scdha_latent_embeddings.csv. The result is reproducible by setting a seed, as the scDHA functions support seed-based reproducibility.

## Related tools

- **scDHA** (Provides the stacked Bayesian autoencoder (second module) whose modified VAE is trained with the two-stage AdamW schedule and repeated latent perturbation) — github.com/duct317/scDHA

## Examples

```
install.packages('scDHA'); data <- log2(data + 1); result <- scDHA(data, seed = 1)
```

## Evaluation signals

- Output file scdha_latent_embeddings.csv exists and contains one embedding row per input cell with 15 latent dimensions
- The compressed latent space qualitatively preserves the cell-type structure of the input data
- The KL-divergence loss is added only after the warm-up reconstruction phase, following the two-stage AdamW schedule
- Multiple realizations of z are drawn via the re-parameterization trick and the latent space is perturbed during training (overfitting check)
- Result is reproducible when the seed is fixed (e.g., seed = 1 in scDHA functions)

## Limitations

- Accuracy claims for the modified VAE are stated as expectations rather than demonstrated results in the discussion
- No explicit limitations or caveats section is acknowledged in the discussion
- Comparative performance against scVI is deferred to supplementary materials rather than presented in the main discussion

## Evidence

- [abstract] stacked VAE architecture and low-dimensional projection: "The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed)."
- [abstract] repeated latent perturbation rationale: "we repeatedly perturb the compressed space to learn a more generalized representation of the data"
- [readme] implementation framework for training: "The machine learning framework is changed from Tensorflow to Torch. Torch can be called directly from R. Python environment is no longer required."
