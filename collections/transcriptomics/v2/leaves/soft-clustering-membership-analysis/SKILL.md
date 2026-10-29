---
name: soft-clustering-membership-analysis
description: Use when a single-cell expression dataset has been coarse-grained into
  K attractor basins (e.g., K_cluster from Eigen-Peak Index or Louvain/Leiden priors)
  and cells must be assigned fractional membership across basins to distinguish stable
  from transition cells.
when_to_use_negative:
- The dataset has too few cells for high-resolution identification of transition cells
  (e.g., the myelopoiesis dataset, where the article says special care is needed)
- The goal is hard clustering into discrete cell states, where high-accuracy methods
  like SC3 or SIMLR are more appropriate since they enforce distinct states
- Input is raw unprocessed reads rather than a pre-processed expression matrix loadable
  by MuTrans DynamicalAnalysis
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: MuTrans
  role: Performs the multiscale rwTPM learning and soft-membership optimization (DynamicalAnalysis)
    and computes TCS/transition entropy via GeneAnalysis within the Transcendental
    procedure
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: Transcendental
  role: Downstream procedure applied on each transition process to identify transition
    cells from stable cells using the optimized membership function rho*(x)
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Used with MuTrans outputs to compute effective transition-path currents in
    the coarse-grained dynamics that contextualize membership-based transition cells
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:2e19f477301eb46dfe375d24f91c1af7d611ee52d16e7bf0a977725f197c2930
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - MuTrans
  - PyEMMA
  - Transcendental
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# soft-clustering-membership-analysis

## Summary

Optimize and use the soft cell-cluster membership matrix rho*(x) from MuTrans's multiscale rwTPM learning to place each cell continuously within attractor basins, enabling downstream transition cell score (TCS) and transition entropy computation on the cell-fate dynamical manifold.

## When to use

Use when a single-cell expression dataset has been coarse-grained into K attractor basins (e.g., K_cluster from Eigen-Peak Index or Louvain/Leiden priors) and cells must be assigned fractional membership across basins to distinguish stable from transition cells.

## When NOT to use

- The dataset has too few cells for high-resolution identification of transition cells (e.g., the myelopoiesis dataset, where the article says special care is needed)
- The goal is hard clustering into discrete cell states, where high-accuracy methods like SC3 or SIMLR are more appropriate since they enforce distinct states
- Input is raw unprocessed reads rather than a pre-processed expression matrix loadable by MuTrans DynamicalAnalysis

## Inputs

- Pre-processed single-cell gene expression matrix (N_cells x N_genes) from MuTrans-release Data deposit
- Cell-cell scale rwTPM learned via the Gaussian-like affinity kernel (e.g., rwtpm_cell_cell.npz)
- Number of attractors K_cluster (from Eigen-Peak Index eigen-gaps, Louvain/Leiden clustering, or biological prior)
- Coarse-grained attractor basins Sk* and cluster-cluster transition matrix Phat_ij from the coarse-graining optimization

## Outputs

- Optimized soft membership matrix rho* with one column per attractor and rows summing to 1 (membership_matrix.csv)
- Per-cell transition cell score tau_ij(x) for each analyzed state transition i->j (e.g., tcs_saddle_node.csv with cell ID, membership fractions, TCS)
- Per-cell transition entropy H(x) in [0,1] (transition_cell_analysis.csv)
- Attractor basin and transition-layer cell classification based on TCS extremes

## How to apply

After MuTrans learns the cell-cell rwTPM via a Gaussian-like kernel with row normalization p(x,y) = w(x,y)/d(x), initialize the soft membership with indicator functions rho_i^0(x) = 1_{Si*}(x) from the coarse-grained attractor basins. Optimize rho_k by minimizing E[rho_k] = ||ptilde_p[rho_k] - p||^2_mu (Hilbert-Schmidt norm weighted by stationary distribution mu) subject to sum_k rho_k(x) = 1 and rho_k(x) >= 0, using a quasi-Newton method; in practice this is done inside DynamicalAnalysis (pm.dynamical_analysis / Output object) with the chosen K_cluster. From the converged rho*(x), compute per-cell transition entropy H(x) = -sum_k rho_k*(x) log rho_k*(x) and, for each state transition i->j via GeneAnalysis, the transition cell score tau_ij(x) = rho_i*(x)/(rho_i*(x)+rho_j*(x)). Stable-branch cells should show TCS near 0 or 1 while transition-layer cells form a continuum of intermediate values in (0,1), as validated on the saddle-node bifurcation simulation (N=2,000). Save the membership matrix and per-cell TCS/entropy to machine-readable outputs (e.g., membership_matrix.csv, tcs_saddle_node.csv) for validation.

## Related tools

- **MuTrans** (Performs the multiscale rwTPM learning and soft-membership optimization (DynamicalAnalysis) and computes TCS/transition entropy via GeneAnalysis within the Transcendental procedure) — https://github.com/cliffzhou92/MuTrans-release
- **Transcendental** (Downstream procedure applied on each transition process to identify transition cells from stable cells using the optimized membership function rho*(x)) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Used with MuTrans outputs to compute effective transition-path currents in the coarse-grained dynamics that contextualize membership-based transition cells)

## Examples

```
par = {"K_cluster": 5.0}; adata = pm.dynamical_analysis(adata, par)
```

## Evaluation signals

- Membership matrix rows are non-negative and sum to 1 for every cell (one column per attractor K_cluster)
- rwTPM inputs are row-stochastic: all row sums within [1-1e-6, 1+1e-6]
- All per-cell TCS and transition entropy values lie within [0,1]
- In the saddle-node bifurcation simulation (N=2,000), stable-branch cells approach TCS of 1 or 0, and the fraction with TCS < 0.05 or > 0.95 matches the reported Fig. 2a pattern
- Computed TCS values match the tutorial notebook's reference output (example_saddle_node.mlx) within tolerance (output_matches_reference)

## Limitations

- The number of cells must be sufficiently large to obtain high-resolution identification of transition cells; small datasets (e.g., myelopoiesis) require special care and further confirmation
- The continuum limit of the soft membership function in embedded over-damped Langevin dynamical systems remains an open problem
- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- Detailed balance may be violated at the microscopic scale, relying on the coarse-grained dynamics to recover transitions

## Evidence

- [methods] Initialize the soft membership matrix with indicator functions rho_i^0(x) = 1_{Si*}(x) and optimize rho_k by minimizing E[rho_k] = ||ptilde_p[rho_k] - p||^2_mu: [evidence span withheld — anchor status: not_found]
- [methods] we define the transition cell score (TCS) to measure the relative position of cell x in different clusters.: [evidence span withheld — anchor status: not_found]
- [intro] constructed via optimizing a cell-cluster membership matrix: "constructed via optimizing a cell-cluster membership matrix"
- [readme] %% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [intro] the Transition Cell Score (TCS) and transition entropy allows us to distinguish between attractors and transition cells: "the barrier height naturally quantifies the likelihood of cell-fate switch, and the Transition Cell Score (TCS) and transition entropy allows us to distinguish between attractors and transition cells"
