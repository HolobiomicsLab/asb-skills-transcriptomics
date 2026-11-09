---
name: random-walk-hitting-time-computation
description: Use when reconstructing pseudotime/directionality on a PARC cluster graph
  for single-cell trajectory data (e.g., mESC CyTOF cell-cycle), where a root cluster
  is known and hitting-time-based ordering is needed.
when_to_use_negative:
- Data lacks a defensible root/start population, since hitting times are computed
  from a specified root cluster
- The goal is a fully data-driven tree/fate analysis without cluster-level abstraction
  — VIA's cluster-graph hitting times assume PARC clustering granularity is appropriate
- Trajectory topology requires higher-order memory effects beyond the first-order
  lazy-teleporting walk (StaVia/Via 2.0's memory walks address this)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: VIA
  role: Implements the lazy-teleporting expected-hitting-time pseudotime computation
    and MCMC refinement on the cluster graph
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Builds the jaccard-pruned, Leiden-clustered graph on which the hitting-time
    random walk is run
  repo: https://github.com/ShobiStassen/VIA
- name: dyntoy
  role: Source of toy/simulated trajectory datasets used in VIA tutorials (e.g., 4-leaf
    multifurcation)
  repo: https://github.com/dynverse/dyntoy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:3972985b3007050f587389233a27ff00628419c18a8207ffcbacbcb059b5e89b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - VIA
  - dyntoy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# random-walk-hitting-time-computation

## Summary

Computes single-cell pseudotime and graph directionality on the PARC cluster graph via the theoretical expected hitting times of a lazy-teleporting random walk (VIA stage 1, Eqs. 1-8), followed by lazy-teleporting MCMC refinement on the forward-biased graph (VIA stage 2).

## When to use

Use when reconstructing pseudotime/directionality on a PARC cluster graph for single-cell trajectory data (e.g., mESC CyTOF cell-cycle), where a root cluster is known and hitting-time-based ordering is needed.

## When NOT to use

- Data lacks a defensible root/start population, since hitting times are computed from a specified root cluster
- The goal is a fully data-driven tree/fate analysis without cluster-level abstraction — VIA's cluster-graph hitting times assume PARC clustering granularity is appropriate
- Trajectory topology requires higher-order memory effects beyond the first-order lazy-teleporting walk (StaVia/Via 2.0's memory walks address this)

## Inputs

- mESC CyTOF cell-cycle dataset (loaded via pyVIA.datasets_via cell_cycle / cell_cycle_cyto_data loaders)
- PARC cluster graph of cells (Leiden clustering, jaccard-weighted pruned edges)
- Root/start cell or cluster specification (root_user)

## Outputs

- single-cell pseudotime (single_cell_pt_markov) derived from expected hitting times
- Directed cluster graph with terminal states flagged at the 30th pseudotime percentile
- Refined pseudotime and forward-biased branch probabilities from MCMC simulations
- pseudotime_validation.csv containing Spearman correlations with day labels and full-VIA pseudotimes

## How to apply

Build the PARC cluster graph with jaccard-weighted edge pruning (edgepruning_clustering_resolution=0.15, too_big_factor=0.4, small_pop=10, Leiden resolution_parameter=1, random_seed=42). In stage 1, compute the closed-form expected hitting time of the modified lazy-teleporting random walk from the root cluster over the cluster graph using x_lazy=0.95 and alpha_teleport=0.99; the resulting expected hitting times define pseudotime (single_cell_pt_markov) and hence graph directionality. Flag terminal states at pseudotime_threshold_TS=30 percentile, then in stage 2 refine pseudotime and branch probabilities with num_sim_branch_probability=500 lazy-teleporting MCMC simulations per terminal state on the forward-biased graph. Validate by Spearman-correlating the standalone pseudotimes against annotated day labels and against the full VIA run's pseudotimes.

## Related tools

- **VIA** (Implements the lazy-teleporting expected-hitting-time pseudotime computation and MCMC refinement on the cluster graph) — https://github.com/ShobiStassen/VIA
- **PARC** (Builds the jaccard-pruned, Leiden-clustered graph on which the hitting-time random walk is run) — https://github.com/ShobiStassen/VIA
- **dyntoy** (Source of toy/simulated trajectory datasets used in VIA tutorials (e.g., 4-leaf multifurcation)) — https://github.com/dynverse/dyntoy

## Examples

```
v = via.core.VIA(adata.obsm['X_pca'], true_label, jac_std_global=0.15, too_big_factor=0.4, small_pop=10, root_user=[42], x_lazy=0.95, alpha_teleport=0.99, pseudotime_threshold_TS=30, num_sim_branch_probability=500); v.run_via()
```

## Evaluation signals

- Spearman correlation of standalone pseudotimes with annotated day labels is positive and strong
- Spearman correlation of standalone pseudotimes with full-VIA pseudotimes is positive and strong
- pseudotime_validation.csv exists and contains both correlation values
- Terminal states are flagged at the expected pseudotime_threshold_TS=30 percentile and each receives 500 MCMC simulations (num_sim_branch_probability=500)
- Stage-1 parameters match the canonical values x_lazy=0.95 and alpha_teleport=0.99

## Limitations

- Stage-1 pseudotime depends on the correctness of the root cluster and PARC clustering granularity (edgepruning_clustering_resolution, too_big_factor, small_pop)
- First-order lazy-teleporting walks may miss higher-order pathway structure; StaVia (Via 2.0) extends to higher-order random walks with memory for lineage detection
- TOO-large clusters (share of cells > too_big_factor=0.4) trigger re-clustering, changing the graph the walk runs on

## Evidence

- [abstract] VIA uses lazy-teleporting random walks: "We present VIA, a scalable trajectory inference algorithm that overcomes these limitations by using lazy-teleporting random walks"
- [intro] Two-stage procedure: pseudotime strategy plus MCMC lineage reconstruction: "we present VIA, a graph-based TI algorithm that uses a new strategy to compute pseudotime, and reconstruct cell lineages based on lazy-teleporting random walks integrated with Markov chain Monte"
- [intro] Cluster graph from PARC underlies trajectory computations: "Single-cell level graph is clustered such that each node represents a cluster of single cells (computed by our clustering algorithm PARC14)"
- [discussion] mESC CyTOF validation of lazy-teleporting MCMC strategy: "We first applied VIA to the mESC CyTOF dataset and showed that the lazy-teleporting MCMCs strategy in VIA enables it to outperform other methods"
- [methods] Terminal-state percentile filter parameter: [evidence span withheld — anchor status: not_found]
- [readme] Lazy-teleporting random walks reconstruct complex trajectories: [evidence span withheld — anchor status: not_found]
