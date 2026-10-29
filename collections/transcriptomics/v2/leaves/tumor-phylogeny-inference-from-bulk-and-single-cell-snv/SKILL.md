---
name: tumor-phylogeny-inference-from-bulk-and-single-cell-snv
description: Use when you have matched bulk DNA WES and scRNA-seq SNV read counts
  (Rs, Xs, Rb, Xb) for the same tumor and need to infer subclone number, mutational
  profiles, cell-of-origin, and cell-to-clone assignments jointly.
when_to_use_negative:
- You have only scRNA-seq SNVs without matched bulk DNA-seq data — Cardelino with
  a guide clonal tree is the alternative that takes scRNA-seq input indirectly informed
  by a first-step Canopy run.
- The read count matrices have not passed the stringent QC / pre-processing pipeline
  (somatic and germline variant calling; Zenodo scripts) needed to produce the finalized
  SNV callset.
- Mutations lack estimable bursting kinetics — subset alternative and total read counts
  to only mutations for which gene burstiness (alpha, beta) can be estimated before
  calling get_trees().
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_3308
tools:
- name: Canopy2
  role: 'Primary R package: implements the Bayesian hierarchical model, get_trees()
    MCMC sampler, get_diagnostics(), and get_best_tree() BIC selection'
  repo: https://github.com/annweideman/canopy2
- name: R
  role: Execution environment; Canopy2 is an open-source R package installed via devtools::install_github
- name: BPSC
  role: Dependency used by get_burstiness_bpsc() to estimate mutation-specific gene
    activation (alpha) and deactivation (beta) rates
  repo: https://github.com/nghiavtr/BPSC
- name: Canopy
  role: Predecessor method; used to produce a guide clonal tree for Cardelino in benchmarking
    comparisons
- name: Cardelino
  role: Comparison method that takes scRNA-seq input with a guide clonal tree from
    a first-step Canopy run
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:6d6c402f106676b0bc355bb86fd468840aac30e7af09572679a78cf530ba7ee0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - BPSC
  - Canopy
  - Canopy2
  - Cardelino
  - R
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tumor-phylogeny-inference-from-bulk-and-single-cell-snv

## Summary

Joint Bayesian inference of tumor phylogeny from SNVs derived from bulk DNA WES and scRNA-seq using the Canopy2 R package, which samples the clonal configuration Z, single-cell assignment P^s, and bulk composition P^b via Markov chain Monte Carlo from a mixture of binomial (bulk) and beta-binomial (single-cell) likelihoods.

## When to use

Use when you have matched bulk DNA WES and scRNA-seq SNV read counts (Rs, Xs, Rb, Xb) for the same tumor and need to infer subclone number, mutational profiles, cell-of-origin, and cell-to-clone assignments jointly.

## When NOT to use

- You have only scRNA-seq SNVs without matched bulk DNA-seq data — Cardelino with a guide clonal tree is the alternative that takes scRNA-seq input indirectly informed by a first-step Canopy run.
- The read count matrices have not passed the stringent QC / pre-processing pipeline (somatic and germline variant calling; Zenodo scripts) needed to produce the finalized SNV callset.
- Mutations lack estimable bursting kinetics — subset alternative and total read counts to only mutations for which gene burstiness (alpha, beta) can be estimated before calling get_trees().

## Inputs

- Rs: M x N (mutation by single-cell) matrix of alternative read counts from scRNA-seq
- Xs: M x N (mutation by single-cell) matrix of total read counts from scRNA-seq
- Rb: M x T (mutation by bulk sample) matrix of alternative read counts from bulk DNA WES
- Xb: M x T (mutation by bulk sample) matrix of total read counts from bulk DNA WES
- alpha and beta: 1 x M vectors of mutation-specific gene activation and deactivation rates
- kappa and tau: scalars defining the sequencing error rate kappa/(kappa+tau) (default 1 and 999)
- Klist: range of candidate numbers of subclones for BIC selection

## Outputs

- Z: M x K binary clonal tree configuration matrix assigning mutations to clones
- Ps: K x N binary cell-to-clone assignment matrix (P^s)
- Pb: K x T sample-to-clone assignment matrix giving bulk clone percentages (P^b)
- Trees, posteriors, and Metropolis-Hastings acceptance rates from get_trees()
- BIC values versus number of subclones for selecting the optimal configuration
- Diagnostic plots (ACF, posterior densities, trace plots) from get_diagnostics()

