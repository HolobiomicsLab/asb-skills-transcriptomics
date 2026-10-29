---
name: negative-log-likelihood-loss-optimization
description: Use when pre-training a source classification model on raw scRNA-seq
  count matrices where dropout/dropout events require a probabilistic denoising loss
  (ZINB NLL) alongside supervised classification loss.
when_to_use_negative:
- Input is already normalized/log-transformed expression rather than raw integer counts
  — the ZINB likelihood models raw count structure
- Source data are unavailable or cannot be accessed — scEMAIL's source pre-training
  stage requires well-labeled source data; only the pre-trained weights are shared
  downstream
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scEMAIL
  role: Implements the ZINB model-based autoencoder and the joint ZINB NLL + cross-entropy
    source model pre-training
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:a2edd3438aa23b452c1bec428acd8f8756e4ab1d07a644a1d0b206a2f52e65d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# negative-log-likelihood-loss-optimization

## Summary

Optimization of a ZINB negative log-likelihood reconstruction loss in scEMAIL's source model pre-training, where a ZINB model-based denoising autoencoder depicts the probabilistic structure of scRNA-seq count data. Combined with supervised cross-entropy, it yields denoised latent representations and pre-trained encoder/decoder weights shared downstream.

## When to use

Use when pre-training a source classification model on raw scRNA-seq count matrices where dropout/dropout events require a probabilistic denoising loss (ZINB NLL) alongside supervised classification loss.

## When NOT to use

- Input is already normalized/log-transformed expression rather than raw integer counts — the ZINB likelihood models raw count structure
- Source data are unavailable or cannot be accessed — scEMAIL's source pre-training stage requires well-labeled source data; only the pre-trained weights are shared downstream

## Inputs

- Source scRNA-seq gene expression count matrix (N_s cells x p genes, raw integer counts)
- Source cell-type labels for the supervised cross-entropy component

## Outputs

- Trained ZINB autoencoder weights (zinb_autoencoder.pt with encoder h and decoder g)
- Per-gene ZINB parameters (zero-inflation probability omega, mean mu, dispersion phi)
- Pre-trained source classification model weights (including two auxiliary classifiers) for source-free transfer

## How to apply

Load the source scRNA-seq gene expression count matrix and represent each cell's gene expression x_ij for the denoising module. Build an encoder h that projects raw counts into a latent representation z, and a decoder g that outputs three sets of ZINB parameters per gene: zero-inflation probability omega (dropout event probability), mean mu, and dispersion phi. Compute the ZINB negative log-likelihood reconstruction loss L_zinb averaged over all N_s cells and p genes, and minimize it jointly with the supervised cross-entropy classification loss (with two auxiliary classifiers c_j, j=1,2 taking features from h). Train the deep-count-autoencoder-style architecture until the reconstruction loss converges, then save the encoder/decoder weights for later source-free adaptation. The rationale is that modeling counts via the zero-inflated negative binomial distribution reveals the global probabilistic structure of scRNA-seq data and denoises dropout before the source model is shared.

## Related tools

- **scEMAIL** (Implements the ZINB model-based autoencoder and the joint ZINB NLL + cross-entropy source model pre-training) — https://github.com/aster-ww/scEMAIL

## Evaluation signals

- Training script runs without error and zinb_autoencoder.pt contains both encoder and decoder weights
- ZINB negative log-likelihood loss is finite and monotonically decreasing during pre-training
- Reconstruction loss converges before the encoder/decoder weights are saved
- Joint objective balances L_zinb with cross-entropy classification loss; auxiliary classifiers c_1 and c_2 both train from features of h

## Limitations

- Implementation details are deferred to File S1, so exact hyperparameters of the ZINB autoencoder must be obtained from the supplement or repository
- The ZINB denoising loss is applied only during source pre-training and target adaptation with source data; raw source data availability is assumed at this stage even though adaptation itself is source-free

## Evidence

- [methods] Then, we adopt a ZINB model-based autoencoder to reveal the global probabilistic structure of scRNA-seq data.: "Then, we adopt a ZINB model-based autoencoder to reveal the global probabilistic structure of scRNA-seq data."
- [intro] the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data.: "the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data."
- [methods] we further introduce two auxiliary classifiers cj, j = 1, 2, which also take features from h for further uncertainty measurement: "we further introduce two auxiliary classifiers cj, j = 1, 2, which also take features from h for further uncertainty measurement"
- [intro] Generally, we first utilize source data to train the source classification model and then only apply this source model to annotate target data.: "Generally, we first utilize source data to train the source classification model and then only apply this source model to annotate target data."
