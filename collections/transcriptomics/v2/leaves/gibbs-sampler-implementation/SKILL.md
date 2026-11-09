---
name: gibbs-sampler-implementation
description: Use when clustering droplet-based UMI count matrices from multiple individuals
  with BAMM-SC and you must run, tune, or validate the MCMC/Gibbs sampler (burn-in,
  iteration budget, log-likelihood tolerance) behind the hierarchical Dirichlet multinomial
  mixture.
when_to_use_negative:
- Input is a single UMI count matrix rather than a list of per-individual matrices
  — the package then falls back to DIMMSC instead of the multi-individual Gibbs sampler
- You need explicit modeling of zero inflation or dropouts, which BAMM-SC's multinomial-Dirichlet
  likelihood does not model
- You expect posterior probabilities to quantify cell cycle or developmental stage
  — BAMM-SC assumes each cell belongs to one distinct cluster and posterior probability
  only measures clustering uncertainty
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BAMMSC
  role: R package providing the BAMMSC() Gibbs sampler over the hierarchical Dirichlet
    multinomial mixture and BAMMSC.predict() for supervised label prediction
  repo: https://github.com/lichen-lab/BAMMSC
- name: dirmult
  role: Dependency attached for Dirichlet-multinomial computations used by the sampler
- name: Rcpp
  role: Dependency enabling the compiled MCMC routines of the package
- name: RcppArmadillo
  role: Dependency providing linear algebra backend for the Gibbs sampler
- name: mclust
  role: Provides adjustedRandIndex to score recovered cell labels against simulated
    true cluster labels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:24ed104e2ba37bf8ac1d41f308e6df58f7d0d11a69478ed20f81d506e1cec669
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - BAMM-SC
  - BAMMSC
  - DIMM-SC
  - Rcpp
  - RcppArmadillo
  - dirmult
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# gibbs-sampler-implementation

## Summary

Implements the Gibbs MCMC core of BAMM-SC, a Bayesian hierarchical Dirichlet multinomial mixture model, alternating updates of the Dirichlet concentration parameters alpha_ilk (under a log-normal LN(mu_ik, sigma_ik^2) prior) and cell cluster labels z_jl for multi-individual droplet-based UMI counts.

## When to use

Use when clustering droplet-based UMI count matrices from multiple individuals with BAMM-SC and you must run, tune, or validate the MCMC/Gibbs sampler (burn-in, iteration budget, log-likelihood tolerance) behind the hierarchical Dirichlet multinomial mixture.

## When NOT to use

- Input is a single UMI count matrix rather than a list of per-individual matrices — the package then falls back to DIMMSC instead of the multi-individual Gibbs sampler
- You need explicit modeling of zero inflation or dropouts, which BAMM-SC's multinomial-Dirichlet likelihood does not model
- You expect posterior probabilities to quantify cell cycle or developmental stage — BAMM-SC assumes each cell belongs to one distinct cluster and posterior probability only measures clustering uncertainty

## Inputs

- X: a list of UMI count matrices, one per individual, with rows as genes and columns as cells (same genes across individuals, cell numbers may differ)
- K: number of clusters (e.g., K=4)
- MCMC settings: nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10
- Optional: true cell labels Z for ARI evaluation via mclust

## Outputs

- mem: list of per-individual cell clustering membership vectors
- loglik: vector of log-likelihood values across all MCMC iterations
- alpha: 3-D matrix estimates [#gene, #cluster, #individuals] per iteration
- sigma2: 3-D matrix of sigma^2 estimates [#gene, #cluster, #iteration]
- delta: per-individual posterior probability matrices [#cell, #cluster, #iteration]
- pi: cluster proportion estimates; AIC and BIC for model checking
- ARI value (via mclust::adjustedRandIndex) when true labels are available

## How to apply

Represent the data as a list of gene-by-cell UMI count matrices (one element per individual; identical gene sets, cell counts may differ). Implement the multinomial-Dirichlet likelihood with the log-normal LN(mu_ik, sigma_ik^2) prior on alpha_ilk capturing gene/cell-type/individual heterogeneity and a Gamma hyper-prior on sigma^2, then run the Gibbs sampler alternating updates of alpha_ilk and cell labels z_jl. Configure MCMC with nBurn=400 burn-in, maxIter.BAMMSC=500 maximum iterations, and likTol.BAMMSC=1e-10 log-likelihood convergence tolerance, averaging alpha estimates over the post-burn-in portion of the chain. Choose the number of clusters K from prior tissue knowledge or model-checking criteria (AIC/BIC are returned), and initialize clusters via kmeans with Ronning's method for alpha by default. Judge convergence by a plateaued MCMC log-likelihood trace and clustering quality by mclust's adjustedRandIndex against known labels.

## Related tools

- **BAMMSC** (R package providing the BAMMSC() Gibbs sampler over the hierarchical Dirichlet multinomial mixture and BAMMSC.predict() for supervised label prediction) — https://github.com/lichen-lab/BAMMSC
- **dirmult** (Dependency attached for Dirichlet-multinomial computations used by the sampler)
- **Rcpp** (Dependency enabling the compiled MCMC routines of the package)
- **RcppArmadillo** (Dependency providing linear algebra backend for the Gibbs sampler)
- **mclust** (Provides adjustedRandIndex to score recovered cell labels against simulated true cluster labels)

## Examples

```
library(BAMMSC); data("data_BAMMSC"); result=BAMMSC(data_BAMMSC,K=4,nBurn=400,maxIter.BAMMSC=500,likTol.BAMMSC=1e-10)
```

## Evaluation signals

- MCMC log-likelihood trace (result$loglik) plateaus within the maxIter.BAMMSC=500 iteration budget after nBurn=400 burn-in
- adjustedRandIndex(unlist(result$mem), unlist(Z)) is high when clustering the simulated data_BAMMSC with 10 individuals, 100 genes, 400 cells each and K=4 true clusters
- alpha estimates are averaged over the post-burn-in portion of the chain, not taken from a single iteration
- Output dimensions check out: mem is a list of length #individuals, sigma2 is [#gene, #cluster, #iteration], delta is [#cell, #cluster, #iteration] per individual
- K chosen or validated via returned AIC/BIC rather than a single arbitrary value

## Limitations

- Genes with excessive zeros are filtered before analysis, which may be problematic for rare cell type identification
- Zero-inflation is not explicitly modeled, which may or may not affect clustering accuracy
- Each cell is assumed to belong to one distinct cluster; posterior probability measures only clustering uncertainty
- The supervised strategy for large datasets (BAMMSC.predict, p=0.5) may potentially miss some rare clusters; the authors recommend trying a range of cluster numbers in practice

## Evidence

- [methods] log-normal prior: [evidence span withheld — anchor status: not_found]
- [methods] post-burn-in averaging: [evidence span withheld — anchor status: not_found]
- [readme] readme burn-in: "nBurn: number of iterations discarded at the initial portion of the Markov chain, default is 400"
- [readme] readme tolerance: "likTol.BAMMSC: a convergence tolerance for the difference of log-likelihoods between iterations in MCMC, default is 1e-10"
- [abstract] heterogeneity prior: [evidence span withheld — anchor status: not_found]
