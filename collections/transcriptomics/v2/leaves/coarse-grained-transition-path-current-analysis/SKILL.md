---
name: coarse-grained-transition-path-current-analysis
description: Use when MuTrans DynamicalAnalysis has identified attractor basins Sk*
  and a coarse-grained transition matrix P̂ij, and you need to rank discrete transition
  routes between two cell states by likelihood.
when_to_use_negative:
- Datasets with relatively few cells (e.g., the Olsson myelopoiesis dataset) where
  transition-cell resolution is low and results need special independent confirmation
- Systems with strong non-stationary effects (cell cycle or proliferation dynamics)
  that the current Markov random-walk treatment does not model
- Questions answerable by static dimension-reduction or plain clustering (SC3, SIMLR)
  — those methods enforce distinct states and cannot quantify transition-path likelihoods
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: MuTrans
  role: Runs DynamicalAnalysis to produce the attractor basins, coarse-grained transition
    probabilities, and membership matrix on which transition path theory is applied;
    pyMuTrans exposes infer_lineage to plot the dominant transition trajectory
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Provides the transition-path-theory functions used to compute effective transition-path
    currents fij+ between coarse-grained attractor states
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:32efa97fa4e80de7a9993a5a1ee738af831a36f406d6c7d9835f8152bb1e8faa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - MuTrans
  - PyEMMA
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# coarse-grained-transition-path-current-analysis

## Summary

Quantifies transition trajectories between MuTrans attractor basins by computing effective transition-path currents with PyEMMA's transition path theory functions on the coarse-grained transition matrix, ranking paths by their capacity proportions. This identifies the most probable paths (analogous to saddles) connecting meta-stable cell states.

## When to use

Use when MuTrans DynamicalAnalysis has identified attractor basins Sk* and a coarse-grained transition matrix P̂ij, and you need to rank discrete transition routes between two cell states by likelihood.

## When NOT to use

- Datasets with relatively few cells (e.g., the Olsson myelopoiesis dataset) where transition-cell resolution is low and results need special independent confirmation
- Systems with strong non-stationary effects (cell cycle or proliferation dynamics) that the current Markov random-walk treatment does not model
- Questions answerable by static dimension-reduction or plain clustering (SC3, SIMLR) — those methods enforce distinct states and cannot quantify transition-path likelihoods

## Inputs

- Coarse-grained (cluster-cluster) transition probability matrix P̂ij from MuTrans DynamicalAnalysis
- Attractor basin labels Sk* and optimized membership function ρ*(x) per cell
- Pre-processed N_cells x N_genes single-cell gene expression matrix (AnnData object for pyMuTrans)

## Outputs

- Effective transition-path currents fij+ between attractor pairs
- Ranked list of transition trajectories with capacity proportions c(wdr) = min fij+ along each path
- Dominant (most probable) path(s) between states i and j for lineage plotting and Transcendental transition-cell analysis

## How to apply

After running MuTrans's DynamicalAnalysis (which returns attractor basins, coarse-grained transition probabilities, and membership assignment), select a source attractor Si* and target attractor Sj*. Use PyEMMA (>=2.5.6) transition path theory functions on the coarse-grained transition matrix to compute effective forward transition-path currents fij+. Define the capacity c(wdr) of each discrete route as the minimum fij+ along the path (bottleneck current), then rank transition trajectories by their capacity proportions. Feed the top-ranked paths into the Transcendental downstream procedure to place transition cells along them and to compute the transition cell score τij(x) = ρi*(x)/(ρi*(x)+ρj*(x)) for cells near each path. Optionally visualize the dominant route on the dynamical manifold via pm.infer_lineage using MPPT or MPFT.

## Related tools

- **MuTrans** (Runs DynamicalAnalysis to produce the attractor basins, coarse-grained transition probabilities, and membership matrix on which transition path theory is applied; pyMuTrans exposes infer_lineage to plot the dominant transition trajectory) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Provides the transition-path-theory functions used to compute effective transition-path currents fij+ between coarse-grained attractor states)

## Examples

```
import pyMuTrans as pm; adata = pm.dynamical_analysis(adata, {"K_cluster": 5.0}); pm.infer_lineage(adata, si=2, sf=0, method="MPPT")
```

## Evaluation signals

- The coarse-grained transition probability between attractors should be consistent with Kramers' reaction rate theory for over-damped Langevin dynamics under steady-state and detailed-balance conditions
- Ranked path capacities c(wdr) (minimum fij+ along each route) should sum to a normalized set of capacity proportions, with the dominant path stable to perturbation of K_cluster
- Dominant paths should coincide with the transition cells placed along them by the Transcendental procedure (high TCS and transition entropy cells)
- Cross-check the inferred lineage by comparing MPPT and MPFT trees from pm.infer_lineage for the same si/sf attractor pair
- Sanity-check against simulation benchmarks (saddle-node, double-well, triple-well potentials) where the true transition paths are known

## Limitations

- Detailed balance may be violated at the microscopic (cell-cell rwTPM) scale; recovery of transitions relies on the estimated coarse-grained mesoscopic dynamics
- Sufficiently large cell numbers are required for high-resolution identification of transition cells; small datasets (e.g., myelopoiesis) need special care and confirmation
- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method

## Evidence

- [abstract] MuTrans quantifies the likelihood of all possible transition trajectories between cell states using coarse-grained transition path theory.: "MuTrans quantifies the likelihood of all possible transition trajectories between cell states using coarse-grained transition path theory."
- [methods] TPT applied after multi-scale analysis returns attractor basins and coarse-grained transition probabilities: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [results] compute effective transition-path currents with PyEMMA functions and rank paths by capacity: [evidence span withheld — anchor status: not_found]
- [discussion] Transition probability consistent with Kramers' theory under steady-state and detailed balance: "The transition probability is shown to be consistent with Kramers’ reaction rate theory for over-damped Langevin dynamics if steady-state assumption and detailed-balanced condition are satisfied"
- [readme] README shows infer_lineage plotting of transition trajectory between states on the manifold: "pm.infer_lineage(adata,si=2,sf=0,method = "MPPT",size_point =40, size_text = 10,alpha_point = 0.5) # plot the transition trajectory on dynamical manifold"
