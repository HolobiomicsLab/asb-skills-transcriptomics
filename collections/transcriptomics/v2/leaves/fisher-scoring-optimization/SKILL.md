---
name: fisher-scoring-optimization
description: Use when fitting a scDBM or negative-binomial count model to a scRNA-seq
  UMI count matrix and the per-gene inverse dispersion parameter θ needs to be estimated
  or periodically refreshed during training.
when_to_use_negative:
- Input is not raw UMI counts (e.g., already log-normalized or Smart-seq2-style non-count
  data), since the negative-binomial variance μ + μ²/θ parametrization assumes count
  data
- The goal is batch correction — scDBMs currently cannot learn batch effects, and
  batch correction was outside the paper's scope
- 'A benchmarked/validated dispersion estimator is required: the paper describes this
  Fisher scoring module but reports no separate quantitative benchmark for it'
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Julia
  role: Implementation language for the scDBM Fisher scoring estimator and training
    loop
- name: BoltzmannMachines.jl
  role: Base Julia package on which scDBM.jl and the Fisher scoring machinery are
    built
- name: scDBM.jl
  role: Reference implementation of the scDBM with negative-binomial dispersion estimation
  repo: github.com/MTreppner/scDBM.jl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:4e29848e7f98bf437fd6e0c26abca9b8131dd166dc1afa3c90b5d36fa32557d7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BoltzmannMachines.jl
  - Julia
  - scDBM.jl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fisher-scoring-optimization

## Summary

Estimates the per-gene inverse dispersion parameter θ of a negative-binomial likelihood in the scDBM generative model via a regularized Fisher scoring update, alternated with stochastic-gradient training. This is the mechanism by which the scDBM adapts deep Boltzmann machines to model scRNA-seq UMI counts.

## When to use

Use when fitting a scDBM or negative-binomial count model to a scRNA-seq UMI count matrix and the per-gene inverse dispersion parameter θ needs to be estimated or periodically refreshed during training.

## When NOT to use

- Input is not raw UMI counts (e.g., already log-normalized or Smart-seq2-style non-count data), since the negative-binomial variance μ + μ²/θ parametrization assumes count data
- The goal is batch correction — scDBMs currently cannot learn batch effects, and batch correction was outside the paper's scope
- A benchmarked/validated dispersion estimator is required: the paper describes this Fisher scoring module but reports no separate quantitative benchmark for it

## Inputs

- Preprocessed scRNA-seq UMI count matrix (e.g., PBMC4k: 4182 cells × 1000 highly variable genes)
- Initial scDBM parameters (a, b⁽¹⁾, b⁽²⁾, W⁽¹⁾, W⁽²⁾) and a regularization strength λ

## Outputs

- theta_estimates.csv with one positive inverse-dispersion θ entry per gene m
- Refreshed θ̂_m values plugged back into the scDBM training loop every ~5 epochs

## How to apply

Formulate the negative-binomial log-likelihood p_NB(v; μ, θ) under the Risso et al. parametrization with variance μ + μ²/θ, and derive the score function V(θ) and Fisher information I(θ). For each gene m, iterate the regularized Fisher scoring update θ_{m,k+1} = θ_{m,k} + [V(θ_{m,k}) + λ·2/θ³_{m,k}] / [I(θ_{m,k}) + λ·6/θ⁴_{m,k}], where the λ/θ² term regularizes the log-likelihood (smaller θ estimates receive proportionally larger regularization). During scDBM training, fix θ while learning (a, b⁽¹⁾, b⁽²⁾, W⁽¹⁾, W⁽²⁾) by stochastic gradient descent, apply the exponential-family constraints a_m = min{a_m, −ε} with ε = 10⁻¹⁰ and w_{m,k} = min{w_{m,k}, 0}, and re-run Fisher scoring to refresh θ̂_m every predefined number of epochs (e.g., five), plugging the new estimates back into the scDBM. Converged per-gene estimates are written to theta_estimates.csv. The paper describes but does not separately benchmark this module.

## Related tools

- **Julia** (Implementation language for the scDBM Fisher scoring estimator and training loop)
- **BoltzmannMachines.jl** (Base Julia package on which scDBM.jl and the Fisher scoring machinery are built)
- **scDBM.jl** (Reference implementation of the scDBM with negative-binomial dispersion estimation) — github.com/MTreppner/scDBM.jl

## Examples

```
julia> import Pluto; Pluto.run()  # then open scDBM_notebook.jl from the scDBM-paper repo
```

## Evaluation signals

- theta_estimates.csv exists and contains exactly one θ entry per gene (e.g., 1000 rows for PBMC4k's 1000 HVGs)
- All θ values in theta_estimates.csv are positive
- The iterative update converges: successive θ_{m,k} changes shrink to a stable value for each gene
- Regularization behaves as described: smaller θ estimates receive proportionally larger regularization (λ/θ² term dominates as θ → 0)
- θ̂_m values are actually refreshed in the scDBM at the predefined interval (e.g., every five epochs) and remain consistent with the exponential-family constraints a_m = min{a_m, −ε} (ε = 10⁻¹⁰) and w_{m,k} ≤ 0

## Limitations

- The paper describes the Fisher scoring mechanism but does not separately benchmark it; correctness must be confirmed by expert review of convergence and regularization behavior
- Because the regularizer λ/θ² grows as θ shrinks, small inverse-dispersion estimates (highly overdispersed genes) are strongly pulled by regularization, which may bias estimates for very overdispersed genes
- scDBMs currently cannot learn batch effects, so dispersion estimates are not batch-corrected

## Evidence

- [full_text] scDBM adaptation models counts with negative binomial and estimates inverse dispersion via regularized Fisher scoring: [evidence span withheld — anchor status: not_found]
- [full_text] Periodic refresh of theta during scDBM training: [evidence span withheld — anchor status: not_found]
- [abstract] Second, we propose single-cell deep Boltzmann machines (scDBM).: "Second, we propose single-cell deep Boltzmann machines (scDBM)."
- [readme] README: package is the implementation of the scDBM paper and builds on BoltzmannMachines.jl: "This is the implementation for **"Synthetic Single-Cell RNA-Sequencing Data from Small Pilot Studies using Deep Generative Models"**."
