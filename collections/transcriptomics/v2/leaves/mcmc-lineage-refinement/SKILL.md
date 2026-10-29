---
name: mcmc-lineage-refinement
description: Use when you need to reproduce or unit-test VIA's pseudotime computation
  and forward-biased MCMC lineage refinement on a PARC cluster graph (e.g. mESC CyTOF
  cell-cycle data) with explicit x_lazy/alpha_teleport settings.
when_to_use_negative:
- You need atlas-scale visualization with metadata integration, higher-order random
  walks with memory, or Atlas View — those are StaVia (Via 2.0) features, not this
  Via 1.0 hitting-time + MCMC stage
- The input is raw single-cell data without a PARC cluster graph or a defined root
  cell — the expected hitting time computation requires a rooted cluster graph
- You need only pseudotime ranking with no lineage/branch-probability output — the
  500-simulation-per-terminal-state MCMC stage is then unnecessary overhead
tools:
- name: VIA
  role: Implements the lazy-teleporting expected-hitting-time pseudotime and the forward-biased
    MCMC branch-probability refinement (via.via_wrapper / pyVIA standalone module)
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Builds the cluster graph on which the random-walk hitting times and MCMC simulations
    are computed
  repo: https://github.com/ShobiStassen/VIA
- name: dynverse/dyntoy
  role: Provides toy multifurcating/disconnected datasets used to prototype and test
    the trajectory stage
  repo: https://github.com/dynverse/dyntoy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:9b4ee7f48cadc15199ffa0633c7c1a2f40c4a6015e92b449f875c6b951328b1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - VIA
  - dynverse/dyntoy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# mcmc-lineage-refinement

## Summary

Reconstructs VIA's two-stage lazy-teleporting pseudotime and lineage-inference stage on the PARC cluster graph: expected hitting times (Eqs. 1-8) give initial pseudotime/directionality, then lazy-teleporting MCMC simulations on the forward-biased graph refine branch probabilities and terminal states.

## When to use

Use when you need to reproduce or unit-test VIA's pseudotime computation and forward-biased MCMC lineage refinement on a PARC cluster graph (e.g. mESC CyTOF cell-cycle data) with explicit x_lazy/alpha_teleport settings.

## When NOT to use

- You need atlas-scale visualization with metadata integration, higher-order random walks with memory, or Atlas View — those are StaVia (Via 2.0) features, not this Via 1.0 hitting-time + MCMC stage
- The input is raw single-cell data without a PARC cluster graph or a defined root cell — the expected hitting time computation requires a rooted cluster graph
- You need only pseudotime ranking with no lineage/branch-probability output — the 500-simulation-per-terminal-state MCMC stage is then unnecessary overhead

## Inputs

- Single-cell omics data loaded as AnnData with PCA stored in adata.obsm['X_pca'] (e.g. mESC CyTOF cell-cycle data from pyVIA.datasets_via loaders)
- Root/start cell annotation via root_user (e.g. root=[42])
- PARC cluster graph of cells with jaccard-weighted edge pruning

## Outputs

- single_cell_pt_markov: single-cell pseudotime vector from closed-form expected hitting times
- Terminal-state calls at the pseudotime_threshold_TS=30 percentile
- Forward-biased branch probabilities refined by 500 MCMC simulations per terminal state
- pseudotime_validation.csv with Spearman correlations against day labels and full-VIA pseudotimes

## How to apply

Build the PARC cluster graph first (jaccard-weighted pruning, edgepruning_clustering_resolution=0.15, too_big_factor=0.4, small_pop=10, Leiden resolution_parameter=1, random_seed=42). Compute the closed-form expected hitting time of the lazy-teleporting random walk from the root cell over this cluster graph with x_lazy=0.95 and alpha_teleport=0.99 to obtain single-cell pseudotime (single_cell_pt_markov); this first stage fixes graph directionality without MCMC. Flag candidate terminal states at the pseudotime_threshold_TS=30 percentile, then run num_sim_branch_probability=500 lazy-teleporting MCMC simulations per terminal state on the forward-biased graph to refine branch probabilities and lineage structure. The rationale is that hitting times give a theoretical, computationally cheap pseudotime estimate, while MCMC simulations on the biased graph capture multi-branch fate probabilities robustly enough that VIA outperformed SlingShot, Palantir, STREAM and Monocle3 on the mESC CyTOF dataset.

## Related tools

- **VIA** (Implements the lazy-teleporting expected-hitting-time pseudotime and the forward-biased MCMC branch-probability refinement (via.via_wrapper / pyVIA standalone module)) — https://github.com/ShobiStassen/VIA
- **PARC** (Builds the cluster graph on which the random-walk hitting times and MCMC simulations are computed) — https://github.com/ShobiStassen/VIA
- **dynverse/dyntoy** (Provides toy multifurcating/disconnected datasets used to prototype and test the trajectory stage) — https://github.com/dynverse/dyntoy

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=10, jac_std_global=0.15, root=[42], x_lazy=0.95, alpha_teleport=0.99, pseudotime_threshold_TS=30, num_sim_branch_probability=500)
```

## Evaluation signals

- Standalone pseudotimes show strong positive Spearman correlation with annotated day labels on the mESC CyTOF cell-cycle dataset
- Standalone pseudotimes are strongly positively correlated with pseudotimes from the full VIA run (Spearman)
- pseudotime_validation.csv exists and contains both correlation values
- Terminal states are called consistently with the pseudotime_threshold_TS=30 percentile and each receives 500 branch-probability MCMC simulations
- Cluster graph construction matches the specified pruning parameters (edgepruning_clustering_resolution=0.15, too_big_factor=0.4, small_pop=10)

## Limitations

- VIA 1.0's random walks lack memory; StaVia (Via 2.0) extends the lazy-teleporting walks to higher-order random walks with memory for better lineage detection and pathway recovery
- The article reports topology/lineage accuracy results only in supplementary figures — no quantitative accuracy metrics are stated in the discussion
- No software version numbers are stated for VIA or the compared methods in the discussion, complicating exact reproduction
- Pseudotime depends on the choice of root cell (root_user); an arbitrary or unsuitable start cell affects the hitting-time computation

## Evidence

- [abstract] VIA uses lazy-teleporting random walks to compute pseudotime and lineage: "We present VIA, a scalable trajectory inference algorithm that overcomes these limitations by using lazy-teleporting random walks"
- [intro] VIA computes pseudotime and reconstructs lineages via lazy-teleporting random walks with MCMC refinement: "we present VIA, a graph-based TI algorithm that uses a new strategy to compute pseudotime, and reconstruct cell lineages based on lazy-teleporting random walks"
- [discussion] MCMC stage validated on the mESC CyTOF dataset: "We first applied VIA to the mESC CyTOF dataset and showed that the lazy-teleporting MCMCs strategy in VIA enables it to outperform other methods"
- [discussion] Two-part workflow: cluster-graph construction, then pseudotime/lineage via random walks and MCMC: "fast and scalable construction of cluster-graph of cells, followed by pseudotime, and reconstructing cell lineages based on lazy-teleporting random walks and MCMC simulations"
- [readme] Via 2.0 extends the lazy-teleporting walks to higher-order random walks with memory (limitation of 1.0): "StaVia extends the lazy-teleporting walks to higher order random walks with **memory** to allow better lineage detection"
- [methods] Terminal-state percentile filter default: "pseudotime_threshold_TS  (optional, default = 30) Percentile threshold for potential node to qualify as Terminal State"
