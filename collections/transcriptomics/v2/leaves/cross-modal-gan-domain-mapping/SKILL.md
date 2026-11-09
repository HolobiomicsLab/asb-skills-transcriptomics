---
name: cross-modal-gan-domain-mapping
description: Use when two separately measured single-cell modalities of related cells
  must be aligned or a missing modality predicted — e.g., spatial transcriptomics
  to scRNA-seq or paired ATAC-seq/RNA-seq — while preserving data geometry.
when_to_use_negative:
- Aligning more than two modalities at once — the number of generators grows quadratically,
  forcing small or separately trained networks
- When no meaningful distance metric between data points can be chosen — the geometry-based
  loss is not completely 'plug-and-play'
- When only a feature table without underlying data geometry is available, since the
  method depends on diffusion-geometry correspondence
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: scMMGAN
  role: Core adversarial framework performing cross-modal domain mapping with diffusion-geometry
    correspondence loss
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Graph construction, training loop (train_op_G/train_op_D), and session runs
    producing output_fake/output_fake2
- name: MAGIC
  role: Manifold-smoothing dropout correction applied to both modalities before training
- name: CycleGAN
  role: Baseline without the geometry-preserving correspondence loss (cycle-loss coefficient
    1)
- name: CMAE
  role: Baseline with separate encoders/decoders crossing into a shared latent space
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:699f929bf91f5ee39312b9d5a582c5a8b8dfb4241ec94c8440fde7dc7628b96d
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
  - CycleGAN
  - MAGIC
  - TensorFlow
  - scMMGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-modal-gan-domain-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train scMMGAN, a multi-modal GAN with a geometry-preserving correspondence loss, to map single-cell measurements (e.g., spatial transcriptomics, scATAC-seq) into a target single-cell modality (e.g., scRNA-seq), generating point-wise aligned synthetic profiles for downstream analysis.

## When to use

Use when two separately measured single-cell modalities of related cells must be aligned or a missing modality predicted — e.g., spatial transcriptomics to scRNA-seq or paired ATAC-seq/RNA-seq — while preserving data geometry.

## When NOT to use

- Aligning more than two modalities at once — the number of generators grows quadratically, forcing small or separately trained networks
- When no meaningful distance metric between data points can be chosen — the geometry-based loss is not completely 'plug-and-play'
- When only a feature table without underlying data geometry is available, since the method depends on diffusion-geometry correspondence

## Inputs

- Paired or related single-cell modality matrices X1 and X2 (e.g., spatial transcriptomics spots x scRNA-seq genes; granulocyte scATAC-seq x scRNA-seq)
- Preprocessed feature matrices (MAGIC-corrected, zero-centered/unit-scaled, reduced to 50 PCs)
- Hyperparameter settings (TRAINING_STEPS, batch_size, nfilt, lambda_cycle, lambda_correspondence, add_noise, use_bn)

## Outputs

- Generated cross-domain profiles from output_fake2 (x1 mapped into second domain) and output_fake (x2 mapped into first domain)
- Inverse-transformed predictions in the original feature space
- Benchmark metrics: MMD between generated and true distributions and point-wise gene-level correlation (e.g., table2_metrics.csv)

## How to apply

Preprocess both domains (x1, x2) with MAGIC dropout correction, zero-centering and unit scaling so columns fall mostly in [-10, 10], then reduce to 50 principal components. Build the TensorFlow graph with a generator of 128/256/512-neuron layers and a 1024/512/256-neuron discriminator, learning rate 0.0001, batch size 256, lambda_correspondence=10, lambda_cycle=1; the correspondence loss uses inverse-distance kernels with kNN-adaptive bandwidth, row-normalized into Markov transition matrices, enforced on the first 10–20 diffusion eigenvectors with sign correction when correlation falls below εr=-0.5. Optimize the generator (train_op_G) then the discriminator (train_op_D) each iteration; collect generated data from output_fake (x2→domain 1) or output_fake2 (x1→domain 2) and invert preprocessing such as scaling. Compare against CycleGAN and CMAE baselines trained on the same architecture using MMD and point-wise correlation to the measured target modality.

## Related tools

- **scMMGAN** (Core adversarial framework performing cross-modal domain mapping with diffusion-geometry correspondence loss) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Graph construction, training loop (train_op_G/train_op_D), and session runs producing output_fake/output_fake2)
- **MAGIC** (Manifold-smoothing dropout correction applied to both modalities before training)
- **CycleGAN** (Baseline without the geometry-preserving correspondence loss (cycle-loss coefficient 1))
- **CMAE** (Baseline with separate encoders/decoders crossing into a shared latent space)

## Examples

```
gen = sess.run(output_fake2, feed_dict={x1: x1_preprocessed})  # map x1 (e.g., spatial transcriptomics) into second (scRNA-seq) domain
```

## Evaluation signals

- Point-wise correlation between predicted and measured target modality falls within the reported ~0.313–0.358 range (scMMGAN average r=0.336) while CycleGAN and CMAE predictions are essentially uncorrelated
- MMD between generated and true distributions shows scMMGAN preserves distributional properties
- Spectral k-means (k=5) clusters of generated profiles, plotted on original spatial coordinates, preserve the spatial patterns of the tissue (visual check vs Figure 5B)
- Sanity-check losses printed every 100 training steps via training_counter remain stable and generated output is stable across iterations before stopping
- Eigenvector sign-correlation check: values of φG(x) multiplied by −1 when correlation is below εr=-0.5 before computing the loss

## Limitations

- GANs are difficult to train due to adversarial losses; the model can deteriorate from effective to ineffective quickly across training iterations
- Mode collapse can occur; early stopping and the geometric loss only informally mitigate it, and effects may still be present in some contexts
- The geometry-based loss requires a choice of distance between data points and is not completely plug-and-play
- Aligned results and novel gene correlations require continued external verification with specifically designed experiments

## Evidence

- [methods] we correct for dropout with the manifold smoothing method MAGIC: "we correct for dropout with the manifold smoothing method MAGIC"
- [intro] geometry-preserving loss intro: "extend the application to multi-modal integration and specify a more powerful, generally applicable correspondence loss: the geometry-preserving loss."
- [methods] eigenvector restriction: [evidence span withheld — anchor status: not_found]
