---
name: single-cell-umi-count-modeling
description: Use when raw scRNA-seq UMI count matrices (e.g., PBMC4k 4182 cells ×
  1000 highly variable genes) must be modeled with per-gene negative-binomial dispersion,
  specifically to estimate or refresh the inverse dispersion parameter θ during deep
  generative model training.
when_to_use_negative:
- Input is normalized, log-transformed, or otherwise non-integer expression data —
  the negative-binomial likelihood in scDBM models raw UMI counts only
- Data contain batch effects that need correction — the paper states 'Currently, batch
  effects cannot be learned with scDBMs' and all models were examined without batch
  correction capabilities
- The goal is a quantitatively benchmarked dispersion estimator — the paper describes
  the Fisher scoring module but does not separately benchmark it
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scDBM.jl
  role: Julia implementation of the single-cell deep Boltzmann machine whose training
    loop hosts the regularized Fisher scoring θ-estimator
  repo: github.com/MTreppner/scDBM.jl
- name: BoltzmannMachines.jl
  role: Base Julia package on which scDBM.jl is built, providing DBM training primitives
    (stochastic gradient descent, Gibbs sampling)
  repo: https://github.com/MTreppner/scDBM.jl
- name: Julia 1.5.0
  role: Runtime environment for scDBM.jl and BoltzmannMachines.jl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:4ea15fd3da0d4a12412deebb67dec10bbd43f6daac956333799e0765c7cdf506
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BoltzmannMachines.jl
  - Julia
  - Julia 1.5.0
  - scDBM.jl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-umi-count-modeling

## Summary

Model scRNA-seq UMI counts with a negative-binomial likelihood (variance μ + μ²/θ) inside the scDBM deep Boltzmann machine, estimating the per-gene inverse dispersion parameter θ via a regularized Fisher scoring algorithm with a λ/θ² penalty. This is the paper's count-distribution mechanism that makes DBM training compatible with overdispersed single-cell count data.

## When to use

Use when raw scRNA-seq UMI count matrices (e.g., PBMC4k 4182 cells × 1000 highly variable genes) must be modeled with per-gene negative-binomial dispersion, specifically to estimate or refresh the inverse dispersion parameter θ during deep generative model training.

## When NOT to use

- Input is normalized, log-transformed, or otherwise non-integer expression data — the negative-binomial likelihood in scDBM models raw UMI counts only
- Data contain batch effects that need correction — the paper states 'Currently, batch effects cannot be learned with scDBMs' and all models were examined without batch correction capabilities
- The goal is a quantitatively benchmarked dispersion estimator — the paper describes the Fisher scoring module but does not separately benchmark it

## Inputs

- Preprocessed scRNA-seq UMI count matrix (e.g., PBMC4k: 4182 cells × 1000 highly variable genes)
- Regularization strength λ for the Fisher scoring penalty
- Refresh interval in epochs for θ re-estimation (e.g., five epochs)

## Outputs

- Per-gene inverse dispersion estimates written to theta_estimates.csv (one positive θ entry per gene)
- Trained scDBM parameters (a, b⁽¹⁾, b⁽²⁾, W⁽¹⁾, W⁽²⁾, θ) usable for synthetic count generation

## How to apply

Load a preprocessed UMI count matrix in Julia with scDBM.jl/BoltzmannMachines.jl and formulate the negative-binomial log-likelihood p_NB(v; μ, θ) from the Risso et al. parametrization (variance μ + μ²/θ), deriving the score function V(θ) and Fisher information I(θ). Apply the iterative Fisher scoring update per gene m: θ_{m,k+1} = θ_{m,k} + [V(θ_{m,k}) + λ·2/θ³_{m,k}] / [I(θ_{m,k}) + λ·6/θ⁴_{m,k}], where λ/θ² is added to the log-likelihood as the regularization term so that smaller θ estimates receive proportionally larger shrinkage. During scDBM training, fix θ while learning (a, b⁽¹⁾, b⁽²⁾, W⁽¹⁾, W⁽²⁾) via stochastic gradient descent, and periodically (e.g., every five epochs) re-run Fisher scoring to refresh θ̂_m, plugging updated values back into the model. Enforce the exponential-family constraints a_m = min{a_m, −ε} with ε = 10⁻¹⁰ and w_{m,k} = min{w_{m,k}, 0} throughout training.

## Related tools

- **scDBM.jl** (Julia implementation of the single-cell deep Boltzmann machine whose training loop hosts the regularized Fisher scoring θ-estimator) — github.com/MTreppner/scDBM.jl
- **BoltzmannMachines.jl** (Base Julia package on which scDBM.jl is built, providing DBM training primitives (stochastic gradient descent, Gibbs sampling)) — https://github.com/MTreppner/scDBM.jl
- **Julia 1.5.0** (Runtime environment for scDBM.jl and BoltzmannMachines.jl)

## Examples

```
julia> theta_new = theta .+ (V(theta) .+ lambda .* 2 ./ theta.^3) ./ (Ifisher(theta) .+ lambda .* 6 ./ theta.^4)  # regularized Fisher scoring update per gene
```

## Evaluation signals

- theta_estimates.csv exists with exactly one θ entry per gene and every θ value is positive
- The Fisher scoring iteration converges, and smaller θ estimates receive proportionally larger regularization via the λ/θ² penalty (verifiable by expert review since the paper reports no separate quantitative benchmark)
- Exponential-family constraints hold after training: a_m ≤ −ε with ε = 10⁻¹⁰ and w_{m,k} ≤ 0 for all genes/units
- θ values are refreshed on schedule (e.g., every five epochs) and remain stable across successive Fisher scoring refreshes

## Limitations

- The regularized Fisher scoring module for θ estimation is described by the paper but not separately benchmarked, so its performance must be judged by expert review of convergence and regularization behavior
- Currently, batch effects cannot be learned with scDBMs, and all models were examined without batch correction capabilities
- For sparser 10x Genomics datasets it is more challenging to make an inference from small to larger datasets than for Smart-seq2 data, affecting downstream usefulness of the fitted count model

## Evidence

- [methods] scDBM negative-binomial Fisher scoring: [evidence span withheld — anchor status: not_found]
- [methods] theta update and refresh: [evidence span withheld — anchor status: not_found]
- [methods] parametrization: [evidence span withheld — anchor status: not_found]
- [readme] package basis: "The package is based on the implementation 'BoltzmannMachines.jl'"
- [readme] runtime requirement: "## Main requirements  
Julia: 1.5.0"
