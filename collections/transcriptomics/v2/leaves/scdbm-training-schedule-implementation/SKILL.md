---
name: scdbm-training-schedule-implementation
description: Use when fitting scDBM/BoltzmannMachines models to scRNA-seq count matrices
  and you need per-gene negative-binomial inverse dispersion (θ) estimates refreshed
  periodically during deep-Boltzmann-machine training.
when_to_use_negative:
- The dataset has batch effects that must be modeled or corrected — the paper notes
  'Currently, batch effects cannot be learned with scDBMs' and all models were examined
  without batch-correction capabilities
- You need an independently benchmarked dispersion estimator — the paper describes
  this Fisher scoring mechanism but does not separately benchmark it
- The input is not UMI-style count data matching the negative-binomial (variance μ
  + μ²/θ) assumption, e.g., already-normalized expression values
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scDBM.jl
  role: Julia implementation of the single-cell deep Boltzmann machine whose training
    loop embeds the regularized Fisher scoring θ estimator
  repo: github.com/MTreppner/scDBM.jl
- name: BoltzmannMachines.jl
  role: Base Julia package on which scDBM.jl is built, providing DBM training primitives
    (e.g., stochastic gradient descent, Gibbs sampling)
- name: Julia 1.5.0
  role: Runtime environment required to execute the scDBM.jl training and θ-estimation
    code
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:e8d80077bc9259b8bfbfc64d45d7fa098dfc76e1e6cf3645a30429e6b38ba672
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BoltzmannMachines.jl
  - Julia
  - Julia 1.5.0
  - scDBM.jl
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# scdbm-training-schedule-implementation

## Summary

Reconstructs the regularized Fisher scoring mechanism by which the scDBM (single-cell deep Boltzmann machine) estimates the per-gene inverse dispersion parameter θ of a negative-binomial likelihood for scRNA-seq UMI counts, and integrates it into the model's stochastic-gradient-descent training loop in Julia.

## When to use

Use when fitting scDBM/BoltzmannMachines models to scRNA-seq count matrices and you need per-gene negative-binomial inverse dispersion (θ) estimates refreshed periodically during deep-Boltzmann-machine training.

## When NOT to use

- The dataset has batch effects that must be modeled or corrected — the paper notes 'Currently, batch effects cannot be learned with scDBMs' and all models were examined without batch-correction capabilities
- You need an independently benchmarked dispersion estimator — the paper describes this Fisher scoring mechanism but does not separately benchmark it
- The input is not UMI-style count data matching the negative-binomial (variance μ + μ²/θ) assumption, e.g., already-normalized expression values

## Inputs

- Preprocessed scRNA-seq UMI count matrix (e.g., PBMC4k: 4182 cells × 1000 highly variable genes)
- Regularization hyperparameter λ for the λ/θ² log-likelihood penalty
- Epoch interval (e.g., 5) at which θ estimates are refreshed during training

## Outputs

- theta_estimates.csv with one positive θ entry per gene
- Trained scDBM parameters (a, b⁽¹⁾, b⁽²⁾, W⁽¹⁾, W⁽²⁾, θ) usable for Gibbs-sampling synthetic cells

## How to apply

Load a preprocessed UMI count matrix (e.g., PBMC4k, 4182 cells × 1000 highly variable genes) in Julia with scDBM.jl/BoltzmannMachines.jl. Formulate the negative-binomial log-likelihood p_NB(v; μ, θ) using the Risso et al. parametrization with variance μ + μ²/θ, and derive the score function V(θ) and Fisher information I(θ) per gene. Run the iterative regularized Fisher scoring update θ_{m,k+1} = θ_{m,k} + [V(θ_{m,k}) + λ·2/θ³_{m,k}] / [I(θ_{m,k}) + λ·6/θ⁴_{m,k}], which adds a λ/θ² regularization term to the log-likelihood so smaller θ estimates receive proportionally stronger shrinkage. During scDBM training, hold θ fixed while learning (a, b⁽¹⁾, b⁽²⁾, W⁽¹⁾, W⁽²⁾) via stochastic gradient descent, and re-run Fisher scoring to refresh θ̂_m every predefined number of epochs (e.g., five). Enforce the exponential-family parameter constraints a_m = min{a_m, −ε} with ε = 10⁻¹⁰ and w_{m,k} = min{w_{m,k}, 0} throughout training.

## Related tools

- **scDBM.jl** (Julia implementation of the single-cell deep Boltzmann machine whose training loop embeds the regularized Fisher scoring θ estimator) — github.com/MTreppner/scDBM.jl
- **BoltzmannMachines.jl** (Base Julia package on which scDBM.jl is built, providing DBM training primitives (e.g., stochastic gradient descent, Gibbs sampling))
- **Julia 1.5.0** (Runtime environment required to execute the scDBM.jl training and θ-estimation code)

## Examples

```
for m in 1:size(counts, 2); theta[m] += (V(theta[m]) + lambda*2/theta[m]^3) / (I(theta[m]) + lambda*6/theta[m]^4); end; writedlm("theta_estimates.csv", theta)
```

## Evaluation signals

- Script runs end-to-end and theta_estimates.csv exists with exactly one θ entry per gene (1000 entries for the 1000-HVG PBMC4k input)
- All estimated θ values are strictly positive
- Smaller θ estimates receive proportionally larger regularization via the λ/θ² term, as the paper describes
- Fisher scoring refresh (re-run every ~5 epochs) converges — the paper reports no separate quantitative benchmark, so convergence is verified by expert review of the update sequence
- Exponential-family constraints hold after training: a_m ≤ −ε with ε = 10⁻¹⁰ and all w_{m,k} ≤ 0

## Limitations

- The paper describes the regularized Fisher scoring mechanism but does not separately benchmark it quantitatively
- scDBMs currently cannot learn batch effects, so θ estimation and training assume unbatched or batch-agnostic data
- The approach relies on the unverified assumption that synthetic cells (generated downstream using the trained parameters) map to the correct clusters

## Evidence

- [other] The scDBM adaptation models scRNA-seq counts with a negative binomial distribution and estimates the inverse dispersion parameter via a regularized Fisher scoring algorithm: [evidence span withheld — anchor status: not_found]
- [other] Workflow: initialize parameters, fix θ while learning, refresh every 5 epochs: [evidence span withheld — anchor status: not_found]
- [readme] scDBM.jl is the implementation of the paper and is based on BoltzmannMachines.jl: "This is the implementation for "Synthetic Single-Cell RNA-Sequencing Data from Small Pilot Studies using Deep Generative Models"."
- [discussion] scDBMs cannot currently learn batch effects: "Currently, batch effects cannot be learned with scDBMs."
