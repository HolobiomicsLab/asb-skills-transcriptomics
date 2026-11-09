---
name: exponential-family-harmonium-parametrization
description: Use when fitting a deep Boltzmann machine (scDBM) to raw scRNA-seq UMI
  count matrices (e.g., PBMC4k, 4182 cells × 1000 highly variable genes) and per-gene
  negative-binomial inverse dispersion parameters must be estimated during training.
when_to_use_negative:
- Input is already normalized/log-transformed or batch-corrected expression rather
  than raw UMI counts — the negative-binomial likelihood and Fisher scoring assume
  integer counts
- 'The goal is batch-effect correction: the paper states explicitly that ''Currently,
  batch effects cannot be learned with scDBMs'' and all models were examined without
  batch-correction capabilities'
- A separate quantitative benchmark of the dispersion estimator is required — the
  paper describes the mechanism but does not separately benchmark it
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scDBM.jl
  role: Julia implementation of the single-cell deep Boltzmann machine whose training
    loop embeds the regularized Fisher-scoring θ estimator
  repo: https://github.com/MTreppner/scDBM.jl
- name: BoltzmannMachines.jl
  role: Base Julia package on which scDBM.jl is built, providing the Boltzmann-machine
    training machinery (stochastic gradient descent, Gibbs sampling)
- name: Julia 1.5.0
  role: Runtime environment required by scDBM.jl and the scDBM-paper analysis notebooks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:76d8631b531e1d85b3d410d8f1164fb6a86fc5387dd0390749df75131fb021d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MTreppner/scDBM.jl
  tools:
  - BoltzmannMachines.jl
  - Julia
  - Julia 1.5.0
  - scDBM.jl
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# exponential-family-harmonium-parametrization

## Summary

Adapts deep Boltzmann machines (harmonium-style exponential-family parametrization) to scRNA-seq UMI counts by modeling each gene with a negative-binomial likelihood and estimating the per-gene inverse dispersion θ via a regularized Fisher scoring algorithm, integrated into the scDBM training loop as described by Treppner et al. (2021).

## When to use

Use when fitting a deep Boltzmann machine (scDBM) to raw scRNA-seq UMI count matrices (e.g., PBMC4k, 4182 cells × 1000 highly variable genes) and per-gene negative-binomial inverse dispersion parameters must be estimated during training.

## When NOT to use

- Input is already normalized/log-transformed or batch-corrected expression rather than raw UMI counts — the negative-binomial likelihood and Fisher scoring assume integer counts
- The goal is batch-effect correction: the paper states explicitly that 'Currently, batch effects cannot be learned with scDBMs' and all models were examined without batch-correction capabilities
- A separate quantitative benchmark of the dispersion estimator is required — the paper describes the mechanism but does not separately benchmark it

## Inputs

- Preprocessed scRNA-seq UMI count matrix (e.g., PBMC4k: 4182 cells × 1000 highly variable genes)
- Regularization hyperparameter λ for the θ-squared penalty
- Epoch interval for periodic θ refresh (e.g., every five epochs)

## Outputs

- Per-gene inverse dispersion estimates written to theta_estimates.csv (one positive θ entry per gene)
- Trained scDBM with negative-binomial conditional distributions usable for synthetic data generation via Gibbs sampling

## How to apply

Formulate the negative-binomial log-likelihood p_NB(v; μ, θ) in the Risso et al. parametrization with variance μ + μ²/θ, and derive the score function V(θ) and Fisher information I(θ). For each gene m, iterate the regularized Fisher scoring update θ_{m,k+1} = θ_{m,k} + [V(θ_{m,k}) + λ·2/θ³_{m,k}] / [I(θ_{m,k}) + λ·6/θ⁴_{m,k}], adding a λ/θ² regularization term to the log-likelihood so that smaller θ estimates receive proportionally larger regularization. Integrate the estimator into scDBM training: initialize all parameters, fix θ while learning (a, b⁽¹⁾, b⁽²⁾, W⁽¹⁾, W⁽²⁾) via stochastic gradient descent, and re-run Fisher scoring to refresh θ̂_m every predefined number of epochs (e.g., five), plugging new estimates back into the model. During training, enforce the exponential-family constraints a_m = min{a_m, −ε} with ε = 10⁻¹⁰ and w_{m,k} = min{w_{m,k}, 0}, reflecting the negative conditioning of the count parametrization.

## Related tools

- **scDBM.jl** (Julia implementation of the single-cell deep Boltzmann machine whose training loop embeds the regularized Fisher-scoring θ estimator) — https://github.com/MTreppner/scDBM.jl
- **BoltzmannMachines.jl** (Base Julia package on which scDBM.jl is built, providing the Boltzmann-machine training machinery (stochastic gradient descent, Gibbs sampling))
- **Julia 1.5.0** (Runtime environment required by scDBM.jl and the scDBM-paper analysis notebooks)

## Examples

```
julia -e 'using Pkg; Pkg.add(path="https://github.com/MTreppner/scDBM.jl"); using scDBM'  # train scDBM on PBMC4k UMI counts; Fisher scoring refreshes θ̂_m every 5 epochs, writes theta_estimates.csv
```

## Evaluation signals

- theta_estimates.csv exists after a full run and contains exactly one θ entry per gene (1000 entries for a 1000-HVG PBMC4k run)
- All estimated θ values are strictly positive
- Smaller θ estimates receive proportionally larger λ/θ² regularization, and the Fisher-scoring updates converge for each gene
- The exponential-family constraints hold throughout training: a_m = min{a_m, −ε} with ε = 10⁻¹⁰ and w_{m,k} = min{w_{m,k}, 0}
- The trained scDBM subsequently generates synthetic data via Gibbs sampling that can be evaluated with the paper's DBI, ARI, cluster-proportion, and per-gene-distribution metrics

## Limitations

- The paper describes the regularized Fisher scoring mechanism but does not provide a separate quantitative benchmark for this module
- Batch effects currently cannot be learned with scDBMs, so this parametrization is unsuitable for multi-batch designs
- For higher-sparsity 10x Genomics datasets, inference from small pilot samples to larger synthetic datasets is more challenging than for Smart-seq2 data
- Evaluation of the resulting generative model is difficult because examination based on multiple quality measures (DBI, ARI, proportions, per-gene distributions) can be ambiguous

## Evidence

- [readme] scDBM proposes negative-binomial parametrization: "Second, we propose single-cell deep Boltzmann machines (scDBM)."
- [readme] scDBM.jl builds on BoltzmannMachines.jl: "The package is based on the implementation 'BoltzmannMachines.jl'"
- [full_text] Regularized Fisher scoring mechanism for θ: [evidence span withheld — anchor status: not_found]
- [full_text] Exponential-family parameter constraints during training: [evidence span withheld — anchor status: not_found]
- [results] Gibbs sampling used to generate from the trained scDBM: "we sampled from the scDBM using Gibbs sampling and from scVI using the prior and posterior distribution"
