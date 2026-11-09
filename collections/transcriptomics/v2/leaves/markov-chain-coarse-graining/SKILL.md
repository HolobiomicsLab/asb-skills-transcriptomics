---
name: markov-chain-coarse-graining
description: Use when a pre-processed N_cells x N_genes single-cell expression matrix
  must be reduced from cell-level random-walk dynamics to attractor-level (cluster-cluster)
  transition probabilities and soft membership assignments to quantify cell-fate transitions.
when_to_use_negative:
- The dataset has very few cells (e.g. the myelopoiesis dataset) and only coarse attractor
  structure is needed — the article notes special care is required to confirm transition-cell
  analysis when cell numbers are small
- The system is strongly non-stationary due to cell-cycle or proliferation dynamics,
  which MuTrans's stationary, detailed-balance-assuming Markov model does not consider
- The goal is plain hard clustering without transition dynamics — high-accuracy methods
  like SC3 or SIMLR enforce distinct cell states and are a simpler choice when transient/hybrid
  cells are not of interest
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MuTrans
  role: Implements the cell-cell rwTPM construction, coarse-graining optimization
    (DynamicalAnalysis), and soft membership optimization; available as Matlab functions
    or pyMuTrans Python wrapper
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Dependency package for the Python (pyMuTrans) Markov-state-model analysis
    workflow
  repo: https://github.com/cliffzhou92/MuTrans-release
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:b96ff8adb79faa7cff18c590c25d63f3d7baea676d34267973ae20eca5c89d75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - MuTrans
  - PyEMMA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# markov-chain-coarse-graining

## Summary

Coarse-grain a cell-cell scale random walk transition probability matrix (rwTPM) into cluster-cluster dynamics by optimizing attractor basins and inter-attractor transition probabilities, then refining soft cell-cluster memberships, as implemented in MuTrans's multi-scale analysis.

## When to use

Use when a pre-processed N_cells x N_genes single-cell expression matrix must be reduced from cell-level random-walk dynamics to attractor-level (cluster-cluster) transition probabilities and soft membership assignments to quantify cell-fate transitions.

## When NOT to use

- The dataset has very few cells (e.g. the myelopoiesis dataset) and only coarse attractor structure is needed — the article notes special care is required to confirm transition-cell analysis when cell numbers are small
- The system is strongly non-stationary due to cell-cycle or proliferation dynamics, which MuTrans's stationary, detailed-balance-assuming Markov model does not consider
- The goal is plain hard clustering without transition dynamics — high-accuracy methods like SC3 or SIMLR enforce distinct cell states and are a simpler choice when transient/hybrid cells are not of interest

## Inputs

- Pre-processed single-cell gene expression matrix (N_cells x N_genes), e.g. .npz/.h5ad Anndata object from the MuTrans-release Data repository
- User hyperparameter K_cluster (number of attractors), guided by Eigen-Peak Index eigen-gaps or prior clustering/annotation
- Optional DECLARE parameters (reduce_large_scale flag, reduce_num_meta_cell)

## Outputs

- Cell-cell scale rwTPM (row-stochastic transition matrix)
- Cluster-cluster transition matrix Phat (attractor mutual conversion probabilities)
- Attractor basin assignments (hard cluster assignments Sk)
- Soft membership matrix rho* (N_cells x K, rows sum to 1)
- MuTrans Output object from DynamicalAnalysis for downstream InferLineage / ConstructLandscape / GeneAnalysis

## How to apply

First learn the cell-cell rwTPM from expression data via a Gaussian-like affinity kernel, normalizing rows by the degree d(x) = sum_z w(x,z) so p(x,y) = w(x,y)/d(x) with stationary distribution mu(x) = d(x)/sum_z d(z) satisfying detailed balance. Choose the number of attractors K (the pyMuTrans parameter K_cluster, e.g. 5) guided by the Eigen-Peak Index eigen-gaps of the rwTPM, Louvain/Leiden clustering results, or biological prior knowledge. Solve the coarse-graining optimization min J[Sk, Phat_ij] = ||Phat_p - p||^2_mu under the Hilbert-Schmidt norm ||A||^2_mu = sum_{x,y} mu(x)/mu(y) A(x,y)^2 over cluster assignments Sk and cluster-cluster transitions Phat_ij, via an alternating iteration scheme. Initialize the soft membership matrix with indicator functions rho_i^0(x) = 1_{Si*}(x) and optimize rho_k by minimizing E[rho_k] = ||ptilde_p[rho_k] - p||^2_mu subject to sum_k rho_k(x) = 1 and rho_k(x) >= 0 using a quasi-Newton method (Matlab optimization toolbox). For large datasets, optionally enable the DECLARE module (reduce_large_scale=True, reduce_num_meta_cell ~ 1500) to aggregate cells into micro-states before coarse-graining.

## Related tools

- **MuTrans** (Implements the cell-cell rwTPM construction, coarse-graining optimization (DynamicalAnalysis), and soft membership optimization; available as Matlab functions or pyMuTrans Python wrapper) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Dependency package for the Python (pyMuTrans) Markov-state-model analysis workflow) — https://github.com/cliffzhou92/MuTrans-release

## Examples

```
par = {"K_cluster": 5.0}; par["reduce_large_scale"] = True; par["reduce_num_meta_cell"] = 1500; adata = pm.dynamical_analysis(adata, par)
```

## Evaluation signals

- Both cell-cell and cluster-cluster rwTPMs are row-stochastic: all row sums fall within [1-1e-6, 1+1e-6]
- Membership matrix has exactly K columns (one per attractor) with all entries non-negative and each row summing to one
- Chosen K is consistent with Eigen-Peak Index eigen-gaps of the cell-cell rwTPM (or matches Louvain/Leiden / marker-based prior)
- Cluster-cluster matrix Phat is K x K and coarse-grained probabilities reproduce the fine-scale rwTPM in the J[Sk, Phat_ij] objective sense
- The DynamicalAnalysis script (Matlab or pyMuTrans) runs to completion without error and returns attractor basins, coarse-grained transition probabilities, and membership assignment

## Limitations

- Detailed balance may be violated at the microscopic scale (Eq. 1); the method relies on the estimated coarse-grained (mesoscopic) dynamics to recover transitions
- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- A sufficiently large number of cells is required for high-resolution identification of transition cells; small datasets (e.g. myelopoiesis) need special care/confirmation
- The continuum limit of the soft membership function in embedded over-damped Langevin dynamical systems remains an open problem

## Evidence

- [intro] MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel: "MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel"
- [intro] Coarse-graining of cell-cell rwTPM to cluster-cluster scale: "the method performs coarse-graining on the cell–cell scale rwTPM to learn the dynamics on the cluster-cluster scale"
- [readme] DynamicalAnalysis returns attractor basins, coarse-grained transition probabilities and membership assignment: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [discussion] Kramers consistency of coarse-grained transition probability: [evidence span withheld — anchor status: not_found]
- [readme] K_cluster parameter guidance from README: "number of attractors, can plot EPI to give the rough estimate of clusters or refer to other clustering results such as Louvain/Leiden results or from biological prior"
