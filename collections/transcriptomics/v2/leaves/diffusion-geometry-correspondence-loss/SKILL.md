---
name: diffusion-geometry-correspondence-loss
description: Use when training a GAN to map between single-cell modalities (e.g. paired
  scRNA-seq/scATAC-seq, DBIT-seq transcriptomics/proteomics, or spatial RNA-seq to
  scRNA-seq) and distribution-matching losses alone leave aligned points uncorrelated
  with ground truth, i.
when_to_use_negative:
- The geometry-based loss is not completely 'plug-and-play' — it requires choosing
  a distance between data points — so it is the wrong choice when no meaningful distance
  metric over the input domain exists or the data cannot be embedded in a metric space
- You only need distribution-level matching with no point-wise correspondence requirement;
  a plain CycleGAN or CMAE distribution loss suffices and avoids the extra kernel/eigendecomposition
  machinery
- Aligning more than two large modalities where the quadratic growth in required generators
  would force the networks to be made small or trained separately
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scMMGAN
  role: Framework whose generator objective carries the diffusion-geometry correspondence
    loss; eigenvectors are computed inside its TensorFlow graph so gradients flow
    through the kernel
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Graph construction and training loop (train_op_G then train_op_D) in which
    the transition matrices and eigenvector loss are implemented
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: MAGIC
  role: Manifold smoothing used to correct dropout in the input domains before affinities
    and diffusion eigenvectors are computed
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_003
  - task_001
  - task_008
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:258be6e1e3a91a49a8ac92a8d03443409583d1276fcb295797a49c0c03ddc71c
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# diffusion-geometry-correspondence-loss

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A geometry-preserving correspondence loss for scMMGAN (TensorFlow) that constrains an over-parameterized GAN by enforcing preservation of diffusion-map geometry across the learned cross-modality mapping, using a new gradient-passing inverse-distance kernel instead of the Gaussian kernel.

## When to use

Use when training a GAN to map between single-cell modalities (e.g. paired scRNA-seq/scATAC-seq, DBIT-seq transcriptomics/proteomics, or spatial RNA-seq to scRNA-seq) and distribution-matching losses alone leave aligned points uncorrelated with ground truth, i.e. point-wise correspondence must be preserved.

## When NOT to use

- The geometry-based loss is not completely 'plug-and-play' — it requires choosing a distance between data points — so it is the wrong choice when no meaningful distance metric over the input domain exists or the data cannot be embedded in a metric space
- You only need distribution-level matching with no point-wise correspondence requirement; a plain CycleGAN or CMAE distribution loss suffices and avoids the extra kernel/eigendecomposition machinery
- Aligning more than two large modalities where the quadratic growth in required generators would force the networks to be made small or trained separately

## Inputs

- Preprocessed single-cell data matrices x1 and x2 (e.g. DBIT-seq transcriptomic/proteomic, paired granulocyte scRNA-seq/scATAC-seq, MDA-MB-231 spatial transcriptomics/scRNA-seq) with dropout corrected by MAGIC, zero-centered/unit-scaled columns, reduced to 50 principal components
- Pairwise distance matrix over one data domain (L1 distance) with kNN-adaptive bandwidths sigma_i
- Trained generator mapping G(x) between the two domains, implemented in the TensorFlow graph

## Outputs

- Row-normalized Markov chain transition probability matrices Px1 and PG(x1)
- Diffusion map embedding phi (n_samples x ne eigenvectors, ne = 10-20), values rescaled to [-1, 1], e.g. saved to diffusion_eigenvectors.npy
- Correspondence loss term added to the generator objective with coefficient lambda_correspondence=10

## How to apply

Preprocess each domain (MAGIC dropout correction, zero-centering and unit scaling per dimension, reduction to 50 principal components) so columns lie mostly in [-10, 10]. Compute pairwise L1 distances, convert to affinities with the inverse-distance kernel k(xi,xj)=max(0,1-||xi-xj||1/sigma_i) using a k-nearest-neighbor adaptive bandwidth (chosen over the Gaussian kernel so gradients pass), then row-normalize affinities into Markov transition matrices Px1 and PG(x1). Extract the first 10-20 diffusion eigenvectors within the TensorFlow graph so gradients flow, rescale values to [-1, 1], and flip eigenvector signs when the correlation between eigenvector pairs falls below the threshold epsilon_r=-0.5 before computing the loss. Enforce the correspondence loss with coefficient lambda_correspondence=10 alongside a cycle loss of coefficient 1, training with learning rate 0.0001 and batch size 256, optimizing the generator first (train_op_G) then the discriminator (train_op_D).

## Related tools

- **scMMGAN** (Framework whose generator objective carries the diffusion-geometry correspondence loss; eigenvectors are computed inside its TensorFlow graph so gradients flow through the kernel) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Graph construction and training loop (train_op_G then train_op_D) in which the transition matrices and eigenvector loss are implemented) — github.com/KrishnaswamyLab/scMMGAN
- **MAGIC** (Manifold smoothing used to correct dropout in the input domains before affinities and diffusion eigenvectors are computed)

## Examples

```
D = cdist(X, X, 'minkowski', p=1); A = np.maximum(0, 1 - D / sigma_i); P = A / A.sum(axis=1, keepdims=True); eigvals, phi = scipy.sparse.linalg.eigs(P, k=20); np.save('diffusion_eigenvectors.npy', phi)
```

## Evaluation signals

- On DBIT-seq mouse embryo data, scMMGAN's average column-correlation between known corresponding entries after mapping is approximately 0.154/0.152, while CycleGAN and CMAE yield near-zero correlations
- On the paired granulocyte ATAC-seq/RNA-seq benchmark, the point-wise correlation between scMMGAN predictions and ground truth must fall in the reported ~0.313-0.358 range (average r=0.336) versus essentially uncorrelated baselines
- On the MDA-MB-231 TNBC benchmark, scMMGAN's per-sample MSE stays in the ~0.87-1.03 range while CycleGAN and CMAE baselines fall in the ~1.6-2.1 range
- The saved diffusion embedding has shape (n_samples x ne) with all eigenvector values within [-1, 1]
- Sanity-check losses reported every 100 training steps (training_counter) should show the correspondence and cycle loss terms behaving stably rather than deteriorating across iterations

## Limitations

- The kernel's gradient-passing design only mitigates, and does not remove, GAN training instability and mode collapse; early stopping and the geometric loss help only informally and these effects may still be present in some contexts
- The geometry-based loss is not completely plug-and-play: a choice of distance between data points is still required
- Altering the eigenvector signs via the correlation check (threshold epsilon_r=-0.5) is needed because eigendecomposition sign ambiguity would otherwise corrupt the loss

## Evidence

- [methods] 10-20 eigenvectors: [evidence span withheld — anchor status: not_found]
