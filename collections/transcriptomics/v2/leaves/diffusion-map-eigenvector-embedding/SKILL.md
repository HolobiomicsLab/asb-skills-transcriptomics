---
name: diffusion-map-eigenvector-embedding
description: Use when you need a manifold-aware low-dimensional embedding of single-cell
  (or other high-dimensional) data, or diffusion coordinates to quantify how well
  a mapping preserves intrinsic geometry.
when_to_use_negative:
- You only need distributional alignment between modalities without point-wise geometric
  constraints — a standard adversarial/cycle-consistency loss may suffice
- The data require a different distance metric — the geometry-based loss is not completely
  'plug-and-play' since it still requires a choice of distance between data points
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3517
tools:
- name: scMMGAN
  role: Framework whose diffusion geometry correspondence loss consumes the diffusion
    eigenvector embedding; code at github.com/KrishnaswamyLab/scMMGAN
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Diffusion operator construction and eigendecomposition are implemented inside
    the TensorFlow graph so gradients flow through the kernel
- name: MAGIC
  role: Manifold smoothing/dropout correction applied upstream to produce the input
    matrix for the diffusion map
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:0da1c4ddf8f121641a9d2a68c22c56423448c0ccdcfcde2f8f783efa4d8a4feb
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# diffusion-map-eigenvector-embedding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes a diffusion map embedding φ by eigendecomposing a Markov transition matrix built from pairwise affinities, producing low-dimensional diffusion coordinates whose Euclidean distances reflect diffusion distance on the data manifold. In scMMGAN these eigenvectors feed the diffusion geometry correspondence loss that preserves intrinsic data geometry.

## When to use

Use when you need a manifold-aware low-dimensional embedding of single-cell (or other high-dimensional) data, or diffusion coordinates to quantify how well a mapping preserves intrinsic geometry.

## When NOT to use

- You only need distributional alignment between modalities without point-wise geometric constraints — a standard adversarial/cycle-consistency loss may suffice
- The data require a different distance metric — the geometry-based loss is not completely 'plug-and-play' since it still requires a choice of distance between data points

## Inputs

- Preprocessed data matrix (dropout-corrected with MAGIC, zero-centered/unit-scaled, reduced to 50 principal components)
- Pairwise distance matrix or raw minibatch (e.g. x1 or G(x1), batch size 256) for computing affinities
- Kernel bandwidth parameter σi (k-nearest-neighbor adaptive)
- Number of eigenvectors ne (10–20 in practice)

## Outputs

- n×ne diffusion eigenvector embedding (e.g. saved as diffusion_eigenvectors.npy), values rescaled to [−1, 1]
- Markov chain transition probability matrix P over the dataset
- Diffusion coordinates used to compute the correspondence loss Lc as the L2 distance between eigenvector representations of a point and its mapped counterpart

## How to apply

Start from the preprocessed data matrix (MAGIC dropout-corrected, zero-centered/unit-scaled, reduced to 50 principal components). Compute pairwise distances and convert them to affinities with the gradient-passing inverse-distance kernel k(xi,xj)=max(0,1−||xi−xj||1/σi), where σi is a k-nearest-neighbor adaptive bandwidth; this kernel is chosen over the Gaussian kernel because it does not saturate gradients. Row-normalize the affinity matrix A into a Markov chain transition matrix P (P=(∑jAij)−1A), whose powers Pt represent t diffusion steps. Compute the eigendecomposition of P and keep the first ne eigenvectors (10–20 works best in practice), rescale values to [−1, 1], and apply the sign-correlation check: if an eigenvector pair's correlation is below εr=−0.5, flip the sign of φG(x). Build the embedding inside the TensorFlow graph so gradients flow through the kernel when it is used in a trainable loss.

## Related tools

- **scMMGAN** (Framework whose diffusion geometry correspondence loss consumes the diffusion eigenvector embedding; code at github.com/KrishnaswamyLab/scMMGAN) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Diffusion operator construction and eigendecomposition are implemented inside the TensorFlow graph so gradients flow through the kernel)
- **MAGIC** (Manifold smoothing/dropout correction applied upstream to produce the input matrix for the diffusion map)

## Examples

```
A = np.maximum(0, 1 - D / sigma[:, None]); P = A / A.sum(axis=1, keepdims=True); phi = np.linalg.eig(P)[1][:, 1:ne]
```

## Evaluation signals

- Output embedding file exists with shape (n_samples × ne) where ne is between 10 and 20
- All embedding values fall within [−1, 1] after rescaling
- Sign-correlation check applied: eigenvector pairs with correlation below εr=−0.5 are sign-flipped before output/loss computation
- When used in the correspondence loss, reported loss values are finite, non-negative, and consistent with λc=10 weighting of the L2 eigenvector-distance term
- Loss values logged every 100 training steps via training_counter are stable and finite

## Limitations

- The geometry-based loss is not completely 'plug-and-play' — a choice of distance between data points is still required
- Mitigation of GAN training instability and mode collapse via the geometric loss is only informally observed; these effects may still be present in some contexts
- The Gaussian kernel saturates gradients, hence the inverse-distance kernel is used instead; using a saturating kernel would break gradient flow through the loss

## Evidence
