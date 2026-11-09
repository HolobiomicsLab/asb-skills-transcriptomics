---
name: spatial-transcriptomics-scrnaseq-mapping
description: Use when separately measured single-cell domains (spatial RNA-seq and
  scRNA-seq, e.g. Chromium 10X from tumor xenografts) must be aligned so that mapped
  points retain per-sample correspondence to their inputs, not just distribution-level
  similarity.
when_to_use_negative:
- When only distribution-level matching is needed and point-wise correspondence does
  not matter — a plain CycleGAN or CMAE alignment suffices and the correspondence
  loss is unnecessary overhead.
- 'When more than a few modalities must be aligned simultaneously: the number of generators
  grows quadratically with the number of input modalities, forcing small or separately
  trained networks.'
- When the data have no meaningful diffusion geometry or no defensible choice of distance
  between data points, since the geometry-based loss is not completely plug-and-play.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3678
tools:
- name: scMMGAN
  role: Adversarial model with the diffusion-geometry correspondence loss that performs
    the spatial-to-single-cell mapping
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Graph construction and training loop (train_op_G before train_op_D, sanity-check
    losses every 100 steps)
- name: MAGIC
  role: Manifold smoothing used to correct dropout during preprocessing of both domains
- name: CycleGAN
  role: Baseline alignment method compared against scMMGAN in the MSE/MMD evaluation
- name: CMAE
  role: Baseline alignment method compared against scMMGAN in the MSE/MMD evaluation
- name: Chromium 10X
  role: Technology used to generate the scRNA-seq input data from sorted GFP+ tumor
    cells
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:3a28e4681458b9d711e9a02dfbdb29acc8e15b678b5df2ac88e47e8110defd59
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spatial-transcriptomics-scrnaseq-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces scMMGAN-based mapping between spatial transcriptomics and scRNA-seq domains (e.g., MDA-MB-231 triple-negative breast cancer xenograft samples), where a diffusion-geometry correspondence loss preserves point-wise correspondence while matching the target distribution, outperforming CycleGAN and CMAE baselines on per-sample MSE.

## When to use

Use when separately measured single-cell domains (spatial RNA-seq and scRNA-seq, e.g. Chromium 10X from tumor xenografts) must be aligned so that mapped points retain per-sample correspondence to their inputs, not just distribution-level similarity.

## When NOT to use

- When only distribution-level matching is needed and point-wise correspondence does not matter — a plain CycleGAN or CMAE alignment suffices and the correspondence loss is unnecessary overhead.
- When more than a few modalities must be aligned simultaneously: the number of generators grows quadratically with the number of input modalities, forcing small or separately trained networks.
- When the data have no meaningful diffusion geometry or no defensible choice of distance between data points, since the geometry-based loss is not completely plug-and-play.

## Inputs

- scRNA-seq count matrices from Chromium 10X for MDA-MB-231-GFP xenograft samples (primary tumor, lung, liver, lymph node), FACS-sorted GFP+ alive tumor cells
- Spatial transcriptomics matrices from matched tumor sections
- Pairwise affinity matrices for constructing diffusion maps (row-normalized into transition probability matrices Px1 and PG(x1))

## Outputs

- output_fake2: generated data mapping x1 (scRNA-seq) into the spatial transcriptomics domain, with preprocessing scaling inverted
- output_fake: generated data mapping x2 (spatial transcriptomics) into the scRNA-seq domain
- table3_alignment_metrics.csv: per-model, per-sample MMD and MSE values for scMMGAN, CycleGAN, and CMAE

## How to apply

Preprocess both the scRNA-seq and spatial transcriptomics datasets by correcting dropout with MAGIC manifold smoothing, zero-centering and unit-scaling each dimension, and reducing to 50 principal components so columns mostly lie in [-10, 10] per the README. Build the scMMGAN in TensorFlow with a generator of three internal layers (128, 256, 512 neurons, batch norm, leaky ReLU) and a discriminator (1024, 512, 256 neurons), learning rate 0.0001, batch size 256; concatenate a N(0,1) noise draw with the input to the generator. Add the geometry-preserving correspondence loss with the inverse-distance kernel k(xi,xj)=max(0,1-||xi-xj||1/sigma_i), applied to the first 10-20 diffusion-map eigenvectors after a sign-correlation check (flip phiG(x) when correlation is below epsilon_r=-0.5), with lambda_correspondence=10 and lambda_cycle=1. In the training loop, alternate optimizing train_op_G (generator first) then train_op_D (discriminator), reporting sanity-check losses every 100 steps, and early-stop once generated output is stable. After training, collect output_fake and output_fake2 as the cross-domain mappings, invert any preprocessing scaling, then compute per-sample MMD (distribution match) and MSE (point-wise preservation) per model across the four scRNA-seq samples (primary tumor, lung, liver, lymph node).

## Related tools

- **scMMGAN** (Adversarial model with the diffusion-geometry correspondence loss that performs the spatial-to-single-cell mapping) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Graph construction and training loop (train_op_G before train_op_D, sanity-check losses every 100 steps))
- **MAGIC** (Manifold smoothing used to correct dropout during preprocessing of both domains)
- **CycleGAN** (Baseline alignment method compared against scMMGAN in the MSE/MMD evaluation)
- **CMAE** (Baseline alignment method compared against scMMGAN in the MSE/MMD evaluation)
- **Chromium 10X** (Technology used to generate the scRNA-seq input data from sorted GFP+ tumor cells)

## Examples

```
python scMMGAN.py  # with x1, x2 preprocessed via MAGIC + z-score + 50 PCs; train alternating sess.run(train_op_G) then sess.run(train_op_D); read output_fake/output_fake2
```

## Evaluation signals

- scMMGAN per-sample MSE falls in the reported range of approximately 0.87-1.03 across the four samples, while CycleGAN and CMAE fall in the approximately 1.6-2.1 range.
- All three models achieve reasonably low MMD, confirming the target distribution is matched; scMMGAN is distinguished by low MSE, not MMD.
- Nearby points in the input space remain nearby in the aligned space (data geometry retained), consistent with lambda_correspondence=10 being sufficient.
- Generated output is stable across training iterations when training is stopped (README early-stopping criterion).
- Preprocessed columns lie mostly in the interval [-10, 10] before training, per the README data-loading guidance.

## Limitations

- GANs are difficult to train due to adversarial losses, which can lead to instability, and the model can deteriorate from effective to ineffective quickly across training iterations.
- GANs often suffer from mode collapse because they are not penalized by distribution-level losses; early stopping and the geometric loss mitigate this only informally.
- The geometry-based loss is not completely plug-and-play: a choice of distance between data points is still required.
- Aligned results require continued evaluation through external verification measures; novel gene correlations found by scMMGAN remain potential discoveries needing specifically designed experiments.

## Evidence

- [methods] We only enforce the correspondence loss on the first eigenvectors ... In practice we find using 10–20 eigenvectors works best: [evidence span withheld — anchor status: not_found]
- [discussion] our geometry-based loss is not completely "plug-and-play" in the sense that we still require a choice of distance between data points.: [evidence span withheld — anchor status: not_found]
