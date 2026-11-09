---
name: adaptive-bandwidth-inverse-distance-kernel-construction
description: Use when building affinity/transition matrices for diffusion maps or
  geometry-preserving losses in a TensorFlow GAN pipeline, and gradients must flow
  through the kernel without saturating.
when_to_use_negative:
- Downstream analysis only needs a distance metric and no gradients through the kernel
  — a Gaussian kernel is simpler and sufficient
- Input data is not embedded in a continuous ambient space (e.g. already discrete
  alignment output), where pairwise inverse-distance affinities are not meaningful
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3308
tools:
- name: scMMGAN
  role: Framework whose diffusion geometry loss consumes the kernel-built affinity/transition
    matrices
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Graph construction so the kernel and row normalization are differentiable
    end-to-end
  repo: github.com/KrishnaswamyLab/scMMGAN
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:9ddb3ad96beedf8c1d918379ada44cedc4f371d3b5ce3a2ef87c9bc2b1506aca
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/KrishnaswamyLab/scMMGAN
  tools:
  - TensorFlow
  - scMMGAN
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Adaptive-bandwidth inverse-distance kernel construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the pairwise affinity matrix used by scMMGAN's diffusion geometry loss with a gradient-passing inverse-distance kernel k(xi,xj)=max(0,1−||xi−xj||1/σi), where σi is a k-nearest-neighbor adaptive bandwidth, replacing the Gaussian kernel to avoid gradient saturation.

## When to use

Use when building affinity/transition matrices for diffusion maps or geometry-preserving losses in a TensorFlow GAN pipeline, and gradients must flow through the kernel without saturating.

## When NOT to use

- Downstream analysis only needs a distance metric and no gradients through the kernel — a Gaussian kernel is simpler and sufficient
- Input data is not embedded in a continuous ambient space (e.g. already discrete alignment output), where pairwise inverse-distance affinities are not meaningful

## Inputs

- Preprocessed data matrix (MAGIC dropout-corrected, zero-centered/unit-scaled per column, reduced to 50 principal components)
- k-nearest-neighbor distances used to set the adaptive bandwidth σi per data point

## Outputs

- Pairwise affinity matrix A over the dataset
- Row-normalized Markov chain transition probability matrix P=(∑jAij)−1A feeding diffusion map eigendecomposition

## How to apply

Start from a preprocessed data matrix (dropout-corrected with MAGIC, zero-centered/unit-scaled, reduced to 50 principal components). Compute the pairwise distance matrix and transform it into affinities using the inverse-distance kernel k(xi,xj)=max(0,1−||xi−xj||1/σi), where the bandwidth σi is set per point from its k-nearest-neighbor distance, rather than the Gaussian kernel whose gradient saturates. Row-normalize the affinity matrix A into the Markov chain transition probability matrix via P=(∑j Aij)^−1 A, implemented inside the TensorFlow graph so gradients flow through the kernel. Keep the kernel differentiable everywhere the loss touches it, since the geometry loss is computed on the first 10–20 diffusion eigenvectors.

## Related tools

- **scMMGAN** (Framework whose diffusion geometry loss consumes the kernel-built affinity/transition matrices) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Graph construction so the kernel and row normalization are differentiable end-to-end) — github.com/KrishnaswamyLab/scMMGAN

## Examples

```
A = np.maximum(0, 1 - D / sigma[:, None]); row_sums = A.sum(axis=1, keepdims=True); P = A / row_sums
```

## Evaluation signals

- All kernel values fall in [0, 1] after clipping at 0 via the max(0, ·) rule
- Row-normalized transition matrix P has rows summing to 1 (valid Markov chain transition probabilities)
- Gradients flow through the kernel in the TensorFlow graph (no saturation, unlike the Gaussian kernel)
- Downstream diffusion eigenvectors (first 10–20) used in the correspondence loss are finite and the sign-correlation check (εr=−0.5) resolves sign flips

## Limitations

- The geometry-based loss is not completely plug-and-play: a choice of distance between data points is still required
- Replacing the Gaussian kernel is motivated by gradient saturation, but the adaptive bandwidth choice itself remains a hyperparameter decision

## Evidence

- [methods] A Markov chain transition matrix over the dataset P is constructed from the pairwise affinity matrix A via P=(∑jAij)−1A: "A Markov chain transition matrix over the dataset P is constructed from the pairwise affinity matrix A via P=(∑jAij)−1A"
- [abstract] The framework's key improvement is an additional diffusion geometry loss with a new kernel that constrains the otherwise over-parameterized GAN: "The framework's key improvement is an additional diffusion geometry loss with a new kernel that constrains the otherwise over-parameterized GAN"
- [readme] Running lines through the sess.run call in line 421 prepare the code but do not start any training yet.: "Running lines through the `sess.run` call in line 421 prepare the code but do not start any training yet."
- [discussion] our geometry-based loss is not completely plug-and-play in the sense that we still require a choice of distance between data points: "our geometry-based loss is not completely “plug-and-play” in the sense that we still require a choice of distance between data points."
