---
name: gene-mapping-variance-quantification
description: Use when a trained noise-augmented GAN mapping between transcriptomic
  and proteomic domains is available and you need to quantify which genes' generated
  values are stable vs. unstable under stochastic z draws.
when_to_use_negative:
- The GAN was trained without noise augmentation (add_noise disabled), since without
  the z~N(0,1) input there is no stochastic variation to measure mapping variance
- The data has not been preprocessed to ensure columns are mostly in the interval
  [-10, 10] as the README requires for feeding into the neural network
tools:
- name: scMMGAN
  role: Provides the noise-augmented generator whose stochastic mapping variance per
    gene is quantified
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Graph construction and sess.run execution of the generator over repeated z
    draws
- name: MAGIC
  role: Manifold smoothing used to correct dropout before training and variance quantification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:8bee338b55dc5903440b848d158488d223b4c0bbf04a0beac506bbcf495b7871
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gene-mapping-variance-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies per-gene uncertainty of scMMGAN cross-modal mapping on DBIT-seq data by repeatedly drawing the isotropic normal noise input z and computing the variance of generated gene values across stochastic mappings, comparing genes with vs. without an analogous proteomic measurement.

## When to use

Use when a trained noise-augmented GAN mapping between transcriptomic and proteomic domains is available and you need to quantify which genes' generated values are stable vs. unstable under stochastic z draws.

## When NOT to use

- The GAN was trained without noise augmentation (add_noise disabled), since without the z~N(0,1) input there is no stochastic variation to measure mapping variance
- The data has not been preprocessed to ensure columns are mostly in the interval [-10, 10] as the README requires for feeding into the neural network

## Inputs

- DBIT-seq dataset with transcriptomic (scRNA-seq) measurements and paired proteomic measurements per gene
- Trained scMMGAN model with add_noise=True in TensorFlow (generator output G(xi;z))
- Per-gene annotation of whether an analogous proteomic measurement exists

## Outputs

- Per-gene mapping variance values across stochastic z draws (mapping_variance_results.csv)
- Average mapping variance for genes with vs. without analogous proteomic measurements (reported: 0.026 vs 1.419)
- Ranked list of lowest-variance genes (Gm5049, Gm37500, Gm33051) and highest-variance genes (GM37686, Rp1)

## How to apply

Preprocess the DBIT-seq data with MAGIC dropout correction, zero-centering and unit scaling of each dimension, and reduction to 50 principal components. Build the noise-augmented scMMGAN in TensorFlow, computing the generator output as G(xi;z) with z~N(0,1) concatenated to the original input, using generator layers of (128, 256, 512) neurons and discriminator layers of (1,024, 512, 256) neurons, and train with the geometry-preserving correspondence loss on the first 10-20 diffusion eigenvectors using an inverse-distance kernel with kNN adaptive bandwidth, applying the eigenvector sign-correlation check with threshold -0.5. For each cell, repeatedly draw z and map to the other domain, then compute the per-gene variance of the generated values across draws. Split genes into those with an analogous proteomic measurement and those without, and compare average mapping variance between groups; rank genes by variance to identify the extremes. Correct application is validated when the reproduced group averages match the reported 0.026 (with analog) vs 1.419 (without analog), and the ranked extremes match the named genes.

## Related tools

- **scMMGAN** (Provides the noise-augmented generator whose stochastic mapping variance per gene is quantified) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Graph construction and sess.run execution of the generator over repeated z draws)
- **MAGIC** (Manifold smoothing used to correct dropout before training and variance quantification)

## Examples

```
for i in range(n_draws): z = np.random.normal(0, 1, D); gen = sess.run(output_fake2, feed_dict={x1: X, z_ph: z}); variances[i] = gen.var(axis=0)
```

## Evaluation signals

- Average mapping variance for genes with an analogous proteomic measurement reproduces 0.026
- Average mapping variance for genes without an analogous proteomic measurement reproduces 1.419
- The three lowest-variance genes are Gm5049, Gm37500, and Gm33051
- The highest-variance genes are GM37686 and Rp1
- All results are saved to mapping_variance_results.csv

## Limitations

- GAN training instability: adversarial losses can cause the model to deteriorate from effective to ineffective quickly across training iterations; early stopping and the geometric loss only informally mitigate this
- Mitigation of instability and mode collapse is only informally observed and these effects may still be present in some contexts
- The geometry-based loss is not completely plug-and-play; a choice of distance between data points is still required

## Evidence
