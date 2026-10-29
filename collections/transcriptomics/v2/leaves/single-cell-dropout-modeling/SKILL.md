---
name: single-cell-dropout-modeling
description: Use when pre-training or adapting a single-cell classification model
  on raw scRNA-seq count matrices and a loss must capture the mean-dispersion-dropout
  structure (zero-inflation) of gene expression counts.
when_to_use_negative:
- Input data are not raw scRNA-seq counts (e.g. pre-normalized or log-transformed
  expression), since the ZINB likelihood assumes discrete count structure with dropout
  events
- The goal is purely unsupervised clustering without a source classification model,
  where the ZINB denoising loss of scEMAIL's source pre-training stage is not the
  driving component
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scEMAIL
  role: Framework whose source model pre-training and target adaptation stages use
    the ZINB model-based autoencoder and its negative log-likelihood denoising loss
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:11c916724a782ba94aa5cd7d980cb6987d6d483a7ef7963f6f28d4e3757f4cc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-dropout-modeling

## Summary

Reconstruct the ZINB model-based denoising autoencoder that scEMAIL uses in source model pre-training and target adaptation, minimizing the ZINB negative log-likelihood to depict the global probabilistic structure of scRNA-seq count data, including its dropout (zero-inflation) component.

## When to use

Use when pre-training or adapting a single-cell classification model on raw scRNA-seq count matrices and a loss must capture the mean-dispersion-dropout structure (zero-inflation) of gene expression counts.

## When NOT to use

- Input data are not raw scRNA-seq counts (e.g. pre-normalized or log-transformed expression), since the ZINB likelihood assumes discrete count structure with dropout events
- The goal is purely unsupervised clustering without a source classification model, where the ZINB denoising loss of scEMAIL's source pre-training stage is not the driving component

## Inputs

- Source scRNA-seq gene expression count matrix (cells x genes, raw counts)
- Cell labels for the source data (for the coupled supervised cross-entropy loss)
- Target scRNA-seq count matrix (when the ZINB loss is reused during source-free model adaptation)

## Outputs

- Trained encoder/decoder weights (zinb_autoencoder.pt containing encoder and decoder state dicts)
- Per-gene ZINB parameters: zero-inflation probability omega, mean mu, dispersion phi
- ZINB negative log-likelihood reconstruction loss trajectory (L_zinb averaged over N_s cells and p genes)
- Latent representation z used downstream for the source classifier and E-score uncertainty measurement

## How to apply

Load the source scRNA-seq gene expression count matrix and represent each cell's gene expression x_ij for the denoising module. Build an encoder h that projects raw counts into a latent representation z, and a decoder g that outputs, per gene, the three ZINB parameters: zero-inflation probability omega (dropout event probability), mean mu, and dispersion phi. Compute the ZINB negative log-likelihood reconstruction loss L_zinb averaged over all N_s cells and p genes, and train the autoencoder with a deep-count-autoencoder-style architecture until the reconstruction loss converges; the encoder weights then feed the source classifier, which is trained with cross-entropy loss plus two auxiliary co-training classifiers. During target adaptation, the same ZINB denoising loss is combined with multi-order local affinity constraints to implicitly mitigate batch effect between source and target data, without accessing raw source data.

## Related tools

- **scEMAIL** (Framework whose source model pre-training and target adaptation stages use the ZINB model-based autoencoder and its negative log-likelihood denoising loss) — https://github.com/aster-ww/scEMAIL

## Evaluation signals

- Training script runs without error and zinb_autoencoder.pt contains both encoder and decoder weights
- ZINB negative log-likelihood loss is finite and monotonically decreasing during training until convergence
- Decoder emits exactly three parameter sets per gene (omega, mu, phi) matching the input gene dimension p
- Source classification model jointly trained with the ZINB loss and cross-entropy loss (plus two auxiliary classifiers) achieves plausible annotation performance on target data (e.g. the accuracy/H-score ranges reported for the bone marrow, pancreas, trachea, and mammary gland dataset pairs)
- Reuse of the ZINB loss during target adaptation, together with multi-order local affinity constraints, reduces misclassifications attributable to batch effect between source and target data

## Limitations

- Implementation details of scEMAIL (including ZINB autoencoder specifics) are deferred to File S1 rather than the main methods text
- The ZINB denoising loss only implicitly mitigates batch effect between source and target data; explicit feature alignment is impossible in the source-free setting because raw source data are not accessed
- The framework is validated for scRNA-seq and single-cell ATAC-seq data; behavior on other modalities is only preliminarily illustrated via paired CITE-seq experiments

## Evidence

- [methods] ZINB autoencoder reveals probabilistic structure: "Then, we adopt a ZINB model-based autoencoder to reveal the global probabilistic structure of scRNA-seq data."
- [intro] ZINB loss used for adaptation: "the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data."
- [intro] Source model training precedes source-free annotation: "Generally, we first utilize source data to train the source classification model and then only apply this source model to annotate target data."
- [readme] Robustness motivation from README: "scEMAIL is accurate and robust under various scenarios in both simulation and real data."
