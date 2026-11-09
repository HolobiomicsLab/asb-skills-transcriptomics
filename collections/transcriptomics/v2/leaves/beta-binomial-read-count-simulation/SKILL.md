---
name: beta-binomial-read-count-simulation
description: Use when you need synthetic bulk DNA and single-cell RNA read counts
  with known ground-truth clonal trees (Z, Ps) to quantify reconstruction error of
  tumor phylogeny inference methods.
when_to_use_negative:
- You have real bulk WES and scRNA-seq data — instead use the preprocessing pipeline
  (Zenodo scripts) and Canopy2's get_burstiness_bpsc()/get_burstiness_scale() to estimate
  alpha and beta directly
- The goal is phylogeny inference itself rather than method benchmarking — use get_trees()/get_best_tree()
  instead of simulate_data()
- Only bulk DNA data are available (no scRNA-seq) — predecessor Canopy, not Canopy2's
  joint beta-binomial simulation framework, is the appropriate model
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Canopy2
  role: Provides simulate_data() for beta-binomial/bursting-based read count simulation
    and error_z()/error_ps() for computing reconstruction error against ground-truth
    Z and P^s
  repo: https://github.com/annweideman/canopy2
- name: R
  role: Runtime environment for the Canopy2 package and the simulation-to-benchmark
    workflow
  repo: https://github.com/annweideman/canopy2
- name: Canopy
  role: Comparator method run on the same simulated read counts
- name: Cardelino
  role: Comparator method run on the same simulated read counts, with and without
    a guide clonal tree from a first-step Canopy run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:9039496fe9dadb19b70e81566b1c75ccf591036f5af3f8a1222ceb91ef58c33c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - Canopy
  - Canopy2
  - Cardelino
  - R
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Beta-binomial read count simulation for benchmarking tumor phylogeny inference

## Summary

Generates synthetic bulk DNA WES and scRNA-seq read counts under a beta-binomial model of transcriptional bursting so that mutation-to-clone (Z) and cell-to-clone (Ps) reconstruction error can be benchmarked across methods like Canopy2, Canopy, and Cardelino.

## When to use

Use when you need synthetic bulk DNA and single-cell RNA read counts with known ground-truth clonal trees (Z, Ps) to quantify reconstruction error of tumor phylogeny inference methods.

## When NOT to use

- You have real bulk WES and scRNA-seq data — instead use the preprocessing pipeline (Zenodo scripts) and Canopy2's get_burstiness_bpsc()/get_burstiness_scale() to estimate alpha and beta directly
- The goal is phylogeny inference itself rather than method benchmarking — use get_trees()/get_best_tree() instead of simulate_data()
- Only bulk DNA data are available (no scRNA-seq) — predecessor Canopy, not Canopy2's joint beta-binomial simulation framework, is the appropriate model

## Inputs

- Simulation design parameters: N (single cells, e.g. 30), S (bulk samples, e.g. 2), M (mutations, e.g. 5), Ktrue (true subclone number, e.g. 4)
- Mutation-specific gene activation (alpha) and deactivation (beta) rate vectors defining bursting kinetics
- Sequencing error hyperparameters kappa and tau (defaults 1 and 999)
- Bulk (30x-50x) and single-cell (80x-120x) sequencing depths and scale (e.g. 300)
- Random seed(s) for reproducibility across replicate simulations

## Outputs

- Simulated bulk DNA WES read counts (alternative Rb and total Xb, mutation-by-bulk-sample matrices)
- Simulated scRNA-seq read counts (alternative Rs and total Xs, mutation-by-single-cell matrices)
- Ground-truth clonal configuration matrix Z and cell-to-clone assignment matrix P^s
- Reconstruction error_z() and error_ps() values per method and bursting-kinetics setting (aggregated table, e.g. simulation_error_results.csv)

## How to apply

