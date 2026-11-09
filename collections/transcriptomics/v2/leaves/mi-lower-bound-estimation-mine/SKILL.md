---
name: mi-lower-bound-estimation-mine
description: Use when training a topic/VAE model on multi-batch single-cell counts
  where cell states may be batch-confounded, and you must quantify or penalize the
  dependence between expression-rate predictions and batch covariates.
when_to_use_negative:
- Data has no batch structure or technical covariates — the MI regularization target
  is absent, so the ELBO-only objective suffices
- Cell states are not batch-confounded (standard batch-correction methods like Seurat
  v3, FastMNN, Scanorama, or Harmony are appropriate)
- Input is continuous/normalized expression rather than raw counts — the count-based
  topic model assumption is violated
tools:
- name: CODAL
  role: Variational topic-model objective in which the MINE MI lower bound regularizes
    dependence between predicted expression and technical effects
  repo: https://mira-multiome.readthedocs.io
- name: MIRA
  role: Python package implementing the topic models that all use the CODAL algorithm
    for inference
  repo: https://github.com/cistrome/MIRA
- name: PyTorch
  role: Framework for the neural-network MI estimator Tθ, spectral normalization,
    and minibatch gradient-ascent training
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:fba1c8241f7613dec35a1c74222683a374d4739ee96896c75d59e0f91bc2f794
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CODAL
  - Frankencell
  - MIRA
  - PyTorch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# MI lower bound estimation (MINE)

## Summary

Estimate the mutual information between a topic model's predicted expression and technical-effect predictions using the MINE neural-network lower bound, so the estimate can be used as a regularizer (CODAL objective) that disentangles biological from technical effects in batched single-cell count data.

## When to use

Use when training a topic/VAE model on multi-batch single-cell counts where cell states may be batch-confounded, and you must quantify or penalize the dependence between expression-rate predictions and batch covariates.

## When NOT to use

- Data has no batch structure or technical covariates — the MI regularization target is absent, so the ELBO-only objective suffices
- Cell states are not batch-confounded (standard batch-correction methods like Seurat v3, FastMNN, Scanorama, or Harmony are appropriate)
- Input is continuous/normalized expression rather than raw counts — the count-based topic model assumption is violated

## Inputs

- Raw single-cell count matrix (scRNA-seq, e.g., AnnData) with one-hot batch indicator covariates
- Topic-model latent predictions: expression-rate estimates and technical-effect predictions
- Hyperparameter configuration for cyclical minibatch training (minibatch size 128, cyclic weight annealing)

## Outputs

- Mutual information (MINE lower bound) estimate between predicted expression and technical effects
- Regularized CODAL model with disentangled expression-rate predictions per batch
- Comparison of predicted vs ground-truth rates (e.g., figure_1e_elbo_vs_codal.png)

## How to apply

In CODAL, the mutual information between predicted expression rates and one-hot batch indicator covariates is estimated with the MINE lower bound: a neural network estimator Tθ is trained jointly with the topic model while its 1-Lipschitz continuity is enforced via spectral normalization. The estimated MI term enters the CODAL loss alongside the ELBO on the marginal likelihood (the ELBO-only baseline sets this regularization weight to zero), with term weights annealed during cyclical stochastic minibatch gradient ascent (minibatch size 128). Technical-effect predictions may be Bernoulli corrupted with probability 0.05 as regularization. After fitting, compare the estimated MI between models (e.g., ELBO-only vs CODAL) and check that low MI correlates with expression estimates matching simulated ground-truth rates, as in the bifurcating-system comparison of Fig. 1e.

## Related tools

- **CODAL** (Variational topic-model objective in which the MINE MI lower bound regularizes dependence between predicted expression and technical effects) — https://mira-multiome.readthedocs.io
- **MIRA** (Python package implementing the topic models that all use the CODAL algorithm for inference) — https://github.com/cistrome/MIRA
- **PyTorch** (Framework for the neural-network MI estimator Tθ, spectral normalization, and minibatch gradient-ascent training)

## Examples

```
pip3 install mira-multiome
```

## Evaluation signals

- The MI estimator Tθ must satisfy 1-Lipschitz continuity, verifiable via spectral normalization of its layers
- Estimated MI between expression-rate and technical-effect predictions should be substantially lower for the CODAL-regularized model than the ELBO-only model
- In the simulated bifurcating system, CODAL's gene-A expression rates should match the data-generating ground-truth rates in both batches, while the ELBO model's estimates are entangled with batch
- Training proceeds via cyclical stochastic minibatch gradient ascent with minibatch size 128 and cyclically annealed objective-term weights

## Limitations

- Results require orthogonal biological validation to rule out under-correction of technical effects
- CODAL does not remove the need for good experimental design and appropriate controls (e.g., include cells unaffected by the perturbation in both treatment and control batches)
- Further improvements require better characterization of technical sources of bias in conjunction with the CODAL model

## Evidence

- [methods] MINE spectral normalization constraint: "We employ spectral normalization59 to enforce 1-Lipschitz continuity of the neural network estimator Tθ"
- [abstract] MI regularization disentangles technical and biological factors: "uses a mutual information regularization technique to explicitly disentangle factors related to technical and biological"
- [readme] CODAL inference in MIRA topic models: [evidence span withheld — anchor status: external_doc]
