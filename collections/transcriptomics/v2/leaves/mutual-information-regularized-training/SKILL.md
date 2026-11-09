---
name: mutual-information-regularized-training
description: Use when training a topic model/VAE on multi-batch scRNA-seq (or multiome)
  data where cell states are batch-confounded and ELBO-only training entangles predicted
  expression rates with technical effect estimates.
when_to_use_negative:
- Data come from a single batch with no technical covariates to encode — the disentanglement
  objective has nothing to regularize against
- The goal is de novo ChIP-seq peak calling (MACS is an unrelated tool surfaced in
  the same repositories) rather than multi-batch expression disentanglement
- Cell states are not batch-confounded and a standard integration method (scVI, Harmony,
  Seurat v3) is known to suffice — the authors position CODAL specifically for batch-confounded
  comparative atlases
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CODAL/MIRA
  role: Topic-model package whose inference implements the mutual-information-regularized
    objective; models are created with mira.topics.make_model
  repo: https://github.com/cistrome/MIRA
- name: PyTorch
  role: Backend for the VAE encoder, technical effect network, and spectrally-normalized
    MINE estimator Tθ; GPU support accelerates training
- name: Pyro
  role: Probabilistic programming layer for the variational Dirichlet topic model
    inference
- name: Optuna
  role: Hyperparameter optimization with Gaussian Process (Matern 5/2 kernel) and
    rung-based trial pruning during cyclical training
- name: scanpy
  role: Preprocessing of count matrices (filtering, normalize_total target_sum=10000,
    log+1, HVG selection) before model training
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:97887826c0b55eb92475d4dab3a5ecedb79cd080211d07474a37ead7a35b219b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL/MIRA
  - Frankencell
  - MIRA
  - Optuna
  - PyTorch
  - Pyro
  - scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# mutual-information-regularized-training

## Summary

Train a CODAL topic-model VAE using an objective that augments the standard ELBO with a differentiable MINE mutual-information regularizer, explicitly disentangling technical (batch) effects from biological expression-rate effects in multi-batch single-cell data.

## When to use

Use when training a topic model/VAE on multi-batch scRNA-seq (or multiome) data where cell states are batch-confounded and ELBO-only training entangles predicted expression rates with technical effect estimates.

## When NOT to use

- Data come from a single batch with no technical covariates to encode — the disentanglement objective has nothing to regularize against
- The goal is de novo ChIP-seq peak calling (MACS is an unrelated tool surfaced in the same repositories) rather than multi-batch expression disentanglement
- Cell states are not batch-confounded and a standard integration method (scVI, Harmony, Seurat v3) is known to suffice — the authors position CODAL specifically for batch-confounded comparative atlases

## Inputs

- Preprocessed scRNA-seq count matrix (e.g., NEURIPS bone marrow, 3500 highly variable genes in .h5ad/AnnData)
- One-hot encoded technical covariates C (e.g., sequencing site, donor)
- Model configuration: total pseudocount I=50, Ntopics (e.g., 22), dropout range [0.05, 0.1], minibatch size 128, epochs 24
- Optional ground-truth simulated expression rates (Frankencell bifurcating system) for ELBO-vs-CODAL comparison

## Outputs

- Trained CODAL expression topic model with converged CODAL objective score evaluated on held-out cells
- Posterior topic compositions Z̄ (Ncells × 22)
- β topic-feature weight matrix (22 × 3500 genes)
- Technical effect vectors t = hφ(Z, C) (Ncells × 3500 genes)
- Mutual information estimates between predicted expression rates and technical effects (model vs. ELBO-only control)

## How to apply

Preprocess raw counts to deviance residuals and encode batch variables (e.g., NEURIPS sequencing site and donor) as one-hot technical covariates C, then instantiate the MIRA-based Dirichlet variational encoder (total pseudocount I=50, Ntopics=22, dropout in [0.05, 0.1]). Train with cyclical stochastic minibatch gradient ascent (minibatch size 128, 24 epochs, AdamW with one-cycle LR/momentum policy annealed between 0.85 and 0.95), cyclically annealing the weights of the KL term and the mutual-information regularization term (ε1, ε2). Each optimization step, update the MINE estimator Tθ — a spectrally-normalized 1-Lipschitz network with a 64-node hidden layer (Adam lr=1e-4) — and re-apply spectral normalization; corrupt technical effect predictions with Bernoulli noise (p=0.05) during training. If the training data are simulated with known ground truth (e.g., the bifurcating gene-A system via Frankencell), also train an entangled ELBO-only control with the mutual-information weight set to zero to demonstrate the contrast. Tune the number of topics (15–40 range) with Bayesian hyperparameter optimization using a Gaussian Process with a Matern 5/2 kernel and Optuna-style rung pruning after the 8th and 16th epochs (bottom 50th percentile).

## Related tools

- **CODAL/MIRA** (Topic-model package whose inference implements the mutual-information-regularized objective; models are created with mira.topics.make_model) — https://github.com/cistrome/MIRA
- **PyTorch** (Backend for the VAE encoder, technical effect network, and spectrally-normalized MINE estimator Tθ; GPU support accelerates training)
- **Pyro** (Probabilistic programming layer for the variational Dirichlet topic model inference)
- **Optuna** (Hyperparameter optimization with Gaussian Process (Matern 5/2 kernel) and rung-based trial pruning during cyclical training)
- **scanpy** (Preprocessing of count matrices (filtering, normalize_total target_sum=10000, log+1, HVG selection) before model training)

## Examples

```
model = mira.topics.make_model(adata, covariates=['site','donor'], Ntopics=22); model.train(minibatch_size=128, epochs=24)
```

## Evaluation signals

- Trained model reaches a converged CODAL objective score when evaluated on held-out cells
- Mutual information between predicted expression rates and technical effect predictions is low/uncorrelated in the disentangled model, while the ELBO-only control (regularization weight set to zero) shows covariation
- Serialized arrays have expected shapes: Ncells × 22 topics, 22 × 3500 genes, Ncells × 3500 genes
- Marker genes (Ccl5, Tcf7l2) are predicted highest in their known associated cell types rather than in confounded populations (e.g., Ccl5 erroneously highest in erythroblasts under ELBO)
- On simulated ground truth (gene A in the bifurcating system), CODAL-predicted expression rates match the data-generating rates across both batches, unlike ELBO estimates

## Limitations

- Technical effect variance is proxied by cells' batch of origin, assuming hidden batch-specific factors are captured by this proxy variable
- New batch-confounded or perturbation-induced cell states must be validated with orthogonal biological evidence to rule out under-correction of technical effects
- CODAL does not remove the need for good experimental design and appropriate controls; a control cell group unaffected by the perturbation should be included in both batches
- Further improvement requires better characterization of technical sources of bias, an acknowledged open limitation

## Evidence

- [abstract] Here we propose CODAL, a variational autoencoder-based statistical model: "Here we propose CODAL, a variational autoencoder-based statistical model which uses a mutual information regularization technique to explicitly disentangle factors related to technical and biological"
- [methods] Cyclical training scheme: "CODAL employs a cyclical stochastic minibatch gradient ascent optimization strategy to learn parameters"
- [methods] MINE Lipschitz constraint: "We employ spectral normalization59 to enforce 1-Lipschitz continuity of the neural network estimator Tθ"
- [methods] Technical effect corruption: "the technical effect predictions are Bernoulli corrupted with probability 0.05"
- [methods] Rung pruning: [evidence span withheld — anchor status: not_found]
- [readme] CODAL as MIRA inference: [evidence span withheld — anchor status: external_doc]
