---
name: single-cell-snv-read-count-likelihood-modeling
description: Use when joint inference of a tumor clonal tree is needed from SNV read
  counts where bulk DNA WES alternative/total matrices must be combined with scRNA-seq
  counts whose zeros arise from bursting, dropouts, or non-cancerous cells.
when_to_use_negative:
- No scRNA-seq SNV data are available (only bulk DNA-seq) — the predecessor Canopy,
  not the beta-binomial single-cell likelihood, is the appropriate model
- The read-count matrices contain mutations whose bursting kinetics (alpha, beta)
  cannot be estimated, since the beta-binomial single-cell likelihood requires these
  per-mutation rates
- Data have not passed the stringent QC pre-processing pipeline to produce a finalized
  SNV callset with Rs, Xs, Rb, Xb matrices
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: Canopy2
  role: R package implementing the Bayesian hierarchical model, the binomial/beta-binomial
    likelihood mixture, and the Metropolis-within-Gibbs sampler over Z, P^s, and P^b
  repo: https://github.com/annweideman/canopy2
- name: R
  role: Runtime environment; Canopy2 is an open-source R package installed from GitHub
    with devtools
  repo: https://github.com/annweideman/canopy2
- name: BPSC
  role: Provides the BPSC methodology used internally by get_burstiness_bpsc() to
    estimate mutation-specific alpha and beta bursting rates
  repo: https://github.com/nghiavtr/BPSC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:f50a54f32c7389ced0a30a9e5783e60a3a64f245ef412d9dfe60842d7ab45b22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - BPSC
  - Canopy2
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Single-cell SNV read-count likelihood modeling (Canopy2 beta-binomial/binomial mixture)

## Summary

Canopy2 models somatic SNV alternative/total read counts from scRNA-seq (Rs, Xs) with beta-binomial likelihoods and from bulk DNA WES (Rb, Xb) with a binomial likelihood, and samples the joint posterior over the clonal configuration Z, cell-to-clone assignments P^s, and bulk clone proportions P^b via Metropolis-within-Gibbs MCMC.

## When to use

Use when joint inference of a tumor clonal tree is needed from SNV read counts where bulk DNA WES alternative/total matrices must be combined with scRNA-seq counts whose zeros arise from bursting, dropouts, or non-cancerous cells.

## When NOT to use

- No scRNA-seq SNV data are available (only bulk DNA-seq) — the predecessor Canopy, not the beta-binomial single-cell likelihood, is the appropriate model
- The read-count matrices contain mutations whose bursting kinetics (alpha, beta) cannot be estimated, since the beta-binomial single-cell likelihood requires these per-mutation rates
- Data have not passed the stringent QC pre-processing pipeline to produce a finalized SNV callset with Rs, Xs, Rb, Xb matrices

## Inputs

- Rs: M x N matrix of alternative read counts from single-cell RNA-seq
- Xs: M x N matrix of total read counts (benign + mutated) from single-cell RNA-seq
- Rb: M x T matrix of alternative read counts from bulk DNA WES
- Xb: M x T matrix of total read counts from bulk DNA WES
- alpha, beta: 1 x M vectors of mutation-specific gene activation/deactivation rates
- kappa, tau: scalars defining the sequencing error rate kappa/(kappa+tau)
- Klist: range of candidate subclone numbers for BIC selection

## Outputs

- Z: M x K binary clonal tree configuration matrix assigning mutations to clones
- Ps: K x N binary cell-to-clone assignment matrix P^s
- Pb: K x T bulk sample composition matrix P^b (percentage of each bulk sample per clone)
- Posterior samples and Metropolis-Hastings acceptance rates from get_trees()
- Diagnostic plots (trace, ACF, posterior density) from get_diagnostics()
- Modified BIC values across Klist for tree selection

## How to apply

