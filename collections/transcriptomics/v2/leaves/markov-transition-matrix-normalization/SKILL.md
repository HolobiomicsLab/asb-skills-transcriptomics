---
name: markov-transition-matrix-normalization
description: Use when a pairwise affinity/similarity matrix over data points must
  be converted into a diffusion (random-walk) operator, e.g. before eigendecomposition
  for diffusion map coordinates in a geometry-preserving loss.
when_to_use_negative:
- Input is a raw distance/similarity matrix not yet converted to kernel affinities
  — the kernel construction (adaptive bandwidth, gradient-passing kernel) must precede
  normalization
- The analysis requires a symmetric graph Laplacian (e.g. spectral clustering with
  normalized cuts) rather than an asymmetric row-stochastic random-walk operator
- Data points are not samples on a shared manifold (e.g. feature-by-feature tables
  without meaningful pairwise geometry), since diffusion probabilities would be meaningless
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: scMMGAN
  role: Framework in which the affinity matrix is row-normalized into transition probability
    matrices Px1 and PG(x1) inside the TensorFlow graph for the geometry-preserving
    correspondence loss
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Graph construction so the row normalization and downstream diffusion geometry
    loss keep gradients flowing through the kernel
  repo: github.com/KrishnaswamyLab/scMMGAN
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:5d5bf25951c2fbad08aacf9ae1cf072739a7be785f78d0fe117f1afb6d60ec5d
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# markov-transition-matrix-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Row-normalizes a pairwise affinity matrix (e.g. the gradient-passing inverse-distance kernel used in scMMGAN) into a Markov chain transition probability matrix P, whose powers Pt model forward diffusion steps on the data manifold for diffusion map embeddings.

## When to use

Use when a pairwise affinity/similarity matrix over data points must be converted into a diffusion (random-walk) operator, e.g. before eigendecomposition for diffusion map coordinates in a geometry-preserving loss.

## When NOT to use

- Input is a raw distance/similarity matrix not yet converted to kernel affinities — the kernel construction (adaptive bandwidth, gradient-passing kernel) must precede normalization
- The analysis requires a symmetric graph Laplacian (e.g. spectral clustering with normalized cuts) rather than an asymmetric row-stochastic random-walk operator
- Data points are not samples on a shared manifold (e.g. feature-by-feature tables without meaningful pairwise geometry), since diffusion probabilities would be meaningless

## Inputs

- Pairwise affinity matrix A over n data points, built with the inverse-distance kernel k(xi,xj)=max(0,1−||xi−xj||1/σi) with kNN-adaptive bandwidth σi
- Preprocessed data matrix (dropout-corrected via MAGIC, zero-centered/unit-scaled, reduced to 50 principal components)

## Outputs

- Row-stochastic Markov chain transition probability matrix P (n×n), e.g. Px1 and PG(x1) in scMMGAN
- Powers Pt of P for t forward diffusion steps, feeding eigendecomposition into diffusion map eigenvectors φ

## How to apply

Start from the pairwise affinity matrix A built with the gradient-passing inverse-distance kernel k(xi,xj)=max(0,1−||xi−xj||1/σi), where σi is a k-nearest-neighbor adaptive bandwidth chosen to avoid Gaussian-kernel gradient saturation. Row-normalize A via P=(Σj Aij)^-1 A so each row sums to 1, turning P into a Markov chain transition probability matrix over the dataset. In the scMMGAN TensorFlow graph this yields transition matrices Px1 and PG(x1) (the latter over generated points), both computed inside the graph so gradients flow through the kernel and normalization. P's matrix power Pt represents t forward diffusion steps; subsequent eigendecomposition of P (keeping the first ~10–20 eigenvectors, with sign-correlation flipping below εr=−0.5) produces the diffusion coordinates used in the correspondence loss.

## Related tools

- **scMMGAN** (Framework in which the affinity matrix is row-normalized into transition probability matrices Px1 and PG(x1) inside the TensorFlow graph for the geometry-preserving correspondence loss) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Graph construction so the row normalization and downstream diffusion geometry loss keep gradients flowing through the kernel) — github.com/KrishnaswamyLab/scMMGAN

## Examples

```
P = A / np.sum(A, axis=1, keepdims=True)  # row-normalize affinity matrix A into Markov transition matrix P
```

## Evaluation signals

- Each row of the resulting P sums to 1 (row-stochasticity invariant)
- All entries of P are non-negative, since affinities come from max(0,1−||xi−xj||1/σi)
- P has the same shape as the input affinity matrix (n_samples × n_samples)
- Downstream eigenvectors of P, rescaled to [−1, 1], yield an embedding of shape (n_samples × ne) with ne in the 10–20 range
- Gradients flow: normalization is inside the TensorFlow graph so train_op_G updates reflect the diffusion geometry loss

## Limitations

- The geometry-based loss is not completely 'plug-and-play': a choice of distance between data points (and hence kernel bandwidth σi) is still required
- Using the Gaussian kernel here would saturate gradients; the inverse-distance kernel is required for gradient flow through the normalization
- Only the first 10–20 eigenvectors are enforced in the correspondence loss; later eigenvectors carry no constraint

## Evidence
