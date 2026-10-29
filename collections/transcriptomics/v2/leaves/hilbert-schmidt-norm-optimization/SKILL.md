---
name: hilbert-schmidt-norm-optimization
description: Use when you already have a detailed-balance-satisfying cell-cell rwTPM
  from single-cell expression data and need to reduce it to K attractor basins with
  cluster-cluster conversion probabilities and soft memberships.
when_to_use_negative:
- Input is raw or unnormalized counts rather than a pre-processed expression matrix
  from which a detailed-balance rwTPM has been constructed
- The analysis goal is pseudo-time ordering (DPT, Slingshot, Monocle style) rather
  than coarse-grained stochastic dynamics between attractors
- Non-stationary effects such as cell cycle or cell proliferation dynamics dominate
  the system, since these are not considered by the current method
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MuTrans
  role: Implements the multi-scale analysis (DynamicalAnalysis / pyMuTrans dynamical_analysis)
    that performs the rwTPM coarse-graining and membership optimization
  repo: https://github.com/cliffzhou92/MuTrans-release
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:34b10614e12bd02f8b53d7ee10267747f8809e873e6a5304fbeb8c613bb561d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - MuTrans
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# hilbert-schmidt-norm-optimization

## Summary

MuTrans's coarse-graining optimization that projects the cell-cell scale random walk transition probability matrix (rwTPM) onto K attractors by minimizing the mu-weighted Hilbert-Schmidt norm ||Phat_p - p||^2_mu, yielding cluster-cluster transition probabilities and a soft cell-cluster membership matrix.

## When to use

Use when you already have a detailed-balance-satisfying cell-cell rwTPM from single-cell expression data and need to reduce it to K attractor basins with cluster-cluster conversion probabilities and soft memberships.

## When NOT to use

- Input is raw or unnormalized counts rather than a pre-processed expression matrix from which a detailed-balance rwTPM has been constructed
- The analysis goal is pseudo-time ordering (DPT, Slingshot, Monocle style) rather than coarse-grained stochastic dynamics between attractors
- Non-stationary effects such as cell cycle or cell proliferation dynamics dominate the system, since these are not considered by the current method

## Inputs

- Pre-processed single-cell gene expression matrix (N_cells x N_genes)
- Cell-cell scale random walk transition probability matrix (rwTPM) with Gaussian-like kernel affinities w(x,y) and degrees d(x)
- Number of attractors K (hyperparameter, guided by Eigen-Peak Index eigen-gaps or prior annotations)

## Outputs

- Cluster-cluster transition probability matrix Phat (rwtpm_cluster_cluster.csv)
- Attractor basin assignments S_i* for each cell
- Soft membership matrix rho* with one column per attractor (membership_matrix.csv)

## How to apply

First select the number of attractors K (user hyperparameter) using the Eigen-Peak Index eigen-gaps of the cell-cell scale rwTPM or prior marker-gene/Louvain-Leiden annotations. Solve the coarse-graining problem min J[Sk, Phat_ij] = ||Phat_p - p||^2_mu, where the Hilbert-Schmidt norm is weighted by the stationary distribution mu(x) = d(x)/sum_z d(z) with d(x) = sum_z w(x,z), over cluster assignments Sk and cluster-cluster transition probabilities Phat_ij via the alternating iteration scheme. Initialize the soft membership matrix with indicator functions rho_i^0(x) = 1_{Si*}(x), then optimize rho_k by minimizing E[rho_k] = ||ptilde_p[rho_k] - p||^2_mu subject to sum_k rho_k(x) = 1 and rho_k(x) >= 0 using the quasi-Newton method. The rationale is that detailed balance of the cell-cell rwTPM makes the mu-weighted norm the correct metric, so coarse-grained dynamics preserve the Kramers-consistent transition probabilities. Persist Phat (rwtpm_cluster_cluster.csv) and rho* (membership_matrix.csv) alongside the cell-cell rwTPM (rwtpm_cell_cell.npz).

## Related tools

- **MuTrans** (Implements the multi-scale analysis (DynamicalAnalysis / pyMuTrans dynamical_analysis) that performs the rwTPM coarse-graining and membership optimization) — https://github.com/cliffzhou92/MuTrans-release

## Examples

```
par = {"K_cluster": 5.0}; adata = pm.dynamical_analysis(adata,par)
```

## Evaluation signals

- Both the cell-cell rwTPM and the coarse-grained Phat are row-stochastic: all row sums fall within [1-1e-6, 1+1e-6]
- Membership matrix has exactly one column per attractor K, with every row non-negative and summing to one (simplex constraint sum_k rho_k(x) = 1)
- The Hilbert-Schmidt objective J[Sk, Phat_ij] decreases monotonically across the alternating iteration scheme
- The quasi-Newton optimization of E[rho_k] converges without violating rho_k(x) >= 0
- DynamicalAnalysis runs to completion and returns attractor basins, coarse-grained transition probabilities and membership assignment

## Limitations

- Detailed balance may be violated at the microscopic scale (Eq. 1); the method relies on the estimated coarse-grained (mesoscopic) dynamics to recover transitions
- When the number of cells is relatively small, such as in the myelopoiesis dataset, special care is needed to confirm the analysis of transition cells
- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- The continuum limit of the soft membership function in embedded over-damped Langevin dynamical systems remains an open problem

## Evidence

- [intro] MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel: "MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel"
- [intro] Coarse-graining description: "the method performs coarse-graining on the cell–cell scale rwTPM to learn the dynamics on the cluster-cluster scale"
- [intro] Membership optimization: "constructed via optimizing a cell-cluster membership matrix"
- [methods] EPI for choosing K: "We use the Eigen-Peak Index (EPI) to visualize the multiple eigen-gaps of cell-cell scale rwTPM (Supplementary Note 2)."
- [readme] DynamicalAnalysis returns: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [discussion] Microscopic detailed-balance caveat: "although the detailed balance may be violated at the microscopic scale described in Eq. (1) the estimated coarse-grained (mesoscopic) dynamics in MuTrans can be sufficient to recover the transitions"
