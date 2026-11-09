---
name: gan-based-cross-modal-data-alignment
description: Use when two discrete single-cell datasets measuring different modalities
  (scRNA-seq, spatial transcriptomics, proteomics) of related samples must be mapped
  into each other's feature space while retaining per-sample correspondence, not just
  distribution-level similarity.
when_to_use_negative:
- When only distribution-level similarity between modalities matters and point-wise
  per-sample correspondence is irrelevant — CycleGAN or CMAE style distribution-only
  losses suffice
- When the input data are already jointly measured (simultaneous multi-omic measurements)
  rather than two discrete, separately measured datasets, since the GAN alignment
  of discrete domains is unnecessary
- When a very large number of modalities must be aligned at once, since the number
  of required generators grows quadratically, forcing small or separately trained
  networks
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMMGAN
  role: 'Core model: multi-modal GAN with adversarial, cycle, and diffusion-geometry
    correspondence losses mapping between the two domains'
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Graph construction and training loop; train_op_G and train_op_D sessions alternate
    generator/discriminator optimization
- name: MAGIC
  role: Manifold-smoothing dropout correction applied to input data before scaling
    and PCA
- name: CycleGAN
  role: Distribution-only baseline used to show correspondence is lost without the
    geometry-preserving loss
- name: CMAE
  role: Autoencoder-style distribution-only baseline compared against scMMGAN in MMD/MSE
    and correlation benchmarks
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:275a4b2b21a9d571cef7904169f4683624b830d2fe98dce2b000414b28e3da31
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/KrishnaswamyLab/scMMGAN
  tools:
  - CMAE
  - Chromium 10X
  - CycleGAN
  - MAGIC
  - TensorFlow
  - scMMGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gan-based-cross-modal-data-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for aligning two separately measured single-cell multi-omics modalities (e.g. spatial transcriptomics and scRNA-seq) with scMMGAN, a GAN whose adversarial and cycle losses are constrained by a diffusion-geometry correspondence loss so that point-wise correspondence is preserved alongside distribution matching.

## When to use

Use when two discrete single-cell datasets measuring different modalities (scRNA-seq, spatial transcriptomics, proteomics) of related samples must be mapped into each other's feature space while retaining per-sample correspondence, not just distribution-level similarity.

## When NOT to use

- When only distribution-level similarity between modalities matters and point-wise per-sample correspondence is irrelevant — CycleGAN or CMAE style distribution-only losses suffice
- When the input data are already jointly measured (simultaneous multi-omic measurements) rather than two discrete, separately measured datasets, since the GAN alignment of discrete domains is unnecessary
- When a very large number of modalities must be aligned at once, since the number of required generators grows quadratically, forcing small or separately trained networks

## Inputs

- Two preprocessed single-cell data matrices x1 and x2 from different modalities (e.g. MDA-MB-231 scRNA-seq and spatial transcriptomics reduced to 50 PCs; DBIT-seq spatial transcriptomic and proteomic matrices)
- Pairwise affinity/transition probability matrices Px1 and PG(x1) for the diffusion-geometry loss
- Hyperparameter configuration (batch_size=256, learning_rate=0.0001, lambda_correspondence=10, lambda_cycle=1, TRAINING_STEPS, nfilt, add_noise, use_bn)

## Outputs

- output_fake2: generated data mapping x1 into the second domain (scRNA-seq -> spatial transcriptomics)
- output_fake: generated data mapping x2 into the first domain (spatial transcriptomics -> scRNA-seq), with preprocessing scaling inverted
- Cross-modal alignment benchmark tables (e.g. table1_metrics.csv with MMD and correlation per model; table3_alignment_metrics.csv with per-sample MMD and MSE for primary tumor, lung, liver, lymph node samples)

## How to apply

Preprocess both input matrices (x1, x2) with MAGIC dropout correction, zero-center and unit-scale each dimension so columns fall mostly in [-10, 10], then reduce to 50 principal components. Build the scMMGAN TensorFlow graph with generators of three internal layers (128, 256, 512 neurons, batch norm + leaky ReLU) and discriminators (1024, 512, 256 neurons), learning rate 0.0001, batch size 256, correspondence loss coefficient 10, cycle-loss coefficient 1. The correspondence loss computes diffusion maps from pairwise affinities (row-normalized into transition matrices Px1 and PG(x1)), applies an inverse-distance kernel k(xi,xj)=max(0,1-||xi-xj||1/sigma_i), sign-corrects eigenvectors whose correlation is below epsilon_r=-0.5 by multiplying phi_G(x) by -1, and is enforced only on the first 10-20 eigenvectors. Each training step alternates sess.run of train_op_G (generator first) then train_op_D (discriminator second), with loss sanity checks every 100 steps. Extract output_fake2 (x1 mapped to domain 2) and output_fake (x2 mapped to domain 1), inverting any preprocessing scaling; benchmark against distribution-only baselines (CycleGAN, CMAE) using MMD plus a correspondence metric (per-sample MSE or correlation of corresponding columns).

## Related tools

- **scMMGAN** (Core model: multi-modal GAN with adversarial, cycle, and diffusion-geometry correspondence losses mapping between the two domains) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Graph construction and training loop; train_op_G and train_op_D sessions alternate generator/discriminator optimization)
- **MAGIC** (Manifold-smoothing dropout correction applied to input data before scaling and PCA)
- **CycleGAN** (Distribution-only baseline used to show correspondence is lost without the geometry-preserving loss)
- **CMAE** (Autoencoder-style distribution-only baseline compared against scMMGAN in MMD/MSE and correlation benchmarks)

## Examples

```
lambda_correspondence=10; lambda_cycle=1; batch_size=256; learning_rate=0.0001  # per TensorFlow session: sess.run(train_op_G); sess.run(train_op_D)
```

## Evaluation signals

- Per-sample MSE of the alignment should fall in the reported ~0.87-1.03 range across the four TNBC scRNA-seq samples (primary tumor, lung, liver, lymph node), versus ~1.6-2.1 for CycleGAN and CMAE
- On DBIT-seq data, average correlation between columns known to correspond should reach approximately r=0.154/0.152 in the two directions for scMMGAN while CycleGAN and CMAE give near-zero correlations
- MMD between G(x1) and x2 and between x1 and G(x2) should be similarly low for all three models — scMMGAN's advantage shows in MSE/correlation, not MMD
- Loss sanity checks every 100 training steps (training_counter) should show stable, non-diverging adversarial losses; output should become stable across training iterations before stopping
- Correlations between input eigenvectors and mapped eigenvectors should exceed the epsilon_r=-0.5 sign-correction threshold after flipping, confirming diffusion geometry is preserved

## Limitations

- GANs are difficult to train because of adversarial losses, which can lead to instability; the model can deteriorate from effective to ineffective quickly across training iterations
- GANs often suffer from mode collapse because they are not penalized by distribution-level losses to match the entire distribution; early stopping and the geometric loss only informally mitigate this
- The geometry-based loss is not completely 'plug-and-play': a choice of distance between data points is still required
- The number of generators necessary grows quadratically with the number of input modalities, forcing small or separately trained networks for many modalities
- Aligned results require continued evaluation through external verification measures; novel gene correlations found by scMMGAN need further investigation with specifically designed experiments

## Evidence
