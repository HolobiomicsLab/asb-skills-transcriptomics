---
name: beta-binomial-zero-inflation-modeling
description: Use when single-cell mutation read counts (e.g., Rs/Xs matrices) show
  excess zeros from transcriptional bursting or dropout, and you need overdispersed
  counts modeling that decomposes zeros rather than a plain binomial likelihood.
when_to_use_negative:
- 'Your data are bulk DNA-seq only: the beta-binomial zero-inflation term targets
  scRNA-seq zeros; bulk DNA read counts are modeled with a plain binomial likelihood
  in Canopy2.'
- You only need a descriptive zero-classification without parameter inference — the
  three-tier Z/Ps/Pb sampling via MCMC is unnecessary overhead.
- Mutations lack estimable bursting kinetics (no burstiness estimate for alpha/beta);
  they must be subset out before modeling.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: Canopy2
  role: R package implementing the joint binomial (bulk) + beta-binomial (scRNA-seq)
    MCMC sampler via get_trees(); provides get_burstiness_bpsc()/get_burstiness_scale()
    for estimating alpha and beta
  repo: https://github.com/annweideman/canopy2
- name: R
  role: Runtime environment for the Canopy2 package and its MCMC and diagnostic functions
- name: BPSC
  role: Recommended methodology behind get_burstiness_bpsc() for estimating the mutation-specific
    bursting parameters alpha and beta
  repo: nghiavtr/BPSC
- name: SCALE
  role: Alternative methodology behind get_burstiness_scale() for estimating alpha
    and beta
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:635cc71507841869889ad4e6240a37309d0ced86184794b600ac666c2e2058f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - BPSC
  - Canopy2
  - R
  - SCALE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# beta-binomial zero-inflation modeling

## Summary

Model mutation-level alternative/total read counts from scRNA-seq with beta-binomial likelihoods whose overdispersion is driven by mutation-specific gene bursting rates, as done in Canopy2 to separate non-cancerous, stochastic (bursting), and technical zeros.

## When to use

Use when single-cell mutation read counts (e.g., Rs/Xs matrices) show excess zeros from transcriptional bursting or dropout, and you need overdispersed counts modeling that decomposes zeros rather than a plain binomial likelihood.

## When NOT to use

- Your data are bulk DNA-seq only: the beta-binomial zero-inflation term targets scRNA-seq zeros; bulk DNA read counts are modeled with a plain binomial likelihood in Canopy2.
- You only need a descriptive zero-classification without parameter inference — the three-tier Z/Ps/Pb sampling via MCMC is unnecessary overhead.
- Mutations lack estimable bursting kinetics (no burstiness estimate for alpha/beta); they must be subset out before modeling.

## Inputs

- Rs: M x N mutation-by-single-cell alternative read count matrix from scRNA-seq
- Xs: M x N mutation-by-single-cell total (benign + mutated) read count matrix from scRNA-seq
- alpha and beta: 1 x M vectors of mutation-specific gene activation and deactivation (bursting) rates
- kappa and tau: 1 x 1 scalars defining the sequencing error rate kappa/(kappa+tau)

## Outputs

- Z: M x K binary clonal tree configuration matrix assigning mutations to clones
- Ps: K x N binary cell-to-clone assignment matrix
- Pb: K x T sample-to-clone assignment matrix of bulk clone percentages
- Posterior samples, Metropolis-Hastings acceptance rates, and modified-BIC values across Klist

## How to apply

Estimate mutation-specific gene activation (alpha) and deactivation (beta) rates with get_burstiness_bpsc() (BPSC methodology, recommended) or get_burstiness_scale() (SCALE methodology); only mutations with estimable burstiness and non-zero alternative counts across cells should be retained (rowSums(Rs0)!=0 filtering). Set sequencing-error hyperparameters kappa=1 and tau=999 so the beta-distributed error rate has mean kappa/(kappa+tau)=0.001. Combine the beta-binomial likelihood for single-cell data with a binomial likelihood for bulk DNA data and sample Z (clonal configuration), P^s (cell-to-clone assignment), and P^b (bulk composition) via Metropolis-within-Gibbs MCMC, e.g., get_trees() with Klist=4:8, niter=10000, nchains=10, thin=20, pburn=0.2. The rationale is that the joint posterior has no closed form, so repeated full-conditional sampling is required, and bursting-induced stochastic zeros need an overdispersion mechanism the binomial lacks.

## Related tools

- **Canopy2** (R package implementing the joint binomial (bulk) + beta-binomial (scRNA-seq) MCMC sampler via get_trees(); provides get_burstiness_bpsc()/get_burstiness_scale() for estimating alpha and beta) — https://github.com/annweideman/canopy2
- **R** (Runtime environment for the Canopy2 package and its MCMC and diagnostic functions)
- **BPSC** (Recommended methodology behind get_burstiness_bpsc() for estimating the mutation-specific bursting parameters alpha and beta) — nghiavtr/BPSC
- **SCALE** (Alternative methodology behind get_burstiness_scale() for estimating alpha and beta)

## Examples

```
trees.out <- get_trees(Rs, Xs, Rb, Xb, alpha=param.est$alpha, beta=param.est$beta, kappa=1, tau=999, Klist=4:8, niter=10000, nchains=10, thin=20, pburn=0.2)
```

## Evaluation signals

- MCMC convergence checks: lag ACF below the 0.3 threshold, spatially aligned posterior densities, and random-noise trace plots from get_diagnostics()
- Metropolis-Hastings acceptance rates reported by get_trees() indicate adequate proposal mixing
- Optimal subclone count is the one minimizing the modified BIC across Klist (expected K=6 for the GBM10 demonstration)
- Selected tree's posterior Z, Ps, Pb matrices can be saved (canopy2_best_tree.rda) and inspected for binary-valued configuration entries
- Sequencing error hyperparameters yield a mean error rate of 0.001 (kappa/(kappa+tau) with kappa=1, tau=999)

## Limitations

- Reliance on Metropolis-within-Gibbs sampling creates a convergence bottleneck influenced by proposal distributions; successive chain values depend on previous iterations
- Poor proposal distributions can lead to settling on a local rather than global optimum or delayed convergence tied to the initial state
- Mutations for which gene burstiness (alpha/beta) cannot be estimated must be excluded, potentially reducing the mutation set
- Canopy2 is reported to be the slowest of the four methods evaluated in microbenchmarking

## Evidence

- [abstract] MCMC mixture of binomial and beta-binomial: "Canopy2 uses Markov chain Monte Carlo methods to sample from a joint probability distribution involving a mixture of binomial and beta-binomial distributions"
- [abstract] zeros decomposition: "Canopy2 demystifies the sources of zeros in the single-cell data and separates zeros categorized as non-cancerous (cells without mutations), stochastic (mutations not expressed due to bursting), and"
- [methods] alpha/beta estimation and passing to sampler: "The estimates for $\alpha$ and $\beta$ are passed to `get_trees()`, which outputs trees, posteriors, and Metropolis-Hastings acceptance rate"
- [readme] bursting rate inputs: [evidence span withheld — anchor status: external_doc]
- [readme] sequencing error defaults: [evidence span withheld — anchor status: external_doc]
- [discussion] convergence bottleneck limitation: "A limitation of our methodology is its reliance on sampling via Metropolis-within-Gibbs, where successive values in the chain depend on previous iterations, creating a bottleneck in convergence"
