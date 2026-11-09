---
name: variational-topic-model-training
description: Use when fitting an interpretable topic model to multi-batch single-cell
  expression data where technical effects (batch, donor, sequencing site) must be
  disentangled from biological cell states, or when latent cell representations are
  needed for downstream clustering.
when_to_use_negative:
- When the analysis goal is not disentangling technical from biological effects —
  an entangled ELBO-only control (MI regularization weight set to zero) is the appropriate
  baseline for comparison, not the target model
- When data are not raw count matrices (e.g. already normalized feature tables), since
  the pipeline expects count filtering, deviance-residual transformation, and count-based
  topic modeling
- When single-condition, single-batch data with no technical covariates are available
  — the CODAL covariate-disentangling machinery has no batch structure to exploit
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MIRA
  role: Provides the Dirichlet variational topic-model architecture that CODAL extends;
    mira.topics.make_model instantiates the expression topic model
  repo: https://github.com/cistrome/MIRA
- name: CODAL
  role: The COvariate Disentangling Augmented Loss objective trained on the topic
    model to disentangle technical from biological effects
  repo: https://github.com/cistrome/MIRA
- name: scanpy
  role: Count filtering, normalize_total (target_sum=10000), log+1 transform, and
    highly variable gene selection preprocessing
- name: PyTorch
  role: AdamW optimizer, spectral normalization for the 1-Lipschitz MINE estimator,
    and minibatch training loop
- name: Pyro
  role: Probabilistic programming backend for the variational inference in MIRA topic
    models
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:49fafccb6ab27e0996fe8f7d33e4091f84fc8bd5ddf56b6a2a0d2e371abcc2f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - MIRA
  - PyTorch
  - Pyro
  - UMAP-learn
  - scanpy
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# variational-topic-model-training

## Summary

Train a CODAL/MIRA-based Dirichlet variational topic model on scRNA-seq count data using cyclical stochastic minibatch gradient ascent with a MINE mutual-information regularizer, yielding interpretable topic compositions (Z̄), a topic-feature β matrix, and technical effect vectors t = hϕ(Z, C).

## When to use

Use when fitting an interpretable topic model to multi-batch single-cell expression data where technical effects (batch, donor, sequencing site) must be disentangled from biological cell states, or when latent cell representations are needed for downstream clustering.

## When NOT to use

- When the analysis goal is not disentangling technical from biological effects — an entangled ELBO-only control (MI regularization weight set to zero) is the appropriate baseline for comparison, not the target model
- When data are not raw count matrices (e.g. already normalized feature tables), since the pipeline expects count filtering, deviance-residual transformation, and count-based topic modeling
- When single-condition, single-batch data with no technical covariates are available — the CODAL covariate-disentangling machinery has no batch structure to exploit

## Inputs

- scRNA-seq count matrix (AnnData/h5ad) from multi-batch experiments, e.g. NEURIPS 2021 bone marrow or mouse gastrulation atlas
- Per-cell batch/technical covariate annotations (sequencing site, donor, chimeric status, collection day) as one-hot encoded variables
- Deviance-residual-transformed expression counts fed to the encoder

## Outputs

- Posterior topic composition matrix Z̄ (Ncells × Ntopics)
- β topic-feature weight matrix (Ntopics × Ngenes)
- Technical effect vectors t = hϕ(Z, C) (Ncells × Ngenes)
- Predicted expression-rate estimates (Ncells × Ngenes)
- Serialized model and converged CODAL objective score on held-out cells

## How to apply

