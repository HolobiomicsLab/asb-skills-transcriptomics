---
name: clonal-configuration-error-evaluation
description: Use when benchmarking clonal-inference methods on simulated multi-sample
  bulk DNA WES plus scRNA-seq data and you need a permutation-invariant reconstruction
  error for the Z and P^s configuration matrices.
when_to_use_negative:
- The task is real-data application of Canopy2 rather than controlled simulation benchmarking
  of reconstruction error — use the three-tier Z/Ps/Pb output directly instead.
- Only one method is being run and no true clonal configuration exists against which
  to compute error_z/error_ps.
- The input data lack a known ground-truth Ktrue subclone structure (e.g., patient
  datasets like GBM2, GBM9, GBM10, BC03, BC07), so permutation-invariant error cannot
  be defined.
tools:
- name: Canopy2
  role: Provides simulate_data() to generate ground-truth Z and P^s, get_trees()/get_best_tree()
    to infer configurations, and error_z()/error_ps() to compute reconstruction errors
  repo: https://github.com/annweideman/canopy2
- name: Canopy
  role: Comparator method benchmarked on the same simulated read counts for Z and
    P^s reconstruction error
- name: Cardelino
  role: Comparator method run both with and without a guide clonal tree from a first-step
    Canopy run
- name: R
  role: Implementation environment for the canopy2 package and the benchmark scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:7592544e8055e5420cc6344b67abdca93019bebae8092ca25dea18489f7063ec
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# clonal-configuration-error-evaluation

## Summary

Quantifies how faithfully competing tumor phylogeny methods (Canopy2, Canopy, Cardelino with/without a guide clonal tree) reconstruct the mutation-to-clone matrix Z and the cell-to-clone assignment matrix P^s from simulated bulk DNA and scRNA-seq data, using permutation-invariant absolute error metrics error_z() and error_ps().

## When to use

Use when benchmarking clonal-inference methods on simulated multi-sample bulk DNA WES plus scRNA-seq data and you need a permutation-invariant reconstruction error for the Z and P^s configuration matrices.

## When NOT to use

- The task is real-data application of Canopy2 rather than controlled simulation benchmarking of reconstruction error — use the three-tier Z/Ps/Pb output directly instead.
- Only one method is being run and no true clonal configuration exists against which to compute error_z/error_ps.
- The input data lack a known ground-truth Ktrue subclone structure (e.g., patient datasets like GBM2, GBM9, GBM10, BC03, BC07), so permutation-invariant error cannot be defined.

## Inputs

- True (simulated) mutation-to-clone configuration matrix Z (M x K binary clonal tree matrix)
- True (simulated) cell-to-clone assignment matrix P^s (K x N binary matrix)
- Inferred Z and P^s from each benchmarked method (Canopy2 three-tier output, Canopy, Cardelino with/without guide tree)
- Simulated read-count matrices Rs, Xs (M x N scRNA-seq) and Rb, Xb (M x T bulk DNA WES) per seed

## Outputs

- Per-seed, per-method error_z values (absolute reconstruction error for Z)
- Per-seed, per-method error_ps values (absolute reconstruction error for P^s, minimized over all permutations of the true configuration)
- Aggregated error table across 100 seeds and bursting-kinetics settings written to simulation_error_results.csv

## How to apply

For each simulated dataset (generated via canopy2's simulate_data() with N=30 single cells, S=2 bulk samples, M=5 mutations, Ktrue=4 subclones, alpha=0.1/beta=1 baseline bursting rates varied across high, bursty, and low bursting kinetics, kappa=1 and tau=999 sequencing-error hyperparameters), run each method on the same read counts: Canopy2 via get_trees() with a Klist covering the true clone number, thinning=20 and 20% burn-in, selecting the tree with minimum BIC via get_best_tree(); Canopy; and Cardelino both with and without a guide clonal tree from a first-step Canopy run. Then compute the absolute reconstruction error for the clonal configuration matrix Z with error_z() and for the single-cell to clone assignment matrix P^s with error_ps(), taking the minimum distance over all permutations of the true configuration so that clone-label relabeling does not inflate the error. Aggregate errors per method and per bursting-kinetics setting across 100 seeds and write them to simulation_error_results.csv.

## Related tools

- **Canopy2** (Provides simulate_data() to generate ground-truth Z and P^s, get_trees()/get_best_tree() to infer configurations, and error_z()/error_ps() to compute reconstruction errors) — https://github.com/annweideman/canopy2
- **Canopy** (Comparator method benchmarked on the same simulated read counts for Z and P^s reconstruction error)
- **Cardelino** (Comparator method run both with and without a guide clonal tree from a first-step Canopy run)
- **R** (Implementation environment for the canopy2 package and the benchmark scripts)

## Examples

```
devtools::install_github('annweideman/canopy2'); sims.out <- simulate_data(N=30, S=2, M=5, alpha=0.1, beta=1, kappa=1, tau=999); trees <- get_trees(...); best <- get_best_tree(trees); err <- error_z(best$Z, true_Z)
```

## Evaluation signals

- error_z and error_ps values are produced for every method x bursting-kinetics combination across all 100 seeds (no missing cells in simulation_error_results.csv).
- error_ps is computed as the minimum distance over all permutations of the true configuration, so error values are invariant to clone-label ordering.
- Canopy2 exhibits consistently lower reconstruction error for Z than Canopy and Cardelino (with and without guide tree), matching the manuscript's reported simulation results.
- The inferred Klist range includes the true clone number Ktrue=4, and the selected tree is the one with minimum BIC from get_best_tree().
- Simulation parameters are reproducible: N=30, S=2, M=5, Ktrue=4, alpha=0.1, beta=1 (baseline), kappa=1, tau=999, bulk depth 30x-50x, single-cell depth 80x-120x, scale=300.

## Limitations

- Canopy2 is the slowest of the four methods evaluated in microbenchmarking (run on a Windows laptop with a 12th Gen Intel Core i9-12900HK processor, ncores=20).
- Reliance on Metropolis-within-Gibbs sampling creates a convergence bottleneck influenced by proposal distributions; poor proposals can delay convergence or settle on a local rather than global optimum.
- Canopy2 is not published on CRAN as of 6/24/23; the developmental GitHub version is recommended to pick up post-publication bug fixes.

## Evidence

- [abstract] Canopy2 outperforms competitors on Z reconstruction: "Simulations demonstrate that Canopy2 consistently outperforms competing methods and reconstructs the clonal tree with high fidelity"
- [methods] Simulation function call grounding parameters: "sims.out<-simulate_data(N=N, S=S, M=M, alpha=alpha, beta=beta, kappa=kappa,"
- [readme] BIC-based tree selection: [evidence span withheld — anchor status: external_doc]
- [readme] Definition of the Z output matrix being scored: [evidence span withheld — anchor status: external_doc]
