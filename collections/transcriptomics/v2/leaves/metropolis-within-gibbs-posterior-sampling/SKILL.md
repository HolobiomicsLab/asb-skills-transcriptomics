---
name: metropolis-within-gibbs-posterior-sampling
description: Use when jointly inferring tumor phylogeny from bulk DNA-seq and scRNA-seq
  SNV read counts and the joint posterior over Z, P^s, P^b admits no closed form,
  requiring iterative full-conditional MCMC sampling.
when_to_use_negative:
- Input lacks single-cell RNA-seq read counts (Rs/Xs), so the beta-binomial bursting-likelihood
  component of the joint posterior cannot be formed — the predecessor Canopy, which
  uses bulk DNA alone, is the appropriate choice.
- Mutations have zero alternative read counts across all cells (or cells have zero
  total counts), so mutation-specific bursting rates alpha and beta cannot be estimated
  and the mutations should be filtered out before sampling.
- The goal is point-estimate inference where a closed-form posterior exists and MCMC
  sampling (with its convergence bottleneck) is unnecessary.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_2269
tools:
- name: Canopy2
  role: R package implementing the Bayesian hierarchical model and the get_trees/get_diagnostics/get_best_tree
    sampling workflow over Z, P^s, and P^b
  repo: https://github.com/annweideman/canopy2
- name: R
  role: Runtime environment in which the Canopy2 package's Metropolis-within-Gibbs
    sampler is executed
  repo: https://github.com/annweideman/canopy2
- name: BPSC
  role: Provides the get_burstiness_bpsc() methodology for estimating the mutation-specific
    alpha and beta bursting rates passed to get_trees()
  repo: https://github.com/nghiavtr/BPSC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:65e44adf8414a3c187431794f7b712146f38e6ed5f7dcb2656c286b62b0f210d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - BPSC
  - Canopy2
  - R
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metropolis-within-gibbs-posterior-sampling

## Summary

Canopy2's Bayesian sampling engine: because the joint posterior over clonal configuration Z, single-cell assignment P^s, and bulk composition P^b has no closed form, Metropolis-within-Gibbs MCMC repeatedly samples each parameter from its full conditional, mixing a binomial likelihood (bulk DNA WES) with beta-binomial likelihoods (scRNA-seq).

## When to use

Use when jointly inferring tumor phylogeny from bulk DNA-seq and scRNA-seq SNV read counts and the joint posterior over Z, P^s, P^b admits no closed form, requiring iterative full-conditional MCMC sampling.

## When NOT to use

- Input lacks single-cell RNA-seq read counts (Rs/Xs), so the beta-binomial bursting-likelihood component of the joint posterior cannot be formed — the predecessor Canopy, which uses bulk DNA alone, is the appropriate choice.
- Mutations have zero alternative read counts across all cells (or cells have zero total counts), so mutation-specific bursting rates alpha and beta cannot be estimated and the mutations should be filtered out before sampling.
- The goal is point-estimate inference where a closed-form posterior exists and MCMC sampling (with its convergence bottleneck) is unnecessary.

## Inputs

- Rs: M x N mutation-by-single-cell matrix of alternative read counts from scRNA-seq
- Xs: M x N mutation-by-single-cell matrix of total read counts (benign + mutated) from scRNA-seq
- Rb: M x T mutation-by-bulk-sample matrix of alternative read counts from bulk DNA WES
- Xb: M x T matrix of total read counts from bulk DNA WES
- alpha, beta: 1 x M vectors of mutation-specific gene activation/deactivation (bursting) rates
- kappa, tau: scalars setting the sequencing error rate kappa/(kappa+tau)
- Klist: range of candidate subclone numbers for BIC selection

## Outputs

- Sampled trees and posteriors for each K in Klist with Metropolis-Hastings acceptance rates
- Three-tier configuration: Z (M x K clonal tree matrix), Ps (K x N cell-to-clone matrix), Pb (K x T bulk-sample composition matrix)
- BIC values versus number of subclones for selecting the optimal tree configuration