Preprocess counts with the scanpy workflow (filter cells <400 counts and genes <30 counts, normalize_total target_sum=10000, log+1 transform, select highly variable genes at minimum dispersion 0.3, e.g. 3500 genes for NEURIPS bone marrow) and transform raw counts to deviance residuals before the encoder. Encode technical covariates (e.g. sequencing site and donor) as one-hot variables C, then instantiate the model via mira.topics.make_model as a MIRA Dirichlet variational encoder with total pseudocount I=50 and the chosen number of topics (22 for NEURIPS tuned over 15–40; 63 for the mouse gastrulation atlas tuned over 35–80 via 60 Bayesian iterations with a Gaussian Process/Matern 5/2 kernel). Train by cyclical stochastic minibatch gradient ascent (minibatch size 128, AdamW, one-cycle learning-rate/momentum policy annealing momentum 0.85–0.95, 24 epochs with 3-cycle annealing of the ε1 KL and ε2 MINE weights), pruning trials at rungs after the 8th and 16th epoch when in the bottom 50th percentile of scores. Each step update the spectrally-normalized 1-Lipschitz MINE estimator Tθ (64-node hidden layer, Adam lr=1e-4), followed by spectral normalization. After training, extract posterior topic compositions Z̄, the β topic-feature weight matrix, and technical effect vectors t = hϕ(Z, C); optionally apply dropout regularization of technical effects via Bernoulli corruption at p=0.05.

## Related tools

- **MIRA** (Provides the Dirichlet variational topic-model architecture that CODAL extends; mira.topics.make_model instantiates the expression topic model) — https://github.com/cistrome/MIRA
- **CODAL** (The COvariate Disentangling Augmented Loss objective trained on the topic model to disentangle technical from biological effects) — https://github.com/cistrome/MIRA
- **scanpy** (Count filtering, normalize_total (target_sum=10000), log+1 transform, and highly variable gene selection preprocessing)
- **PyTorch** (AdamW optimizer, spectral normalization for the 1-Lipschitz MINE estimator, and minibatch training loop)
- **Pyro** (Probabilistic programming backend for the variational inference in MIRA topic models)

## Examples

```
conda create --name mira-env -c conda-forge -c pytorch -c bioconda scanpy jupyter leidenalg; conda activate mira-env; pip3 install mira-multiome
```

## Evaluation signals

- Serialized arrays have expected shapes: Ncells × 22 topics (compositions), 22 × 3500 genes (β matrix), Ncells × 3500 genes (technical effects / expression rates)
- Converged CODAL objective score evaluated on held-out cells
- Technical effect predictions are independent of predicted expression rates for marker genes (e.g. Ccl5, Tcf7l2), unlike the ELBO-only entangled control
- Marker genes map to known cell types: the disentangled model predicts highest Ccl5/Tcf7l2 expression in their associated cell types (e.g. not erroneously highest in erythroblasts)
- UMAP computed from ILR-transformed topic compositions shows batch intermixing and cell-type separation comparable to or better than the scanpy PCA baseline

## Limitations

- Results must be validated with orthogonal biological evidence to rule out under-correction of technical effects for batch-confounded or perturbation-induced cell states
- CODAL does not remove the need for good experimental design — experimentalists must include appropriate controls (e.g. control cells unaffected by the perturbation in both treatment and control batches)
- Further improvements depend on better characterization of technical sources of bias, indicating an open limitation
- Technical effect predictions are Bernoulli-corrupted with probability 0.05 during training, a regularization assumption on technical-effect variance

## Evidence

- [methods] Hyperparameter tuning setup: "We use a Gaussian Process model with a Matern kernel68 to predict the distribution of scores given hyperparameters."
- [methods] MINE Lipschitz constraint: "We employ spectral normalization59 to enforce 1-Lipschitz continuity of the neural network estimator Tθ"
- [intro] Batch-proxy assumption: "the cells’ batch of origin can be used as a proxy variable to indicate which cells were subjected to a protocol using common reagents and under similar conditions."
- [readme] MIRA/CODAL packaging and training approach: [evidence span withheld — anchor status: external_doc]
- [readme] MIRA dependencies: "MIRA works on top of `Scanpy <https://scanpy.readthedocs.io/en/stable/>`_ and `Anndata <https://anndata.readthedocs.io/en/latest/>`_"