Use the Canopy2 function simulate_data() to generate data for each of 100 seeds with N=30 single cells, S=2 bulk samples, M=5 mutations, and Ktrue=4 subclones, specifying mutation-specific gene activation and deactivation rates (alpha, beta) to vary transcriptional bursting across high, bursty, and low bursting kinetics. Sequencing error is encoded via the kappa/(kappa+tau) ratio with defaults kappa=1 and tau=999, giving the 0.1% per-nucleotide NGS error rate; bulk depth is 30x-50x, single-cell depth 80x-120x, and scale=300. For each simulated dataset, run each method (Canopy2 via get_trees() over a Klist covering the true clone number with thinning=20 and 20% burn-in, selecting the minimum-BIC tree with get_best_tree(); plus Canopy and Cardelino with/without a guide clonal tree from a first-step Canopy run). Compute absolute reconstruction error for the clonal configuration matrix Z with error_z() and for the single-cell to clone assignment matrix P^s with error_ps(), taking the minimum distance over all permutations of the true configuration. Aggregate errors per method and bursting-kinetics setting across seeds (e.g. into simulation_error_results.csv) and confirm the expected ordering (Canopy2 lowest error). Alpha and beta for real-data applications can be estimated beforehand using get_burstiness_bpsc() (recommended) or get_burstiness_scale().

## Related tools

- **Canopy2** (Provides simulate_data() for beta-binomial/bursting-based read count simulation and error_z()/error_ps() for computing reconstruction error against ground-truth Z and P^s) — https://github.com/annweideman/canopy2
- **R** (Runtime environment for the Canopy2 package and the simulation-to-benchmark workflow) — https://github.com/annweideman/canopy2
- **Canopy** (Comparator method run on the same simulated read counts)
- **Cardelino** (Comparator method run on the same simulated read counts, with and without a guide clonal tree from a first-step Canopy run)

## Examples

```
sims.out <- simulate_data(N=30, S=2, M=5, alpha=alpha, beta=beta, kappa=1, tau=999, Ktrue=4, scale=300)
```

## Evaluation signals

- error_z and error_ps values are produced for every method x bursting-kinetics combination across all 100 seeds
- Canopy2 shows lower reconstruction error for Z and P^s than Canopy and Cardelino, consistent with the manuscript's simulation results
- Simulated datasets encode the expected kappa/(kappa+tau) sequencing error of 0.1% (kappa=1, tau=999)
- Ground-truth Ktrue subclones fall within the Klist passed to get_trees(), and the minimum-BIC tree is selectable via get_best_tree()
- Errors are minimized over all permutations of the true clonal configuration before comparison across methods

## Limitations

- Canopy2 is the slowest of the four benchmarked methods (microbenched on a Windows laptop with a 12th Gen Intel Core i9-12900HK processor, ncores=20)
- Reliance on Metropolis-within-Gibbs sampling creates a convergence bottleneck influenced by proposal distributions; poor proposals can delay convergence or settle on a local rather than global optimum
- Bursting kinetics parameters (alpha, beta) must be known or estimated; the SCALE/BPSC estimation approaches offer alternative estimators (get_burstiness_bpsc() recommended)

## Evidence

- [results] Across all four benchmarked methods, Canopy2 consistently demonstrated the best performance in reconstructing the mutation-to-clone assignment matrix Z in simulation studies.: [evidence span withheld — anchor status: not_found]
- [abstract] Simulations demonstrate that Canopy2 consistently outperforms competing methods and reconstructs the clonal tree with high fidelity, even in situations involving low sequencing depth, poor: "Simulations demonstrate that Canopy2 consistently outperforms competing methods and reconstructs the clonal tree with high fidelity, even in situations involving low sequencing depth, poor"
- [methods] sims.out<-simulate_data(N=N, S=S, M=M, alpha=alpha, beta=beta, kappa=kappa,: "sims.out<-simulate_data(N=N, S=S, M=M, alpha=alpha, beta=beta, kappa=kappa,"
- [readme] kappa and tau:   1 x 1 scalars used to compute the sequencing error by kappa/(kappa + tau). The average error rate of next-generation sequencing is reported to be 0.1% per nucleotide, so these parameters have been set to default values of kappa=1 and tau = 999: [evidence span withheld — anchor status: external_doc]
- [readme] alpha and beta: 1 x M (number of mutations) vectors of mutation specific gene activation and gene deactivation rates. These can be estimated using either the BPSC methodology in function get_burstiness_bpsc() (recommended) or the SCALE methodology with function get_burstiness_scale().: [evidence span withheld — anchor status: external_doc]
