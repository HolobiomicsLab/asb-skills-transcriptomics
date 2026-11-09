---
name: eigenvector-sign-correlation-check
description: Use when computing a correspondence or geometry-preserving loss over
  diffusion map eigenvectors, or comparing eigendecompositions of two related Markov
  transition matrices, where eigenvector sign is arbitrary.
when_to_use_negative:
- When the eigenvectors being compared come from unrelated transition matrices with
  no correspondence loss enforced between them — the check only corrects alignment
  for the geometry-preserving loss.
- When the eigenvector values have not been computed within the TensorFlow graph of
  scMMGAN (the check is part of the correspondence loss computation there, not a standalone
  preprocessing step).
tools:
- name: scMMGAN
  role: Framework in which the sign-correlation check is applied to diffusion eigenvectors
    before computing the correspondence loss
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Graph construction within which the eigenvector comparison and sign flip are
    implemented so gradients flow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:51a8604a1ac82938f846175ca9cbd460da13edef0e1c6858603cdab4dc9337fd
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# eigenvector-sign-correlation-check

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A sign-alignment step in the scMMGAN diffusion geometry pipeline that checks the correlation between eigenvector pairs (e.g., eigenvectors of the data-domain transition matrix P versus eigenvectors of the generator output) and flips the sign of an eigenvector when the correlation falls below the threshold εr = −0.5, so that the correspondence loss is not computed between anti-aligned eigenvectors.

## When to use

Use when computing a correspondence or geometry-preserving loss over diffusion map eigenvectors, or comparing eigendecompositions of two related Markov transition matrices, where eigenvector sign is arbitrary.

## When NOT to use

- When the eigenvectors being compared come from unrelated transition matrices with no correspondence loss enforced between them — the check only corrects alignment for the geometry-preserving loss.
- When the eigenvector values have not been computed within the TensorFlow graph of scMMGAN (the check is part of the correspondence loss computation there, not a standalone preprocessing step).

## Inputs

- Eigenvector matrix of the reference data-domain diffusion operator (n_samples × ne, e.g. diffusion_eigenvectors.npy)
- Eigenvector matrix of the compared domain, e.g. φG(x) from the generator output in the TensorFlow graph
- Sign-correlation threshold εr = −0.5

## Outputs

- Sign-corrected eigenvector matrix (n_samples × ne) with values rescaled to [−1, 1]
- Correspondence/geometry loss computed on aligned eigenvector pairs

## How to apply

After computing the diffusion map eigenvectors (the first 10–20 eigenvectors of the row-normalized transition probability matrix P built from the pairwise affinity matrix), correlate each eigenvector of the target domain (e.g., φG(x) from the generator output) with the corresponding eigenvector of the reference domain. If the correlation is below the threshold εr = −0.5, multiply the values of that eigenvector (φG(x)) by −1 before computing the loss. This is necessary because eigenvectors are defined only up to sign, so uncorrected sign flips would make the correspondence loss spuriously large even for a perfect mapping. In the scMMGAN TensorFlow implementation this check is applied to the eigenvector pairs before the geometry loss term, and the eigenvectors are subsequently rescaled to lie within [−1, 1].

## Related tools

- **scMMGAN** (Framework in which the sign-correlation check is applied to diffusion eigenvectors before computing the correspondence loss) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Graph construction within which the eigenvector comparison and sign flip are implemented so gradients flow)

## Examples

```
if np.corrcoef(phi_P[:, i], phi_G[:, i])[0, 1] < -0.5: phi_G[:, i] *= -1
```

## Evaluation signals

- After the sign flip, the correlation of each eigenvector pair is ≥ εr = −0.5 (no remaining strongly negative correlations between matched eigenvectors).
- The output embedding has shape (n_samples × ne) with ne = 10–20 eigenvectors retained.
- All corrected eigenvector values fall within [−1, 1] after rescaling.
- The correspondence loss value is not spuriously inflated by anti-aligned eigenvectors — flipping signs should reduce or leave unchanged the loss for matched eigenvectors.

## Limitations

- The check only corrects sign; eigenvectors matched by index may still correspond to different diffusion modes if the underlying manifolds diverge, and the loss would then still be uninformative.
- Threshold εr = −0.5 is a fixed heuristic from the paper; the geometry-based loss is 'not completely plug-and-play' and still requires a choice of distance between data points.

## Evidence

- [methods] Correspondence loss on first 10–20 eigenvectors: [evidence span withheld — anchor status: not_found]
