---
name: elbo-versus-codal-objective-comparison
description: Use when a simulated or real multi-batch single-cell dataset has cell
  types confounded with batch (e.
when_to_use_negative:
- Single-batch data with no batch-confounded cell states — there is no technical-vs-biological
  dependence for MI regularization to disentangle
- The goal is generic atlas integration with scVI/scANVI/Seurat v3/Harmony rather
  than an explicit head-to-head objective comparison with ground-truth rates
- No simulated ground truth or validated control cells exist, so predicted expression
  rates cannot be checked against a data-generating procedure
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CODAL
  role: Variational autoencoder objective (COvariate Disentangling Augmented Loss)
    that adds MINE mutual-information regularization to disentangle technical from
    biological effects
  repo: https://mira-multiome.readthedocs.io
- name: MIRA
  role: Base variational topic modeling architecture that CODAL extends; provides
    the topic model trained under both objectives
  repo: github.com/cistrome/MIRA
- name: PyTorch
  role: Implementation framework for the VAE topic models, MINE estimator with spectral
    normalization, and cyclical stochastic minibatch gradient ascent
- name: Frankencell
  role: Synthetic benchmark generation of differentiation trajectories by mixing reads
    from distinct cell populations, used to build ground-truth simulation scenarios
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:8d39e33ba1fbec6965fa3ce186588452d138f6f3d7109b4dc8fc8bf3a202fe09
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CODAL
  - Frankencell
  - MIRA
  - PyTorch
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# elbo-versus-codal-objective-comparison

## Summary

Trains two topic models on the same batch-confounded single-cell counts — one with only the ELBO (MI regularization weight zero) and one with the CODAL objective (MINE mutual-information penalty between predicted expression and technical effects) — to quantify how disentangling recovers data-generating expression rates.

## When to use

Use when a simulated or real multi-batch single-cell dataset has cell types confounded with batch (e.g., a gene upregulated only in the treatment batch after a bifurcation) and you need to show whether MI-regularized disentanglement outperforms a vanilla ELBO objective at recovering true expression rates.

## When NOT to use

- Single-batch data with no batch-confounded cell states — there is no technical-vs-biological dependence for MI regularization to disentangle
- The goal is generic atlas integration with scVI/scANVI/Seurat v3/Harmony rather than an explicit head-to-head objective comparison with ground-truth rates
- No simulated ground truth or validated control cells exist, so predicted expression rates cannot be checked against a data-generating procedure

## Inputs

- Simulated bifurcating differentiation count matrix with two batches and a treatment-batch-specific upregulated gene (gene A)
- One-hot batch indicator covariate vector per cell
- Ground-truth simulated expression rates for gene A in both batches

## Outputs

- Fitted ELBO-only topic model and fitted CODAL topic model on the same counts
- Mutual information estimates between expression-rate and technical-effect predictions for each model
- Per-model predicted gene A expression rates versus ground-truth rates across both batches
- Fig. 1e reproduction figure (e.g., figure_1e_elbo_vs_codal.png)

## How to apply

Simulate a bifurcating differentiation trajectory sampled into two batches, injecting gene A as upregulated after the bifurcation point only in the treatment batch, and attach one-hot batch indicator covariates. Train the control topic model using only the ELBO lower bound on the marginal likelihood with the mutual-information regularization weight set to zero. Train the second model with the CODAL objective, regularizing dependence between predicted expression and technical effects via the MINE lower bound, enforcing 1-Lipschitz continuity of the estimator Tθ with spectral normalization, using cyclical stochastic minibatch gradient ascent (minibatch size 128) with cyclic annealing of the objective term weights. Estimate the mutual information between expression-rate and technical-effect predictions for both fitted models, then compare each model's predicted gene A rates against the simulated ground truth across both batches. Render the Fig. 1e-style comparison of ELBO batch-confounded predictions versus CODAL disentangled predictions alongside their MI estimates (e.g., figure_1e_elbo_vs_codal.png).

## Related tools

- **CODAL** (Variational autoencoder objective (COvariate Disentangling Augmented Loss) that adds MINE mutual-information regularization to disentangle technical from biological effects) — https://mira-multiome.readthedocs.io
- **MIRA** (Base variational topic modeling architecture that CODAL extends; provides the topic model trained under both objectives) — github.com/cistrome/MIRA
- **PyTorch** (Implementation framework for the VAE topic models, MINE estimator with spectral normalization, and cyclical stochastic minibatch gradient ascent)
- **Frankencell** (Synthetic benchmark generation of differentiation trajectories by mixing reads from distinct cell populations, used to build ground-truth simulation scenarios)

## Examples

```
conda create --name mira-env -c conda-forge -c pytorch -c bioconda scanpy jupyter leidenalg && pip3 install mira-multiome
```

## Evaluation signals

- CODAL's mutual-information estimate between predicted expression rates and technical effects is markedly lower than the ELBO model's
- CODAL-predicted gene A rates match the simulated data-generating rates in both batches, while ELBO predictions are poor and batch-confounded
- MINE estimator Tθ remains 1-Lipschitz (spectral normalization applied) so MI lower bounds stay valid
- Both models are trained on identical counts and batch covariates, differing only in the MI regularization weight (zero for ELBO control)
- Optimization uses cyclical stochastic minibatch gradient ascent (minibatch size 128) with cyclically annealed objective weights for the CODAL run

## Limitations

- New batch-confounded or perturbation-induced states should be validated with orthogonal biological evidence to rule out under-correction of technical effects
- CODAL does not remove the need for good experimental design and appropriate controls (e.g., include cells unaffected by the perturbation in both treatment and control batches)
- Further improvements require better characterization of technical sources of bias in conjunction with the CODAL model

## Evidence

- [abstract] CODAL MI regularization disentanglement: "uses a mutual information regularization technique to explicitly disentangle factors related to technical and biological effects"
- [intro] Batch of origin as technical proxy: "the cells’ batch of origin can be used as a proxy variable to indicate which cells were subjected to a protocol using common reagents"
- [methods] MINE 1-Lipschitz constraint via spectral normalization: "We employ spectral normalization59 to enforce 1-Lipschitz continuity of the neural network estimator Tθ"
- [methods] Cyclical minibatch optimization: "CODAL employs a cyclical stochastic minibatch gradient ascent optimization strategy to learn parameters"
- [readme] CODAL inference in all MIRA topic models: [evidence span withheld — anchor status: external_doc]
