---
name: multiscale-coarse-graining-of-stochastic-dynamics
description: Use when single-cell transcriptome data (e.g., an Anndata object or N_cells
  x N_genes expression matrix) must be dissected into meta-stable attractor basins
  versus transient/transition cells via stochastic dynamics rather than hard clustering.
when_to_use_negative:
- When a static clustering/pseudo-time result suffices and transition cells need not
  be separated — methods like SC3, SIMLR, DPT, Slingshot, Monocle enforce distinct
  cell states or orderings without identifying transient cells
- When the dataset has very few cells (e.g., the small myelopoiesis dataset), since
  high-resolution transition-cell identification requires a sufficiently large cell
  number
- When non-stationary effects such as cell cycle or proliferation dynamics dominate,
  which the current method does not consider
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MuTrans
  role: Core method performing rwTPM construction, multi-scale coarse-graining, attractor
    basin detection and membership optimization (DynamicalAnalysis / pm.dynamical_analysis)
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: DECLARE
  role: Optional pre-processing aggregation module that groups cells into micro-states
    (reduce_num_meta_cell, e.g., 1500) to speed up the dynamical analysis
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: Transcendental
  role: GeneAnalysis transition-cell and gene analysis procedure computing TCS tau_ij(x)
    = rho_i*(x)/(rho_i*(x)+rho_j*(x)) between starting state i and targeting state
    j
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: Eigen-Peak Index (EPI)
  role: Diagnostic for choosing K_cluster by visualizing the multiple eigen-gaps of
    the cell-cell scale rwTPM
  repo: https://github.com/cliffzhou92/MuTrans-release
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:6c834e28ea37f7c89145264d7ce5759eb82261d52d7e1cbee35053cbe4e33b47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - DECLARE
  - Eigen-Peak Index (EPI)
  - MuTrans
  - Transcendental
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# multiscale-coarse-graining-of-stochastic-dynamics

## Summary

Multi-scale reduction of the cell-cell scale random-walk transition probability matrix (rwTPM) as implemented in MuTrans, which coarse-grains stochastic dynamics to recover attractor basins, coarse-grained transition probabilities, and per-cell soft membership rho*(x) on a cell-fate dynamical manifold.

## When to use

Use when single-cell transcriptome data (e.g., an Anndata object or N_cells x N_genes expression matrix) must be dissected into meta-stable attractor basins versus transient/transition cells via stochastic dynamics rather than hard clustering.

## When NOT to use

- When a static clustering/pseudo-time result suffices and transition cells need not be separated — methods like SC3, SIMLR, DPT, Slingshot, Monocle enforce distinct cell states or orderings without identifying transient cells
- When the dataset has very few cells (e.g., the small myelopoiesis dataset), since high-resolution transition-cell identification requires a sufficiently large cell number
- When non-stationary effects such as cell cycle or proliferation dynamics dominate, which the current method does not consider

## Inputs

- Pre-processed single-cell gene expression matrix (N_cells x N_genes) or Scanpy Anndata object
- Number of attractors K_cluster (from EPI, Louvain/Leiden clustering, or biological prior)
- Optional MuTrans-release processed data deposit files (e.g., saddle-node bifurcation Euler-Maruyama simulation, N=2,000 cells)

## Outputs

- Attractor basins with meta-stable cell states
- Coarse-grained cluster-cluster transition probability matrix
- Per-cell soft membership assignment rho*(x)
- Transition cell score (TCS) values per cell for state transitions i->j
- Persisted per-cell table (e.g., tcs_saddle_node.csv with cell ID, membership fractions, TCS)

## How to apply

