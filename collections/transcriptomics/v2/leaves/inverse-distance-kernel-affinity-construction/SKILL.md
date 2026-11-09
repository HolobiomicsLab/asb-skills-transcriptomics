---
name: inverse-distance-kernel-affinity-construction
description: 'Use when preparing inputs for scMMGAN''s geometry-preserving correspondence
  loss: pairwise L1 distances on preprocessed modality matrices (MAGIC-corrected,
  zero-centered/unit-scaled, 50 principal components) must be converted to affinities
  via the inverse-distance kernel with adaptive kNN bandwidth.'
when_to_use_negative:
- If no meaningful choice of distance between data points exists for the modality
  at hand — the article notes the geometry-based loss 'is not completely plug-and-play'
  and requires a distance metric
- If the input is already an affinity or transition-probability matrix, in which case
  kernel construction would double-transform the data
- If the data have not been preprocessed (MAGIC dropout correction, zero-centering/unit
  scaling, 50-PC reduction) — raw columns can fall outside the [-10, 10] interval
  the README says gives best results
tools:
- name: scMMGAN
  role: Framework whose diffusion geometry loss consumes the affinity/transition matrices
    built by this skill
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Graph construction environment in which the differentiable kernel, row normalization,
    and downstream eigenvector loss are computed
- name: MAGIC
  role: Upstream manifold-smoothing dropout correction that produces the input matrices
    the affinities are computed on
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:7edfb3bbd1c3b257b946f7dc62a4927afa2904d81a2789f16dd481802495e77f
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
yes_rate: 0.7142857142857143
n_claims: 7
---

# inverse-distance-kernel-affinity-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs pairwise affinity matrices for scMMGAN's diffusion-geometry correspondence loss by applying the inverse-distance kernel k(xi,xj)=max(0,1−||xi−xj||1/σi) with a k-nearest-neighbor adaptive bandwidth σi, then row-normalizing the affinities into Markov transition probability matrices used for diffusion-coordinate extraction.

## When to use

Use when preparing inputs for scMMGAN's geometry-preserving correspondence loss: pairwise L1 distances on preprocessed modality matrices (MAGIC-corrected, zero-centered/unit-scaled, 50 principal components) must be converted to affinities via the inverse-distance kernel with adaptive kNN bandwidth σi and row-normalized into transition matrices.

## When NOT to use

- If no meaningful choice of distance between data points exists for the modality at hand — the article notes the geometry-based loss 'is not completely plug-and-play' and requires a distance metric
- If the input is already an affinity or transition-probability matrix, in which case kernel construction would double-transform the data
- If the data have not been preprocessed (MAGIC dropout correction, zero-centering/unit scaling, 50-PC reduction) — raw columns can fall outside the [-10, 10] interval the README says gives best results

## Inputs

- Preprocessed modality matrix x1 (dropout-corrected with MAGIC, zero-centered/unit-scaled, reduced to 50 principal components), as a size-256 minibatch
- Generator output G(x1) from the scMMGAN mapping, same dimensions as x1

## Outputs

- Pairwise distance matrix (L1 metric) for each of x1 and G(x1)
- Pairwise affinity matrix computed with the inverse-distance kernel and adaptive bandwidth σi
- Row-normalized Markov transition probability matrices Px1 and PG(x1) ready for diffusion-eigenvector extraction

## How to apply

For each minibatch (e.g. x1 of size 256) and its generator output G(x1), build a matrix of pairwise distances and transform it into pairwise affinities with the inverse-distance kernel k(xi,xj)=max(0,1−||xi−xj||1/σi), where σi is a k-nearest-neighbor adaptive bandwidth set per data point. Truncation via max(0,·) guarantees non-negative affinities and sparsifies long-range connections. Row-normalize the affinity matrices (Px1 and PG(x1)) into Markov chain transition probability matrices via P=(∑jAij)−1A, computed within the TensorFlow graph so the loss is differentiable. The same kernel is applied identically to both the original and mapped points so the subsequent diffusion-eigenvector correspondence comparison measures geometry preservation by the generator, not differences in graph construction. The resulting transition matrices feed extraction of the first 10–20 diffusion eigenvectors used in the Lc loss weighted with λc=10.

## Related tools

- **scMMGAN** (Framework whose diffusion geometry loss consumes the affinity/transition matrices built by this skill) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Graph construction environment in which the differentiable kernel, row normalization, and downstream eigenvector loss are computed)
- **MAGIC** (Upstream manifold-smoothing dropout correction that produces the input matrices the affinities are computed on)

## Examples

```
import numpy as np
D=np.abs(x1[:,None]-x1[None,:]).sum(-1); sig=np.sort(D,1)[:,10]; A=np.maximum(0,1-D/sig[:,None])
```

## Evaluation signals

- All entries of the affinity matrix are non-negative due to the max(0, 1−||xi−xj||1/σi) truncation
- Rows of the transition matrices Px1 and PG(x1) each sum to 1 after row normalization P=(∑jAij)−1A
- σi is strictly positive and varies per data point (adaptive kNN bandwidth, not a global constant)
- Affinity construction is applied identically to x1 and G(x1), so downstream eigenvector-sign checks (εr=−0.5) operate on comparable diffusion coordinates
- Downstream correspondence loss values logged every 100 steps via training_counter are finite and non-negative

## Limitations

- The geometry-based loss is not completely 'plug-and-play': it requires a choice of distance between data points, so kernel behavior depends on the L1 metric choice
- Only the first 10–20 diffusion eigenvectors carry the correspondence constraint, so geometry preservation is enforced at coarse diffusion scales only
- σi selection (kNN adaptive bandwidth) is a hyperparameter whose value is not fixed by the article and must be tuned per dataset

## Evidence
