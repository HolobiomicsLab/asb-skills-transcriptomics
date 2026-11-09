---
name: dirichlet-latent-space-encoding
description: Use when scRNA-seq counts from batched experiments (e.g. NEURIPS bone
  marrow data) need topic compositions and an unconfounded latent representation that
  separates technical covariates from biological cell states.
when_to_use_negative:
- Input is not raw count data — deviance residuals and the Dirichlet-multinomial topic
  model require unnormalized counts, not already-normalized expression matrices
- Single-batch data with no technical covariates to disentangle, where standard MIRA
  topic modeling without the CODAL augmentation suffices
- Data where batch-confounded cell states cannot be validated with orthogonal biological
  evidence and under-correction risk is unacceptable
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CODAL
  role: Variational autoencoder with mutual information regularization that disentangles
    technical from biological effects in the Dirichlet topic latent space
  repo: https://github.com/cistrome/MIRA
- name: MIRA
  role: Provides the underlying Dirichlet variational topic model architecture (mira.topics.make_model)
    that CODAL extends and all MIRA topic models use for inference
  repo: https://github.com/cistrome/MIRA
- name: scanpy
  role: 'Preprocessing of scRNA-seq counts: filtering, normalize_total, log+1, and
    highly variable gene selection before encoding'
- name: PyTorch
  role: Deep learning backend for the encoder and the spectrally-normalized MINE estimator
    Tθ
- name: Pyro
  role: Probabilistic programming dependency supporting the variational inference
    implementation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:97252401fbad5a9bc50c27910a209bfa652240b06bb96b2dc056964723295123
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - MIRA
  - PyTorch
  - Pyro
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# dirichlet-latent-space-encoding

## Summary

Encodes single-cell RNA-seq counts into an interpretable Dirichlet topic latent space using MIRA's Dirichlet variational encoder (total pseudocount 𝓘=50, Ntopics=22), as extended by CODAL to disentangle technical from biological effects via mutual information regularization.

## When to use

Use when scRNA-seq counts from batched experiments (e.g. NEURIPS bone marrow data) need topic compositions and an unconfounded latent representation that separates technical covariates from biological cell states.

## When NOT to use

- Input is not raw count data — deviance residuals and the Dirichlet-multinomial topic model require unnormalized counts, not already-normalized expression matrices
- Single-batch data with no technical covariates to disentangle, where standard MIRA topic modeling without the CODAL augmentation suffices
- Data where batch-confounded cell states cannot be validated with orthogonal biological evidence and under-correction risk is unacceptable

## Inputs

- Raw scRNA-seq count matrix (e.g. NEURIPS bone marrow) in AnnData/scanpy format
- One-hot encoded technical covariates C (sequencing site, donor)
- Number of topics (e.g. Ntopics=22) and total pseudocount 𝓘=50 model configuration

## Outputs

- Posterior topic compositions Z̄ (Ncells × 22 topics)
- β topic-feature weight matrix (22 × 3500 genes)
- Technical effect vectors t = hϕ(Z, C) (Ncells × 3500 genes)
- Converged CODAL objective score on held-out cells

## How to apply

Preprocess counts with scanpy (filter cells <400 and genes <30 counts, normalize_total target_sum=10000, log+1, select 3500 highly variable genes at minimum dispersion 0.3), then transform raw counts to deviance residuals before the encoder. Instantiate the CODAL expression topic model via mira.topics.make_model as a MIRA-based Dirichlet variational encoder with total pseudocount 𝓘=50 and Ntopics=22 (dropout tuned in [0.05, 0.1]), encoding one-hot technical covariates (sequencing site, donor) as C. Train with cyclical stochastic minibatch gradient ascent (minibatch 128, 24 epochs, AdamW with one-cycle lr/momentum policy annealing momentum 0.85–0.95), alternating updates of the MINE estimator Tθ — a spectrally-normalized 1-Lipschitz network with a 64-node hidden layer, Adam lr=1e-4 — followed by spectral normalization each step. The Dirichlet encoding yields posterior topic compositions Z̄ per cell plus the β topic-gene weight matrix, while the CODAL objective (a differentiable approximation of mutual information plus marginal likelihood) removes technical effects from the latent space.

## Related tools

- **CODAL** (Variational autoencoder with mutual information regularization that disentangles technical from biological effects in the Dirichlet topic latent space) — https://github.com/cistrome/MIRA
- **MIRA** (Provides the underlying Dirichlet variational topic model architecture (mira.topics.make_model) that CODAL extends and all MIRA topic models use for inference) — https://github.com/cistrome/MIRA
- **scanpy** (Preprocessing of scRNA-seq counts: filtering, normalize_total, log+1, and highly variable gene selection before encoding)
- **PyTorch** (Deep learning backend for the encoder and the spectrally-normalized MINE estimator Tθ)
- **Pyro** (Probabilistic programming dependency supporting the variational inference implementation)

## Examples

```
import mira.topics; model = mira.topics.make_model(adata.n_vars, 22, feature_type='expression', total_counts=50).batch_and_covariate(adata, 'site', 'donor')
```

## Evaluation signals

- Serialized arrays have expected shapes: Ncells × 22 topics for Z̄, 22 × 3500 genes for β, and Ncells × 3500 genes for technical effect vectors
- Trained model reaches a converged CODAL objective score evaluated on held-out cells
- Latent topic compositions show batch-confounded cell states resolved rather than split by sequencing site or donor
- MINE estimator Tθ maintains 1-Lipschitz continuity via spectral normalization after each update step
- Hyperparameter trials pass the pruning rungs at the 8th and 16th epoch without falling in the bottom 50th percentile of scores

## Limitations

- New batch-confounded or perturbation-induced cell states should be validated using orthogonal biological evidence to rule out under-correction of technical effects
- CODAL does not remove the need for good experimental design; experimentalists must include appropriate controls (e.g. control cells unaffected by the perturbation in both treatment and control batches)
- Further improvements require better characterization of technical sources of bias in conjunction with the CODAL model

## Evidence

- [abstract] CODAL uses mutual information regularization to explicitly disentangle technical and biological effects: "uses a mutual information regularization technique to explicitly disentangle factors related to technical and biological effects"
- [methods] MINE estimator with spectral normalization: "We employ spectral normalization59 to enforce 1-Lipschitz continuity of the neural network estimator Tθ"
- [methods] Deviance residual preprocessing: "Raw expression counts are transformed using deviance residuals61 before being fed through the encoder model."
- [intro] Batch proxy assumption: "the cells’ batch of origin can be used as a proxy variable to indicate which cells were subjected to a protocol using common reagents and under similar conditions."
- [readme] MIRA README on CODAL inference: [evidence span withheld — anchor status: external_doc]