## How to apply

Load the post-processed S4 object (e.g., GBM10_postproc) and extract alternative/total read count matrices Rs, Xs (single-cell, M x N) and Rb, Xb (bulk, M x T), plus mutation-specific bursting hyperparameters alpha and beta estimated with get_burstiness_bpsc() or get_burstiness_scale(). Set sequencing-error hyperparameters kappa=1 and tau=999 so the beta-distributed error rate has mean 0.001. Run get_trees() over a range of subclone counts (e.g., Klist=4:8 or 3:10) with multiple chains (>=10), niter=50k-100k, thin=20, and pburn=0.2 on a computing cluster; the sampler mixes binomial bulk DNA likelihoods with beta-binomial scRNA-seq likelihoods via Metropolis-within-Gibbs. Assess convergence with get_diagnostics(): lag ACF below 0.3, spatially aligned posterior densities, and trace plots resembling random noise. Select the optimal tree with get_best_tree(), which minimizes a modified BIC across Klist, and save the winning configuration's Z, P^s, and P^b matrices.

## Related tools

- **Canopy2** (Primary R package: implements the Bayesian hierarchical model, get_trees() MCMC sampler, get_diagnostics(), and get_best_tree() BIC selection) — https://github.com/annweideman/canopy2
- **R** (Execution environment; Canopy2 is an open-source R package installed via devtools::install_github)
- **BPSC** (Dependency used by get_burstiness_bpsc() to estimate mutation-specific gene activation (alpha) and deactivation (beta) rates) — https://github.com/nghiavtr/BPSC
- **Canopy** (Predecessor method; used to produce a guide clonal tree for Cardelino in benchmarking comparisons)
- **Cardelino** (Comparison method that takes scRNA-seq input with a guide clonal tree from a first-step Canopy run)

## Examples

```
trees <- get_trees(Rs, Xs, Rb, Xb, param.est$alpha, param.est$beta, kappa=1, tau=999, Klist=4:8, niter=10000, nchains=10, thin=20, pburn=0.2)
```

## Evaluation signals

- get_best_tree() converges on the configuration with minimum modified BIC across Klist (e.g., expected K=6 for GBM10)
- MCMC diagnostics pass: lag ACF below 0.3, spatially aligned posterior densities, and trace plots resembling random noise
- Output schema matches the three-tier configuration: Z is M x K binary, Ps is K x N binary, and Pb is K x T percentages
- Metropolis-Hastings acceptance rates returned by get_trees() indicate adequate mixing across chains
- Benchmark against Canopy and Cardelino: Canopy2 should leave fewer malignant cells unassigned (e.g., 0% for BC07 and GBM10 vs. 29.4% and 16.2% for Cardelino with guide)

## Limitations

- Canopy2 was the slowest of the four evaluated methods in microbenchmarking on a single Windows laptop (12th Gen Intel Core i9-12900HK, ncores=20); cluster execution with many iterations is recommended.
- Reliance on Metropolis-within-Gibbs creates a convergence bottleneck since successive chain values depend on previous iterations.
- Poor proposal distributions can lead to dependencies on the initial state, delaying convergence or settling on a local rather than global optimum.
- The CRAN release was not published as of 6/24/23; the GitHub developmental version is recommended so corrected bugs are available.

## Evidence

- [abstract] Canopy2 uses Markov chain Monte Carlo methods to sample from a joint probability distribution involving a mixture of binomial and beta-binomial distributions: "Canopy2 uses Markov chain Monte Carlo methods to sample from a joint probability distribution involving a mixture of binomial and beta-binomial distributions"
- [methods] get_trees passes alpha and beta and outputs trees, posteriors, acceptance rates: "The estimates for $\alpha$ and $\beta$ are passed to `get_trees()`, which outputs trees, posteriors, and Metropolis-Hastings acceptance rate"
- [readme] README description of the method scope: "An R package that uses Bayesian and computational methods to infer the tumor phylogenetic tree from bulk DNA whole exome sequencing (WES) and single-cell RNA sequencing (scRNA-seq)."
- [readme] README output definition for Z: [evidence span withheld — anchor status: external_doc]
- [discussion] Metropolis-within-Gibbs convergence limitation: "A limitation of our methodology is its reliance on sampling via Metropolis-within-Gibbs, where successive values in the chain depend on previous iterations, creating a bottleneck in convergence"
