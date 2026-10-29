---
name: random-walk-transition-probability-matrix-coarse-graining
description: Use when single-cell snapshot transcriptome data must be decomposed into
  stable attractor basins versus transient/transition cells, e.g. Pro-B to Pre-B differentiation,
  EMT, or Langevin simulation wells, with basin count guided by the EPI eigen-gap.
when_to_use_negative:
- Datasets where cell counts are too small for high-resolution transition-cell identification
  (e.g. the myelopoiesis Fluidigm C1 data required special care per the discussion)
- Systems where non-stationary effects from cell cycle or proliferation dominate and
  must be explicitly modeled — MuTrans does not account for these
- The goal is static clustering with hard state assignments (e.g. SC3/SIMLR-style),
  which enforce distinct states and erase transient cells
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MuTrans (pyMuTrans)
  role: Implements rwTPM construction via Gaussian-like kernel, EPI eigen-gap inspection,
    multi-scale coarse-graining, and membership optimization via pm.dynamical_analysis
  repo: github.com/cliffzhou92/MuTrans-release
- name: Scanpy
  role: Loads processed gene expression matrices into AnnData objects that are passed
    to pm.dynamical_analysis
- name: PyEMMA
  role: Dependency used for transition-path-theory quantification of transition trajectory
    likelihoods on the coarse-grained dynamics
- name: DECLARE
  role: Optional dynamics-preserving pre-processing aggregation module (reduce_large_scale=True)
    that clusters cells into ~1500 micro-states to speed up rwTPM computation on large
    datasets
  repo: github.com/cliffzhou92/MuTrans-release
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:9252adc8201d228e782605aa3a327a36a5161d1ef249d7e677b84cf863d828eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DECLARE
  - MuTrans
  - MuTrans (pyMuTrans)
  - NumPy
  - Pandas
  - PyEMMA
  - Scanpy
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# random-walk-transition-probability-matrix-coarse-graining

## Summary

MuTrans core multi-scale procedure that learns a cell-cell random walk transition probability matrix (rwTPM) via a Gaussian-like affinity kernel, coarse-grains it to cluster-cluster scale dynamics to recover attractor basins and inter-basin transition probabilities, and optimizes a soft cell-cluster membership matrix.

## When to use

Use when single-cell snapshot transcriptome data must be decomposed into stable attractor basins versus transient/transition cells, e.g. Pro-B to Pre-B differentiation, EMT, or Langevin simulation wells, with basin count guided by the EPI eigen-gap.

## When NOT to use

- Datasets where cell counts are too small for high-resolution transition-cell identification (e.g. the myelopoiesis Fluidigm C1 data required special care per the discussion)
- Systems where non-stationary effects from cell cycle or proliferation dominate and must be explicitly modeled — MuTrans does not account for these
- The goal is static clustering with hard state assignments (e.g. SC3/SIMLR-style), which enforce distinct states and erase transient cells

## Inputs

- Pre-processed single-cell gene expression matrix (N_cells x N_genes), e.g. AnnData (.h5ad) built with Scanpy
- Processed matrices from MuTrans-release/Data (e.g. Herman et al. Cel-Seq2 example_mpp, Olsson et al. Fluidigm C1, 10x Chromium human bone marrow, triple-well simulation snapshots)
- Parameter dict par (K_cluster, optional reduce_large_scale=True and reduce_num_meta_cell=1500)

## Outputs

- Attractor-basin assignments per cell (membership matrix rho*)
- Coarse-grained K x K cell-cluster rwTPM (e.g. 3x3 for the triple-well)
- Per-cell transition entropy H(x) and transition cell scores (TCS) tau_ij
- MuTrans-annotated AnnData from pm.dynamical_analysis for downstream lineage/manifold steps

## How to apply

