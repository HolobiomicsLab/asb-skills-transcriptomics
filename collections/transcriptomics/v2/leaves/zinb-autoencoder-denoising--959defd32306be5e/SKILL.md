---
name: zinb-autoencoder-denoising
description: Use when pre-training a source classification model on labeled scRNA-seq
  count data and a denoising module that models dropout and count dispersion (ZINB
  NLL) is required before source-free adaptation.
when_to_use_negative:
- Input is already normalized/latent features rather than raw scRNA-seq counts — the
  ZINB likelihood models count data with dropout and dispersion and is not defined
  for non-count inputs
- Raw source count data are unavailable and no pre-trained scEMAIL model weights can
  be loaded — scEMAIL's adaptation stage is source-free and requires only shared model
  weights, not source data
- The goal is annotation without model adaptation (e.g. centroid-similarity tools
  like scmap), which do not use a ZINB autoencoder at all
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scEMAIL
  role: Framework whose source pre-training stage uses the ZINB model-based autoencoder
    with negative log-likelihood reconstruction loss; code at aster-ww/scEMAIL
  repo: https://github.com/aster-ww/scEMAIL
- name: scANVI
  role: Cited as an example autoencoder model the framework can be compatible with
- name: trVAE
  role: Cited as an example autoencoder model the framework can be compatible with
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:3081618bfaad2b6339e0e28d0c971cf12c4d23318931ca972f03fd58650291dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - ItClust
  - Seurat v3
  - scANVI
  - scArches
  - scEMAIL
  - scSemiCluster
  - scmap
  - trVAE
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# zinb-autoencoder-denoising

## Summary

Reconstruct and pre-train the ZINB (zero-inflated negative binomial) model-based denoising autoencoder used in scEMAIL's source model training stage, which depicts the probabilistic structure of scRNA-seq counts via a ZINB negative log-likelihood reconstruction loss.

## When to use

Use when pre-training a source classification model on labeled scRNA-seq count data and a denoising module that models dropout and count dispersion (ZINB NLL) is required before source-free adaptation.

## When NOT to use

- Input is already normalized/latent features rather than raw scRNA-seq counts — the ZINB likelihood models count data with dropout and dispersion and is not defined for non-count inputs
- Raw source count data are unavailable and no pre-trained scEMAIL model weights can be loaded — scEMAIL's adaptation stage is source-free and requires only shared model weights, not source data
- The goal is annotation without model adaptation (e.g. centroid-similarity tools like scmap), which do not use a ZINB autoencoder at all

## Inputs

- Labeled source scRNA-seq gene expression count matrix (cells x genes)
- Source cell-type labels for the supervised cross-entropy classifier
- Raw target-domain count matrices (e.g. scRNA-seq, CITE-seq, or scATAC-seq) for the adaptation stage reusing the ZINB denoising loss

## Outputs

- Trained encoder/decoder weights saved as zinb_autoencoder.pt
- Latent representations z for source (and later target) cells feeding the classifier c and auxiliary classifiers c1/c2
- Finite, monotonically decreasing ZINB negative log-likelihood training curve

## How to apply

Load the labeled source scRNA-seq gene expression count matrix and build an encoder h that projects raw counts into a latent representation z, plus a decoder g that outputs three sets of ZINB parameters per gene: zero-inflation probability omega (dropout event probability), mean mu, and dispersion phi. Compute the ZINB negative log-likelihood reconstruction loss L_zinb averaged over all N_s source cells and p genes, and train the autoencoder with a deep-count-autoencoder-style architecture until the loss converges; then save the encoder and decoder weights (e.g. zinb_autoencoder.pt). The encoder h's latent features then feed the supervised cross-entropy classifier and the two randomly initialized auxiliary classifiers c1/c2 used for the expert ensemble E-score downstream. The ZINB denoising loss is later re-used during target adaptation (classifier weights frozen, only encoder fine-tuned) together with the multi-order local affinity loss over bidirectional, unidirectional (alpha = 0.1), and extensional neighbors to implicitly mitigate batch effect without accessing raw source data.

## Related tools

- **scEMAIL** (Framework whose source pre-training stage uses the ZINB model-based autoencoder with negative log-likelihood reconstruction loss; code at aster-ww/scEMAIL) — https://github.com/aster-ww/scEMAIL
- **scANVI** (Cited as an example autoencoder model the framework can be compatible with)
- **trVAE** (Cited as an example autoencoder model the framework can be compatible with)

## Evaluation signals

- The training script runs without error and zinb_autoencoder.pt contains encoder and decoder weights
- ZINB negative log-likelihood loss is finite and monotonically decreasing during source pre-training
- Decoder emits all three ZINB parameter sets per gene: zero-inflation probability omega, mean mu, and dispersion phi
- Total source loss (ZINB NLL + cross-entropy + auxiliary co-training losses, Eq. 5) converges and downstream E-score components (normalized entropy, maximum confidence, consistency) are computable from the trained encodings
- Reusing the ZINB denoising loss during target fine-tuning (classifier frozen, encoder-only updates) yields benchmark accuracies consistent with the paper (e.g. bone marrow 0.8873, trachea 0.9415)

## Limitations

- The ZINB likelihood is defined for count-like scRNA-seq data; extending to scATAC-seq/CITE-seq relies on the paper's demonstrated flexibility and may not preserve the same probabilistic fit
- Detailed implementation settings are deferred to the paper's File S1 / supplementary materials, not fully specified in the main text
- The article's discussion section does not state explicit method limitations, and the scATAC-seq accuracy values appear only graphically (Figure 10), limiting numeric validation of the adaptation stage

## Evidence

- [methods] Then, we adopt a ZINB model-based autoencoder to reveal the global probabilistic structure of scRNA-seq data.: "Then, we adopt a ZINB model-based autoencoder to reveal the global probabilistic structure of scRNA-seq data."
- [intro] the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data.: "the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data."
- [methods] we further introduce two auxiliary classifiers cj, j = 1, 2, which also take features from h for further uncertainty measurement: "we further introduce two auxiliary classifiers cj, j = 1, 2, which also take features from h for further uncertainty measurement"
- [readme] scEMAIL is accurate and robust under various scenarios in both simulation and real data.: "scEMAIL is accurate and robust under various scenarios in both simulation and real data."