## How to apply

Load the post-processed S4 object (e.g. GBM10_postproc from the canopy2 R package) and extract alternative/total count matrices Rs, Xs (single-cell) and Rb, Xb (bulk), plus mutation-specific bursting hyperparameters alpha and beta estimated by get_burstiness_bpsc() or get_burstiness_scale(). Set sequencing-error hyperparameters kappa=1 and tau=999 so kappa/(kappa+tau)=0.001, matching the ~0.1% NGS error rate. Run get_trees() over Klist=4:8 subclones with niter=10000, nchains=10, thin=20, pburn=0.2, sampling Z, P^s, and P^b via Metropolis-within-Gibbs from the joint posterior; the binomial term models bulk DNA counts while beta-binomial terms model scRNA-seq expression bursting. Assess convergence with get_diagnostics() using lag ACF below the 0.3 threshold, spatially aligned posterior densities, and random-noise trace plots. Finally call get_best_tree() to select the configuration minimizing the modified BIC across Klist (expected K=6 for GBM10) and save its Z, P^s, P^b matrices.

## Related tools

- **Canopy2** (R package implementing the Bayesian hierarchical model and the get_trees/get_diagnostics/get_best_tree sampling workflow over Z, P^s, and P^b) — https://github.com/annweideman/canopy2
- **R** (Runtime environment in which the Canopy2 package's Metropolis-within-Gibbs sampler is executed) — https://github.com/annweideman/canopy2
- **BPSC** (Provides the get_burstiness_bpsc() methodology for estimating the mutation-specific alpha and beta bursting rates passed to get_trees()) — https://github.com/nghiavtr/BPSC

## Examples

```
trees.out <- get_trees(Rs=Rs, Xs=Xs, Rb=Rb, Xb=Xb, alpha=param.est$alpha, beta=param.est$beta, kappa=1, tau=999, Klist=4:8, niter=10000, nchains=10, thin=20, pburn=0.2)
```

## Evaluation signals

- The selected tree is the configuration with minimum modified BIC across Klist (expected K=6 for the GBM10 demonstration dataset).
- get_diagnostics() lag ACF falls below the 0.3 threshold for sampled parameters.
- Posterior densities across chains are spatially aligned and trace plots look like random noise rather than trending walks.
- Metropolis-Hastings acceptance rates are reported by get_trees() and reviewed for adequate mixing of Z, P^s, and P^b.
- Output matrices conform to the three-tier schema: binary M x K Z, binary K x N Ps, and K x T Pb with clone percentages summing appropriately per bulk sample.

## Limitations

- Reliance on Metropolis-within-Gibbs means successive chain values depend on previous iterations, creating a convergence bottleneck influenced by the proposal distributions.
- Proposals that do not allow sufficient mixing can cause the chain to depend on its initial state, delay convergence, or settle on a local rather than global optimum.
- Canopy2 is the slowest of the four methods evaluated in benchmarking (microbenchmarked on a single Windows laptop with a 12th Gen Intel Core i9-12900HK processor, ncores=20).

## Evidence

- [abstract] abstract: "Canopy2 uses Markov chain Monte Carlo methods to sample from a joint probability distribution involving a mixture of binomial and beta-binomial distributions"
- [readme] readme: "`Rs`:   $M \times N$ (mutation by single-cell) matrix of alternative read counts from single-cell RNA sequencing data."
- [methods] methods: "The estimates for $\alpha$ and $\beta$ are passed to `get_trees()`, which outputs trees, posteriors, and Metropolis-Hastings acceptance rate"
- [discussion] discussion: "A limitation of our methodology is its reliance on sampling via Metropolis-within-Gibbs, where successive values in the chain depend on previous iterations, creating a bottleneck in convergence"
- [readme] readme_kappa: [evidence span withheld — anchor status: external_doc]