Assemble the M x N alternative/total scRNA-seq matrices (Rs, Xs) and M x T bulk WES matrices (Rb, Xb), subsetting to mutations for which gene burstiness is estimable (non-zero alternative counts across cells, non-zero total counts per cell). Estimate the 1 x M mutation-specific gene activation/deactivation rates alpha and beta with get_burstiness_bpsc() (recommended) or get_burstiness_scale(). Set sequencing-error hyperparameters kappa=1 and tau=999 so kappa/(kappa+tau)=0.001, matching the ~0.1% NGS error rate, then run get_trees() over a range of subclone numbers (e.g. Klist=4:8, niter=10000, nchains=10, thin=20, pburn=0.2), sampling Z, P^s, and P^b from their full conditionals by Metropolis-within-Gibbs. Select the configuration minimizing the modified BIC (per K, the tree with highest mean posterior density across post-burn-in iterations and chains is kept first), and verify convergence with get_diagnostics().

## Related tools

- **Canopy2** (R package implementing the Bayesian hierarchical model, the binomial/beta-binomial likelihood mixture, and the Metropolis-within-Gibbs sampler over Z, P^s, and P^b) — https://github.com/annweideman/canopy2
- **R** (Runtime environment; Canopy2 is an open-source R package installed from GitHub with devtools) — https://github.com/annweideman/canopy2
- **BPSC** (Provides the BPSC methodology used internally by get_burstiness_bpsc() to estimate mutation-specific alpha and beta bursting rates) — https://github.com/nghiavtr/BPSC

## Examples

```
trees <- get_trees(Rs=Rs, Xs=Xs, Rb=Rb, Xb=Xb, alpha=alpha, beta=beta, kappa=1, tau=999, Klist=4:8, niter=10000, nchains=10, thin=20, pburn=0.2)
```

## Evaluation signals

- Selected tree is the configuration with the minimum modified BIC across Klist (e.g. expected K=6 for the GBM10 demonstration dataset)
- Convergence diagnostics pass: lag ACF below the 0.3 threshold, spatially aligned posterior densities across chains, and random-noise-like trace plots (get_diagnostics())
- For each K in Klist, the retained tree has the highest mean posterior density across all post-burn-in iterations and all chains
- Output structure check: Z is a binary M x K matrix, Ps a binary K x N matrix, and Pb is a K x T composition matrix
- Metropolis-Hastings acceptance rates reported by get_trees() indicate reasonable mixing of the chain

## Limitations

- Reliance on Metropolis-within-Gibbs sampling creates a convergence bottleneck, since successive chain values depend on previous iterations
- Poor proposal distributions that do not allow sufficient mixing can delay convergence or cause the chain to settle on a local rather than global optimum
- Canopy2 was the slowest of the four methods benchmarked (microbenchmarked on a Windows laptop with an Intel Core i9-12900HK, ncores=20)
- Not published on CRAN as of 6/24/23 — the GitHub developmental version is recommended so post-publication bug fixes are available; get_gene_expression() has a known biomaRt/BiocFileCache/dbplyr compatibility bug requiring reinstalling BiocFileCache

## Evidence

- [abstract] Canopy2 uses Markov chain Monte Carlo methods to sample from a joint probability distribution involving a mixture of binomial and beta-binomial distributions: "Canopy2 uses Markov chain Monte Carlo methods to sample from a joint probability distribution involving a mixture of binomial and beta-binomial distributions"
- [methods] get_trees passes alpha and beta and outputs trees, posteriors, and MH acceptance rates: "The estimates for $\alpha$ and $\beta$ are passed to `get_trees()`, which outputs trees, posteriors, and Metropolis-Hastings acceptance rate"
- [readme] kappa and tau define the sequencing error rate of 0.1%: [evidence span withheld — anchor status: external_doc]
- [discussion] Metropolis-within-Gibbs convergence bottleneck limitation: "A limitation of our methodology is its reliance on sampling via Metropolis-within-Gibbs, where successive values in the chain depend on previous iterations, creating a bottleneck in convergence"
- [readme] Three-tier output structure Z, Ps, Pb: [evidence span withheld — anchor status: external_doc]