Load the pre-processed gene expression matrix (N_cells x N_genes, e.g. AnnData via Scanpy for Cel-Seq2 Herman et al. or 10x Chromium data, or the MuTrans-release Data folder) and run pm.dynamical_analysis(adata, par). First choose K_cluster by inspecting the Eigen-Peak Index (EPI) eigen-gaps of the rwTPM (e.g. K=3 for the triple-well simulation with N=2,001 Euler-Maruyama snapshots, or annotated lineages LP/Pro-B/Pre-B for the hematopoietic progenitor data). DynamicalAnalysis iteratively optimizes attractor basins S_k and the K x K coarse-grained rwTPM (e.g. 3x3 for triple-well), returning basin assignments and the optimized membership rho*(x); for large datasets enable the DECLARE module (par['reduce_large_scale']=True, par['reduce_num_meta_cell']=1500) to aggregate cells into ~1500 micro-states first. Rationale: coarse-grained transition probability is consistent with Kramers' reaction rate theory for over-damped Langevin dynamics under steady-state and detailed-balance conditions. Downstream, compute per-cell transition entropy H(x) = -sum_k rho*_k(x) log rho*_k(x) and the transition cell score (TCS) tau_ij = rho_i*/(rho_i* + rho_j*) via GeneAnalysis to separate stable from transient cells.

## Related tools

- **MuTrans (pyMuTrans)** (Implements rwTPM construction via Gaussian-like kernel, EPI eigen-gap inspection, multi-scale coarse-graining, and membership optimization via pm.dynamical_analysis) — github.com/cliffzhou92/MuTrans-release
- **Scanpy** (Loads processed gene expression matrices into AnnData objects that are passed to pm.dynamical_analysis)
- **PyEMMA** (Dependency used for transition-path-theory quantification of transition trajectory likelihoods on the coarse-grained dynamics)
- **DECLARE** (Optional dynamics-preserving pre-processing aggregation module (reduce_large_scale=True) that clusters cells into ~1500 micro-states to speed up rwTPM computation on large datasets) — github.com/cliffzhou92/MuTrans-release

## Examples

```
import pyMuTrans as pm; par = {"K_cluster": 3.0}; par["reduce_large_scale"]=True; par["reduce_num_meta_cell"]=1500; adata = pm.dynamical_analysis(adata, par)
```

## Evaluation signals

- Number of inferred attractor basins equals the K selected from the EPI eigen-gap (e.g. exactly 3 for the triple-well simulation)
- Coarse-grained rwTPM has shape K x K and is row-stochastic (e.g. 3x3 for K_cluster=3)
- Transition entropy is markedly higher for cells near saddle/maximum points between wells and near zero for stable basin cells
- Basin assignments are consistent with prior cell-type annotations (e.g. Pro-B and Pre-B basins matching Supplementary labels in the Herman et al. Cel-Seq2 data)
- With DECLARE enabled, transition path likelihoods and landscape structure agree with the complete-data run, and speedup is at least 5x (reported roughly one order of magnitude)

## Limitations

- Sufficiently large cell numbers are required for high-resolution identification of transition cells
- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the method
- Detailed balance may be violated at the microscopic scale; only the estimated coarse-grained mesoscopic dynamics can be relied upon to recover transitions
- For small datasets such as the myelopoiesis data, special care is needed to further confirm transition-cell analysis

## Evidence

- [intro] MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel: "MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel"
- [intro] the method performs coarse-graining on the cell–cell scale rwTPM to learn the dynamics on the cluster-cluster scale: "the method performs coarse-graining on the cell–cell scale rwTPM to learn the dynamics on the cluster-cluster scale"
- [readme] Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [discussion] The transition probability is shown to be consistent with Kramers’ reaction rate theory for over-damped Langevin dynamics: "The transition probability is shown to be consistent with Kramers’ reaction rate theory for over-damped Langevin dynamics if steady-state assumption and detailed-balanced condition are satisfied"
- [methods] We use the Eigen-Peak Index (EPI) to visualize the multiple eigen-gaps of cell-cell scale rwTPM: "We use the Eigen-Peak Index (EPI) to visualize the multiple eigen-gaps of cell-cell scale rwTPM (Supplementary Note 2)."
