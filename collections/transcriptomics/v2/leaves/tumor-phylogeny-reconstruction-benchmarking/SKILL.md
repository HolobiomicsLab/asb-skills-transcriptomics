---
name: tumor-phylogeny-reconstruction-benchmarking
description: Use when evaluating whether a Bayesian tumor phylogeny inference method
  (e.g., Canopy2) recovers true clonal configurations Z and single-cell clone assignments
  Ps better than competing methods across simulated bursting-kinetics settings.
when_to_use_negative:
- The goal is inference on real patient data rather than simulation-based benchmarking
  against a known ground-truth Z and Ps
- Input lacks paired bulk DNA WES and single-cell RNA-seq SNV read counts, which Canopy2
  requires as Rs/Xs/Rb/Xb
- Only a single method is to be run; the skill's value comes from the head-to-head
  error comparison with Canopy and Cardelino
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: Canopy2
  role: Primary method under benchmark; run via simulate_data(), get_trees(), and
    get_best_tree() on simulated bulk/scRNA-seq read counts
  repo: https://github.com/annweideman/canopy2
- name: Canopy
  role: Comparator method for reconstruction-error benchmarking and source of the
    guide clonal tree used in one Cardelino configuration
- name: Cardelino
  role: Comparator method run both with and without a guide clonal tree from a first-step
    Canopy run
- name: R
  role: Implementation and execution environment for the benchmark scripts
  repo: https://github.com/annweideman/canopy2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:49feb268adbbe5e8c7c73f360d7ec6d37e88ae28c7148ee173f3d687636d713f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - Canopy
  - Canopy2
  - Cardelino
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# tumor-phylogeny-reconstruction-benchmarking

## Summary

Benchmark Canopy2 against Canopy and Cardelino (with/without a guide clonal tree) on simulated bulk DNA-seq plus scRNA-seq read counts, quantifying reconstruction error of the mutation-to-clone matrix Z and the cell-to-clone matrix Ps.

## When to use

Use when evaluating whether a Bayesian tumor phylogeny inference method (e.g., Canopy2) recovers true clonal configurations Z and single-cell clone assignments Ps better than competing methods across simulated bursting-kinetics settings.

## When NOT to use

- The goal is inference on real patient data rather than simulation-based benchmarking against a known ground-truth Z and Ps
- Input lacks paired bulk DNA WES and single-cell RNA-seq SNV read counts, which Canopy2 requires as Rs/Xs/Rb/Xb
- Only a single method is to be run; the skill's value comes from the head-to-head error comparison with Canopy and Cardelino

## Inputs

- Simulated Rs (M x N) alternative read count matrix from single-cell RNA sequencing
- Simulated Xs (M x N) total read count matrix from single-cell RNA sequencing
- Simulated Rb (M x T) alternative read count matrix from bulk DNA WES
- Simulated Xb (M x T) total read count matrix from bulk DNA WES
- Mutation-specific gene activation (alpha) and deactivation (beta) rate vectors
- True clonal configuration matrix Z and single-cell assignment matrix Ps (ground truth)

## Outputs

- Per-method error_z and error_ps values across 100 seeds and three bursting-kinetics settings
- Aggregated comparison table written to simulation_error_results.csv

## How to apply

Install Canopy2 from GitHub via devtools::install_github('annweideman/canopy2'). For each of 100 seeds, generate simulated data with simulate_data() using N=30 single cells, S=2 bulk samples, M=5 mutations, Ktrue=4 subclones, kappa=1 and tau=999 (giving a 0.1% sequencing error rate), bulk depth 30x-50x, single-cell depth 80x-120x, and scale=300, varying alpha and beta across high, bursty, and low bursting kinetics. Run Canopy2 via get_trees() with a Klist spanning the true clone number, thinning=20, and 20% burn-in, then pick the tree with minimum BIC using get_best_tree(). Run Canopy and Cardelino (with and without a guide clonal tree from a first-step Canopy run) on the same simulated read counts. Compute absolute reconstruction errors with error_z() and error_ps(), taking the minimum distance over all permutations of the true configuration, then aggregate per method and kinetics setting into simulation_error_results.csv.

## Related tools

- **Canopy2** (Primary method under benchmark; run via simulate_data(), get_trees(), and get_best_tree() on simulated bulk/scRNA-seq read counts) — https://github.com/annweideman/canopy2
- **Canopy** (Comparator method for reconstruction-error benchmarking and source of the guide clonal tree used in one Cardelino configuration)
- **Cardelino** (Comparator method run both with and without a guide clonal tree from a first-step Canopy run)
- **R** (Implementation and execution environment for the benchmark scripts) — https://github.com/annweideman/canopy2

## Examples

```
devtools::install_github('annweideman/canopy2'); sims.out <- simulate_data(N=30, S=2, M=5, alpha=alpha, beta=beta, kappa=1, tau=999, scale=300)
```

## Evaluation signals

- error_z and error_ps values are produced for every method x bursting-kinetics combination across all 100 seeds
- Canopy2 shows lower reconstruction error for Z than Canopy and Cardelino, matching the manuscript's simulation results
- Selected tree is the minimum-BIC configuration from the Klist explored by get_trees()
- Error computation uses the minimum distance over all permutations of the true clonal configuration (permutation-matched errors)
- Aggregated results are consistent across the high, bursty, and low bursting kinetics settings (e.g., results hold under varied alpha/beta)

## Limitations

- Canopy2 was the slowest of the four methods evaluated in microbenchmarking (run on a Windows laptop with a 12th Gen Intel Core i9-12900HK processor, ncores=20)
- Reliance on Metropolis-within-Gibbs sampling creates a convergence bottleneck influenced by proposal distributions; poor proposals can cause settling on a local rather than global optimum
- No software version stated for Canopy2 or R, and the CRAN release was not published as of 6/24/23, requiring the developmental GitHub install

## Evidence

- [abstract] Simulations demonstrate that Canopy2 consistently outperforms competing methods and reconstructs the clonal tree with high fidelity, even in situations involving low sequencing depth, poor: "Simulations demonstrate that Canopy2 consistently outperforms competing methods and reconstructs the clonal tree with high fidelity, even in situations involving low sequencing depth, poor"
- [methods] The estimates for alpha and beta are passed to get_trees(), which outputs trees, posteriors, and Metropolis-Hastings acceptance rate: "The estimates for $\alpha$ and $\beta$ are passed to `get_trees()`, which outputs trees, posteriors, and Metropolis-Hastings acceptance rate"
- [readme] An R package that uses Bayesian and computational methods to infer the tumor phylogenetic tree from bulk DNA whole exome sequencing (WES) and single-cell RNA sequencing (scRNA-seq).: "An R package that uses Bayesian and computational methods to infer the tumor phylogenetic tree from bulk DNA whole exome sequencing (WES) and single-cell RNA sequencing (scRNA-seq)."
- [intro] Cardelino [13] takes scRNA-seq as input and indirectly utilizes bulk DNA-seq data through a guide clonal tree produced from a first-step run of Canopy [21]: "Cardelino [13] takes scRNA-seq as input and indirectly utilizes bulk DNA-seq data through a guide clonal tree produced from a first-step run of Canopy [21]"
- [discussion] A limitation of our methodology is its reliance on sampling via Metropolis-within-Gibbs, where successive values in the chain depend on previous iterations: "A limitation of our methodology is its reliance on sampling via Metropolis-within-Gibbs, where successive values in the chain depend on previous iterations"
