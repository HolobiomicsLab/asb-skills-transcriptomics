---
name: negative-binomial-dispersion-estimation
description: Use when training scDBM or similar NB-based generative models on scRNA-seq
  UMI count matrices (e.g., PBMC4k 4182 cells × 1000 HVGs) and per-gene inverse dispersion
  (θ) estimates are needed for the negative-binomial likelihood.
when_to_use_negative:
- Input is normalized, log-transformed, or otherwise non-count data — the NB likelihood
  and Fisher information are derived for raw UMI counts
- Batch correction is required — the paper states batch effects cannot currently be
  learned with scDBMs and models were examined without batch correction capabilities
- A separately validated or benchmarked dispersion estimator is needed — the paper
  describes this mechanism but does not separately benchmark it
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Julia
  role: Implementation language for the Fisher scoring estimator and scDBM training
    loop (version 1.5.0)
  repo: https://github.com/MTreppner/scDBM.jl
- name: scDBM.jl
  role: Provides the scDBM model and training loop into which the regularized Fisher
    scoring θ-estimator is integrated
  repo: github.com/MTreppner/scDBM.jl
- name: BoltzmannMachines.jl
  role: Base package on which the scDBM implementation is built
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:0a541908e29169cb71cc373e418af481809d1ef7c3a8cde13deb7608921d00bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MTreppner/scDBM.jl
  tools:
  - BoltzmannMachines.jl
  - Julia
  - scDBM.jl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# negative-binomial-dispersion-estimation

## Summary

Estimates the per-gene inverse dispersion parameter (θ) of a negative-binomial likelihood fitted to scRNA-seq UMI counts using a regularized Fisher scoring algorithm inside the scDBM training loop, as proposed in the scDBM paper (Treppner et al. 2021).

## When to use

Use when training scDBM or similar NB-based generative models on scRNA-seq UMI count matrices (e.g., PBMC4k 4182 cells × 1000 HVGs) and per-gene inverse dispersion (θ) estimates are needed for the negative-binomial likelihood.

## When NOT to use

- Input is normalized, log-transformed, or otherwise non-count data — the NB likelihood and Fisher information are derived for raw UMI counts
- Batch correction is required — the paper states batch effects cannot currently be learned with scDBMs and models were examined without batch correction capabilities
- A separately validated or benchmarked dispersion estimator is needed — the paper describes this mechanism but does not separately benchmark it

## Inputs

- Preprocessed scRNA-seq UMI count matrix (e.g., PBMC4k: 4182 cells × 1000 highly variable genes)
- Regularization strength λ for the λ/θ² penalty term
- Epoch interval for periodic θ refresh (e.g., every five epochs)

## Outputs

- theta_estimates.csv with one positive inverse dispersion value θ per gene
- A trained scDBM whose negative-binomial likelihood uses the refreshed per-gene θ̂_m

## How to apply

Formulate the negative-binomial log-likelihood p_NB(v; μ, θ) using the Risso et al. parametrization with variance μ + μ²/θ, and derive its score function V(θ) and Fisher information I(θ). For each gene m, iteratively apply the regularized Fisher scoring update θ_{m,k+1} = θ_{m,k} + [V(θ_{m,k}) + λ·2/θ³_{m,k}] / [I(θ_{m,k}) + λ·6/θ⁴_{m,k}], where the λ/θ² term regularizes the log-likelihood so that smaller θ estimates receive proportionally larger regularization. Integrate this estimator into the scDBM training loop: initialize all parameters, fix θ while learning (a, b⁽¹⁾, b⁽²⁾, W⁽¹⁾, W⁽²⁾) via stochastic gradient descent, and re-run Fisher scoring to refresh θ̂_m every predefined number of epochs (e.g., five), plugging the new estimates back into the scDBM. Enforce the exponential-family constraints a_m = min{a_m, −ε} with ε = 10⁻¹⁰ and w_{m,k} = min{w_{m,k}, 0} during training. Write the converged per-gene θ estimates to theta_estimates.csv.

## Related tools

- **Julia** (Implementation language for the Fisher scoring estimator and scDBM training loop (version 1.5.0)) — https://github.com/MTreppner/scDBM.jl
- **scDBM.jl** (Provides the scDBM model and training loop into which the regularized Fisher scoring θ-estimator is integrated) — github.com/MTreppner/scDBM.jl
- **BoltzmannMachines.jl** (Base package on which the scDBM implementation is built)

## Examples

```
julia> import Pluto; Pluto.run()
```

## Evaluation signals

- The script runs end-to-end and theta_estimates.csv exists with exactly one θ entry per gene
- All estimated θ values in theta_estimates.csv are strictly positive
- The Fisher scoring update converges for each gene, and smaller θ estimates receive proportionally larger regularization from the λ/θ² term
- θ values are periodically refreshed (e.g., every five epochs) and plugged back into the scDBM during training
- Exponential-family constraints hold after training: a_m ≤ −10⁻¹⁰ and w_{m,k} ≤ 0

## Limitations

- The paper describes but does not separately benchmark this θ-estimation module; validation relies on convergence behavior and expert review
- Currently, batch effects cannot be learned with scDBMs, so θ estimates will absorb or ignore batch structure without any correction
- Estimation quality depends on the pilot dataset being representative; inference from small to larger datasets is more challenging for sparser 10x Genomics data than Smart-seq2

## Evidence

- [other] The scDBM adaptation models scRNA-seq counts with a negative binomial distribution and estimates the inverse dispersion parameter via a regularized Fisher scoring algorithm: [evidence span withheld — anchor status: not_found]
- [other] Formulate the negative-binomial log-likelihood p_NB(v; μ, θ) and derive its score function V(θ) and Fisher information I(θ) from the Risso et al. parametrization with variance μ + μ²/θ.: [evidence span withheld — anchor status: not_found]
- [readme] This is the implementation for "Synthetic Single-Cell RNA-Sequencing Data from Small Pilot Studies using Deep Generative Models".: "This is the implementation for "Synthetic Single-Cell RNA-Sequencing Data from Small Pilot Studies using Deep Generative Models"."
- [readme] The package is based on the implementation 'BoltzmannMachines.jl': "The package is based on the implementation 'BoltzmannMachines.jl'"
- [discussion] Currently, batch effects cannot be learned with scDBMs.: "Currently, batch effects cannot be learned with scDBMs."
