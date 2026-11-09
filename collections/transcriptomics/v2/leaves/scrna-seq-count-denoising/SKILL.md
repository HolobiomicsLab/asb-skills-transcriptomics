---
name: scrna-seq-count-denoising
description: Use when pre-training a source classification model on raw scRNA-seq
  count data where dropout events and overdispersion require a probabilistic denoising
  loss (ZINB negative log-likelihood) rather than mean-squared reconstruction.
when_to_use_negative:
- Input data are already normalized or denoised (e.g., log-transformed expression
  or a feature/embedding table) rather than raw counts, since the ZINB likelihood
  models raw count dropout and dispersion.
- The downstream task requires no transfer learning or denoising — e.g., annotating
  a single dataset with no source model pre-training stage, where a plain classifier
  or clustering method suffices.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: Framework that uses the ZINB-based denoising autoencoder during source model
    pre-training, combining ZINB negative log-likelihood loss with supervised cross-entropy
    classification before sharing only pre-trained model weights.
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:ef8583a676c86854c5aa71ebc3ca8aa4a9473085fa320520a0cf37ab4309e1f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# scrna-seq-count-denoising

## Summary

Denoises raw scRNA-seq gene expression count matrices by training a ZINB (zero-inflated negative binomial) model-based denoising autoencoder, as used in scEMAIL's source model pre-training stage to capture the probabilistic structure of scRNA-seq data.

## When to use

Use when pre-training a source classification model on raw scRNA-seq count data where dropout events and overdispersion require a probabilistic denoising loss (ZINB negative log-likelihood) rather than mean-squared reconstruction.

## When NOT to use

- Input data are already normalized or denoised (e.g., log-transformed expression or a feature/embedding table) rather than raw counts, since the ZINB likelihood models raw count dropout and dispersion.
- The downstream task requires no transfer learning or denoising — e.g., annotating a single dataset with no source model pre-training stage, where a plain classifier or clustering method suffices.

## Inputs

- Source scRNA-seq gene expression count matrix (cells x genes raw counts)
- Per-cell gene expression values x_ij for the denoising module

## Outputs

- Trained encoder h and decoder g weights (e.g., zinb_autoencoder.pt) for downstream source-free adaptation
- Per-gene ZINB parameters (zero-inflation probability omega, mean mu, dispersion phi) from the decoder
- Converged ZINB negative log-likelihood reconstruction loss L_zinb

## How to apply

Load the source scRNA-seq gene expression count matrix and represent each cell's gene expression as x_ij for the denoising module. Build an encoder h that projects raw counts into a latent representation z, and a decoder g that outputs three sets of ZINB parameters per gene: zero-inflation probability omega (dropout event probability), mean mu, and dispersion phi. Compute the ZINB negative log-likelihood reconstruction loss L_zinb averaged over all N_s cells and p genes, and train the autoencoder with a deep-count-autoencoder-style architecture until this loss converges. In scEMAIL, this ZINB denoising loss is combined with a supervised cross-entropy loss for source classification, and the trained encoder/decoder weights (e.g., saved as zinb_autoencoder.pt) are later reused for source-free adaptation. The rationale is that the ZINB distribution explicitly models the excess zeros and count overdispersion characteristic of scRNA-seq data, revealing the global probabilistic structure.

## Related tools

- **scEMAIL** (Framework that uses the ZINB-based denoising autoencoder during source model pre-training, combining ZINB negative log-likelihood loss with supervised cross-entropy classification before sharing only pre-trained model weights.) — https://github.com/aster-ww/scEMAIL

## Examples

```
python train_zinb_autoencoder.py --input source_counts.csv --output zinb_autoencoder.pt  # trains encoder h / decoder g by minimizing ZINB NLL until convergence
```

## Evaluation signals

- The training script runs without error and saves zinb_autoencoder.pt containing both encoder and decoder weights.
- The ZINB negative log-likelihood reconstruction loss L_zinb is finite and monotonically decreases over training epochs until convergence.
- The decoder outputs three well-formed parameter sets per gene (omega, mu, phi) with valid ranges (probabilities in [0,1], positive mean and dispersion).
- Combining the ZINB denoising loss with cross-entropy classification yields a source model whose pre-trained weights transfer correctly to the downstream source-free adaptation stage.

## Limitations

- The ZINB distribution assumes dropout and overdispersion structure appropriate to scRNA-seq counts; it may not suit data types whose generative process differs, although scEMAIL reports the framework can extend to scATAC-seq data.
- Implementation details of the autoencoder architecture are deferred to File S1 / the repository, so exact layer sizes and hyperparameters are not specified in the main article text.

## Evidence

- [methods] ZINB autoencoder depicts probabilistic structure: "Then, we adopt a ZINB model-based autoencoder to reveal the global probabilistic structure of scRNA-seq data."
- [intro] ZINB denoising loss in adaptation stage: "the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data."
- [intro] Source model trained on well-labeled data: "Generally, we first utilize source data to train the source classification model and then only apply this source model to annotate target data."
- [readme] Repository availability: "The source code of scEMAIL can be accessed at https://github.com/aster-ww/scEMAIL"