First learn the rwTPM from the pre-processed expression matrix via a Gaussian-like kernel, then run the multi-scale analysis (Matlab DynamicalAnalysis(data,par) or Python pm.dynamical_analysis(adata,par)) setting par['K_cluster'] to the number of attractors, estimated via the Eigen-Peak Index (EPI), Louvain/Leiden clustering, or biological prior. The coarse-graining returns attractor basins, cluster-cluster transition probabilities, and the optimized cell-cluster membership matrix that assigns each cell a relative position within basins. Optionally enable the DECLARE module (reduce_large_scale=True, reduce_num_meta_cell e.g. 1500) to aggregate cells into micro-states for scalability on large datasets. Downstream, use the soft membership rho*(x) with the Transcendental/GeneAnalysis procedure to compute the transition cell score tau_ij(x) = rho_i*(x)/(rho_i*(x)+rho_j*(x)), so stable cells approach TCS 0 or 1 and transition-layer cells take intermediate values in (0,1).

## Related tools

- **MuTrans** (Core method performing rwTPM construction, multi-scale coarse-graining, attractor basin detection and membership optimization (DynamicalAnalysis / pm.dynamical_analysis)) — https://github.com/cliffzhou92/MuTrans-release
- **DECLARE** (Optional pre-processing aggregation module that groups cells into micro-states (reduce_num_meta_cell, e.g., 1500) to speed up the dynamical analysis) — https://github.com/cliffzhou92/MuTrans-release
- **Transcendental** (GeneAnalysis transition-cell and gene analysis procedure computing TCS tau_ij(x) = rho_i*(x)/(rho_i*(x)+rho_j*(x)) between starting state i and targeting state j) — https://github.com/cliffzhou92/MuTrans-release
- **Eigen-Peak Index (EPI)** (Diagnostic for choosing K_cluster by visualizing the multiple eigen-gaps of the cell-cell scale rwTPM) — https://github.com/cliffzhou92/MuTrans-release

## Examples

```
import pyMuTrans as pm; par = {"K_cluster": 2.0}; par["reduce_large_scale"] = True; par["reduce_num_meta_cell"] = 1500; adata = pm.dynamical_analysis(adata, par)
```

## Evaluation signals

- Stable-branch cells receive TCS approaching one or zero, while transition-layer cells form a continuum of intermediate values in (0,1) (sharp-transition ordering as in Fig. 2a)
- Fraction of stable cells with TCS < 0.05 or > 0.95 is consistent with the reported saddle-node bifurcation pattern
- Computed per-cell TCS values match the example_saddle_node.mlx reference output within tolerance (output_matches_reference)
- K_cluster choice is justified by EPI eigen-gaps or an independent Louvain/Leiden clustering result
- DynamicalAnalysis output contains all three expected artifacts: attractor basins, coarse-grained transition probabilities, and membership assignment

## Limitations

- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- A sufficiently large number of cells is required for high-resolution identification of transition cells; small datasets (e.g., myelopoiesis) need special care
- Detailed balance may be violated at the microscopic scale (Eq. 1); recovery of transitions relies on the coarse-grained mesoscopic dynamics
- The continuum limit of the soft membership function in embedded over-damped Langevin systems remains an open problem
- Matlab code requires R2019b/R2020a with optimization toolbox; Transcendental gene display additionally needs the Curve Fitting toolbox 'smooth' function; Python needs matlab engine, PyEMMA >=2.5.6, Scanpy, Numpy, Pandas, Seaborn

## Evidence

- [intro] MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel: "MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel"
- [methods] we define the transition cell score (TCS) to measure the relative position of cell x in different clusters.: [evidence span withheld — anchor status: not_found]
- [readme] Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [readme] par = {"K_cluster": 5.0} # number of attractors, can plot EPI to give the rough estimate of clusters: "par = {"K_cluster": 5.0} # number of attractors, can plot EPI to give the rough estimate of clusters or refer to other clustering results such as Louvain/Leiden results or from biological prior."
- [intro] the barrier height naturally quantifies the likelihood of cell-fate switch, and the Transition Cell Score (TCS) and transition entropy allows us to distinguish between attractors and transition cells: "the barrier height naturally quantifies the likelihood of cell-fate switch, and the Transition Cell Score (TCS) and transition entropy allows us to distinguish between attractors and transition cells"
