---
name: transition-entropy-cell-state-bifurcation-analysis
description: Use when a single-cell trajectory dataset (e.g. time-course iPSC induction
  data) is suspected to contain a cell-fate bifurcation and you need to localize the
  critical transition point via attractor-basin membership entropy.
when_to_use_negative:
- Datasets with relatively few cells (e.g. the myelopoiesis Fluidigm C1 dataset),
  where the article notes special care is needed and high-resolution identification
  of transition cells requires a sufficiently large number of cells
- When the goal is stable-state clustering rather than transient/bifurcation dynamics
  — high-accuracy methods like SC3 or SIMLR tend to enforce distinct cell states and
  will not capture transition entropy
- Systems where non-stationary cell-cycle or cell-proliferation dynamics dominate,
  since these effects were not considered in the current method
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: MuTrans / pyMuTrans
  role: Constructs the cell-cell rwTPM, coarse-grains it into attractor basins, optimizes
    membership rho_k(x), and provides the dynamical_analysis and infer_lineage (MPPT)
    entry points used to compute transition entropy
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Dependency package required for pyMuTrans installation (>=2.5.6)
- name: Scanpy
  role: Dependency package providing the Anndata object that pm.dynamical_analysis
    operates on
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:5cdaa36ff0b94b59d7077025759909c22c586f811a53bd656f87b5c914e114d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - MuTrans
  - MuTrans / pyMuTrans
  - PyEMMA
  - Scanpy
  - pyMuTrans
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# transition-entropy-cell-state-bifurcation-analysis

## Summary

Quantifies per-cell transition entropy H(x) = -sum_k rho_k*(x) log rho_k*(x) from the MuTrans-optimized attractor membership function to detect bifurcation points in cell-fate dynamics, e.g. peaking near the critical transition during iPSC induction around day 2.5 before fates commit at day 3.

## When to use

Use when a single-cell trajectory dataset (e.g. time-course iPSC induction data) is suspected to contain a cell-fate bifurcation and you need to localize the critical transition point via attractor-basin membership entropy.

## When NOT to use

- Datasets with relatively few cells (e.g. the myelopoiesis Fluidigm C1 dataset), where the article notes special care is needed and high-resolution identification of transition cells requires a sufficiently large number of cells
- When the goal is stable-state clustering rather than transient/bifurcation dynamics — high-accuracy methods like SC3 or SIMLR tend to enforce distinct cell states and will not capture transition entropy
- Systems where non-stationary cell-cycle or cell-proliferation dynamics dominate, since these effects were not considered in the current method

## Inputs

- Processed N_cells x N_genes single-cell gene expression matrix (e.g. iPSC induction single-cell RT-qPCR data from MuTrans-release/Data)
- Anndata object or expression matrix loadable by pyMuTrans
- K_cluster parameter (number of attractors), guided by the Eigen-Peak Index (EPI) of the cell-cell rwTPM
- Per-cell capture time-point/day metadata for grouping entropy values

## Outputs

- Per-cell transition entropy H(x) = -sum_k rho_k*(x) log rho_k*(x) from the optimized membership function
- Attractor-basin assignment for each cell
- Mean-entropy-by-day summary table (e.g. written to ipsc_mutrans_attractor_entropy_results.csv)
- Optimized cell-cluster membership function rho_k(x) and coarse-grained attractor transition probabilities

## How to apply

Load the processed gene expression matrix (e.g. iPSC induction single-cell RT-qPCR data from MuTrans-release/Data) into pyMuTrans and choose K_cluster from the Eigen-Peak Index (EPI) of the cell-cell rwTPM — 9 attractors for the iPSC dataset. Run pm.dynamical_analysis to build the cell-cell random walk transition probability matrix via a Gaussian-like kernel, coarse-grain it into K attractor basins, and optimize the cell-cluster membership function rho_k(x) by quasi-Newton minimization of the cell-cluster rwTPM distance to the original. Compute per-cell transition entropy H(x) = -sum_k rho_k*(x) log rho_k*(x), group entropy values by capture time point (day), and average within each day. A valid bifurcation signature is an increase-then-decrease entropy trend peaking at the bifurcation point (around day 2.5 for iPSC), consistent with the critical transition index, followed by decreases as final fates commit (day 3). Optionally run pm.infer_lineage with method='MPPT' from the starting to targeting attractor to place the bifurcation on the dynamical manifold.

## Related tools

- **MuTrans / pyMuTrans** (Constructs the cell-cell rwTPM, coarse-grains it into attractor basins, optimizes membership rho_k(x), and provides the dynamical_analysis and infer_lineage (MPPT) entry points used to compute transition entropy) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Dependency package required for pyMuTrans installation (>=2.5.6))
- **Scanpy** (Dependency package providing the Anndata object that pm.dynamical_analysis operates on)

## Examples

```
import pyMuTrans as pm; par = {"K_cluster": 9.0}; adata = pm.dynamical_analysis(adata, par); pm.infer_lineage(adata, si=2, sf=0, method="MPPT")
```

## Evaluation signals

- Recovered attractor count equals the expected K_cluster (9 for the iPSC dataset), validated against the EPI of the rwTPM
- Entropy-vs-day summary exhibits an increase-then-decrease trend peaking around day 2.5, matching the reported critical transition index
- Membership optimization converges via quasi-Newton minimization of the cell-cluster rwTPM distance to the original rwTPM
- Mean entropy decreases again by day 3 as final cell-fates are committed and established
- Output CSV contains per-cell attractor-basin assignments plus the mean-entropy-by-day table with no missing day groups

## Limitations

- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- The number of cells needs to be sufficiently large for high-resolution identification of transition cells; small datasets (e.g. myelopoiesis) need special care
- The continuum limit of the soft membership function in embedded over-damped Langevin dynamical systems remains an open problem
- Detailed balance may be violated at the microscopic scale; recovery of transitions relies on the estimated coarse-grained (mesoscopic) dynamics

## Evidence

- [intro] transition entropy distinguishes attractors: "the barrier height naturally quantifies the likelihood of cell-fate switch, and the Transition Cell Score (TCS) and transition entropy allows us to distinguish between attractors and transition cells"
- [methods] coarse-graining membership optimization: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [methods] MPPT lineage inference: "we can infer the cell lineage from the Most Probable Path Tree (MPPT) approach or Maximum Probability Flow Tree (MPFT) approach"
- [methods] EPI for cluster number: "We use the Eigen-Peak Index (EPI) to visualize the multiple eigen-gaps of cell-cell scale rwTPM (Supplementary Note 2)."
- [readme] readme python usage: "par = {"K_cluster": 5.0} # number of attractors, can plot EPI to give the rough estimate of clusters"
- [discussion] small dataset caveat: "When the number of cells is relatively small, such as in the myelopoiesis dataset studied here, special care is needed to further confirm the analysis of transition cells."
