---
name: noise-augmented-gan-generator-training
description: Use when training a cross-modal single-cell GAN where you need stochastic
  generator outputs to estimate mapping uncertainty, e.g. comparing per-gene variance
  for genes with vs without an analogous measured modality.
when_to_use_negative:
- When a deterministic cross-modal mapping suffices and no uncertainty quantification
  is needed — the noise input makes outputs stochastic and adds variance that must
  be averaged over draws
- When the input columns are not approximately in [-10, 10] after preprocessing —
  per the README, further preprocessing is necessary before feeding data into the
  network
- When you have a large number of input modalities to align, since the number of required
  generators grows quadratically and networks must be made small or separately trained
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: scMMGAN
  role: The GAN framework whose generator is noise-augmented with z~N(0,1) to enable
    stochastic mapping and uncertainty measurement
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Framework used for graph construction, sess.run training loop (generator via
    train_op_G, then discriminator via train_op_D), and drawing z
- name: MAGIC
  role: Manifold-smoothing dropout correction applied to inputs before scaling and
    PCA reduction prior to training
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:7559f67682d20cd5a64ee00955010e7bbf1fd83dcca779cec3a4a5668891dcf2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/KrishnaswamyLab/scMMGAN
  tools:
  - MAGIC
  - TensorFlow
  - scMMGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# noise-augmented-gan-generator-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains scMMGAN with a noise-augmented generator G(xi;z), where a draw z from an isotropic N(0,1) is concatenated to the original input, so that repeated stochastic mappings can be used to quantify per-gene mapping uncertainty (e.g., variance of 0.026 for genes with an analogous proteomic measurement vs 1.419 without).

## When to use

Use when training a cross-modal single-cell GAN where you need stochastic generator outputs to estimate mapping uncertainty, e.g. comparing per-gene variance for genes with vs without an analogous measured modality.

## When NOT to use

- When a deterministic cross-modal mapping suffices and no uncertainty quantification is needed — the noise input makes outputs stochastic and adds variance that must be averaged over draws
- When the input columns are not approximately in [-10, 10] after preprocessing — per the README, further preprocessing is necessary before feeding data into the network
- When you have a large number of input modalities to align, since the number of required generators grows quadratically and networks must be made small or separately trained

## Inputs

- Two preprocessed single-cell modalities as x1 and x2 Python variables (e.g. DBIT-seq transcriptomic data reduced to 50 principal components and its proteomic analog), each column mostly in [-10, 10]
- Hyperparameter settings: batch_size, nfilt, learning_rate, lambda_cycle, lambda_correspondence, add_noise, use_bn

## Outputs

- output_fake2: generated data mapping x1 into the second domain; output_fake: generated data mapping x2 into the first domain
- Per-gene mapping variance across repeated stochastic draws (e.g. 0.026 average for genes with analogous proteomic measurement vs 1.419 without), saved as mapping_variance_results.csv
- Ranked gene lists by mapping variance (lowest: Gm5049, Gm37500, Gm33051; highest: GM37686, Rp1)

## How to apply

Build the TensorFlow graph so the generator output is computed as G(xi;z) with z~N(0,1) in R^D concatenated to the original input, enabling stochastic rather than deterministic mappings. Use the standard scMMGAN architectures (generator with 128, 256, 512 neurons; discriminator with 1,024, 512, 256 neurons) and train with the geometry-preserving correspondence loss computed on the first 10-20 diffusion eigenvectors, with the eigenvector sign-correlation check (flip phiG(x) when correlation is below the threshold -0.5) applied before loss computation. Preprocess inputs per the README so each column is mostly in the interval [-10, 10] (e.g. MAGIC dropout correction, zero-centering and unit scaling, reduction to 50 principal components for DBIT-seq data). Set the README's `add_noise` hyperparameter to enable uncertainty measurement; for each cell, repeatedly draw z and compute per-gene variance across stochastic mappings. Decision point: increase `lambda_correspondence` if the aligned mapping retains too little data geometry, and increase `lambda_cycle` if reconstruction retains too little global structure.

## Related tools

- **scMMGAN** (The GAN framework whose generator is noise-augmented with z~N(0,1) to enable stochastic mapping and uncertainty measurement) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Framework used for graph construction, sess.run training loop (generator via train_op_G, then discriminator via train_op_D), and drawing z)
- **MAGIC** (Manifold-smoothing dropout correction applied to inputs before scaling and PCA reduction prior to training)

## Examples

```
z = tf.random.normal([batch_size, D]); output_fake2 = sess.run(G_out, feed_dict={x1_ph: x1_batch, noise_ph: z, add_noise: True})
```

## Evaluation signals

- Reproduced average mapping variance matches reported values: 0.026 for genes with an analogous proteomic measurement and 1.419 for genes without one
- Ranked lowest-variance genes are Gm5049, Gm37500, Gm33051 and highest-variance genes are GM37686 and Rp1
- Generated output is stable across training iterations (per README, further training is unnecessary once output is stable); loss sanity checks printed every 100 steps via training_counter
- Eigenvector sign-correlation check passes: phiG(x) values are flipped by -1 only when correlation is below threshold -0.5 before computing the correspondence loss
- Input columns after preprocessing are mostly within the interval [-10, 10] before feeding the network

## Limitations

- GANs are difficult to train due to adversarial losses, which can lead to instability; the model can deteriorate from effective to ineffective quickly across training iterations
- GANs often suffer from mode collapse because they are not penalized by distribution-level losses; early stopping and the geometric loss help mitigate this only informally
- The geometry-based loss is not completely plug-and-play: a choice of distance between data points is still required

## Evidence
