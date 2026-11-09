---
name: geometry-preserving-correspondence-loss-implementation
description: Use when training a cross-modality GAN (e.g. scMMGAN) where the mapping
  must keep nearby points nearby across domains — e.g. aligning scRNA-seq with spatial
  transcriptomics while preserving manifold structure.
when_to_use_negative:
- You only need distributional (adversarial) alignment without point-wise geometric
  fidelity — plain CycleGAN losses suffice and the eigenvector computation overhead
  is unnecessary
- Your data representation lacks a meaningful point-to-point distance metric — the
  geometry-based loss is not completely plug-and-play and requires a choice of distance
  between data points
- You are aligning more than a few modalities — the number of generators grows quadratically
  with the number of input modalities, forcing small or separately trained networks
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_2229
tools:
- name: scMMGAN
  role: Framework whose generator loss is augmented with the diffusion-geometry correspondence
    loss Lc
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Graph construction and session in which the correspondence loss and train_op_G
    are built and executed
- name: MAGIC
  role: Manifold-smoothing dropout correction applied to input modality matrices before
    diffusion-coordinate computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:34e2accb4193a01df95574a2a6858f75db27d2959c1f50fdad0131cb35d4ba4d
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# geometry-preserving-correspondence-loss-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements scMMGAN's diffusion-geometry correspondence loss Lc, which penalizes the L2 distance between diffusion-operator eigenvectors of original points and their GAN-mapped counterparts so the learned cross-modality mapping preserves intrinsic data geometry beyond distributional alignment.

## When to use

Use when training a cross-modality GAN (e.g. scMMGAN) where the mapping must keep nearby points nearby across domains — e.g. aligning scRNA-seq with spatial transcriptomics while preserving manifold structure.

## When NOT to use

- You only need distributional (adversarial) alignment without point-wise geometric fidelity — plain CycleGAN losses suffice and the eigenvector computation overhead is unnecessary
- Your data representation lacks a meaningful point-to-point distance metric — the geometry-based loss is not completely plug-and-play and requires a choice of distance between data points
- You are aligning more than a few modalities — the number of generators grows quadratically with the number of input modalities, forcing small or separately trained networks

## Inputs

- Two preprocessed single-cell modality matrices (e.g. scRNA-seq and spatial transcriptomics), each MAGIC-corrected, zero-centered/unit-scaled, reduced to 50 principal components
- Minibatches x1 and G(x1) of size 256 from the two domains
- Pairwise affinity matrices built with the inverse-distance kernel and kNN-adaptive bandwidth σi

## Outputs

- Row-normalized Markov transition probability matrices Px1 and PG(x1)
- First 10–20 rescaled (−1 to 1) diffusion eigenvectors φx1 and φG(x)
- Scalar correspondence loss Lc folded into LG1=λr·Lr+λd·Ld+λc·Lc
- Generator checkpoint optimized via train_op_G

## How to apply

Load the two preprocessed modality matrices (dropout-corrected with MAGIC, zero-centered/unit-scaled, reduced to 50 principal components) as x1 and x2 minibatches of size 256. For x1 and G(x1), build pairwise-distance matrices, transform them into affinities with the inverse-distance kernel k(xi,xj)=max(0,1−||xi−xj||1/σi) using a k-nearest-neighbor adaptive bandwidth σi, then row-normalize into Markov transition probability matrices Px1 and PG(x1). Compute the first ne (10–20) eigenvectors as diffusion coordinates, rescale them to [−1, 1], and apply the sign-correlation check: if an eigenvector pair's correlation is below εr=−0.5, multiply φG(x) by −1 before computing the loss. Set Lc as the L2 distance between the diffusion coordinates of x1 and G(x1) and add it to the generator loss LG1=λr·Lr+λd·Ld+λc·Lc with λr=λd=1 and λc=10 inside the TensorFlow graph; attach it to train_op_G (executed before train_op_D) and log losses every 100 steps via training_counter.

## Related tools

- **scMMGAN** (Framework whose generator loss is augmented with the diffusion-geometry correspondence loss Lc) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Graph construction and session in which the correspondence loss and train_op_G are built and executed)
- **MAGIC** (Manifold-smoothing dropout correction applied to input modality matrices before diffusion-coordinate computation)

## Examples

```
LG1 = lambda_r*Lr + lambda_d*Ld + lambda_c*Lc  # with lambda_r=lambda_d=1, lambda_c=10, inside the TensorFlow graph before sess.run([train_op_G, train_op_D])
```

## Evaluation signals

- correspondence_loss.py executes within the TensorFlow session on real minibatches (script_runs) without error
- Reported Lc values are finite, non-negative, and consistent with λc=10 weighting of the L2 eigenvector-distance term
- Loss values logged every 100 training steps via training_counter show the geometry term decreasing or stabilizing
- Generated output is stable across training iterations (no further training needed per README guidance)
- Nearby points in the input space remain nearby in the aligned space (the failure mode λ_correspondence is meant to fix)

## Limitations

- The geometry-based loss is not completely plug-and-play — it still requires a choice of distance between data points
- The correspondence loss is only enforced on the first 10–20 eigenvectors; coarser-scale adjacent-eigenvector comparisons supplement point-wise matching
- Adversarial training can be unstable and suffer mode collapse; early stopping and the geometric loss only informally mitigate these effects

## Evidence

- [methods] eigenvector count: [evidence span withheld — anchor status: not_found]
