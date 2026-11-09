---
name: single-cell-transcriptome-dynamics-modeling
description: Use when the goal is to infer attractor basins, cell-fate transition
  paths, and transition cells from single-cell transcriptome data by modeling cell-state
  dynamics as a Markov random walk across cell-cell and cluster-cluster scales.
when_to_use_negative:
- 'Datasets with too few cells: the article notes that in small datasets (e.g., the
  myelopoiesis dataset) special care is needed to confirm transition-cell analyses,
  so high-resolution identification of transition cells is unreliable'
- Systems where non-stationary effects from cell cycle or cell proliferation dynamics
  dominate — these were not considered in the current method
- 'The input is raw sequencing data: MuTrans expects a pre-processed gene expression
  matrix (e.g. from DECLARE-ready or standard preprocessing), not FASTQ/count alignment'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: MuTrans
  role: 'Core method: runs the multi-scale rwTPM analysis (DynamicalAnalysis/pyMuTrans.dynamical_analysis)
    returning attractor basins, coarse-grained transition probabilities, and membership
    assignment'
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: DECLARE
  role: Optional pre-processing module that aggregates cells into micro-states to
    speed up the multi-scale calculation on large datasets (par['reduce_large_scale']=True)
  repo: https://github.com/cliffzhou92/MuTrans-release
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:f77b7ed38987e82de5ce6feb490f335f71a4a3808179d717776e791b4f7cc798
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - DECLARE
  - MuTrans
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-transcriptome-dynamics-modeling

## Summary

Reconstructs cell-fate stochastic dynamics from a pre-processed single-cell gene expression matrix (N_cells x N_genes) using MuTrans's iterative multi-scale learning: a cell-cell scale random walk transition probability matrix (rwTPM), coarse-grained cluster-cluster transition probabilities, and a soft cell-cluster membership matrix that separate attractor basins from transition cells.

## When to use

Use when the goal is to infer attractor basins, cell-fate transition paths, and transition cells from single-cell transcriptome data by modeling cell-state dynamics as a Markov random walk across cell-cell and cluster-cluster scales.

## When NOT to use

- Datasets with too few cells: the article notes that in small datasets (e.g., the myelopoiesis dataset) special care is needed to confirm transition-cell analyses, so high-resolution identification of transition cells is unreliable
- Systems where non-stationary effects from cell cycle or cell proliferation dynamics dominate — these were not considered in the current method
- The input is raw sequencing data: MuTrans expects a pre-processed gene expression matrix (e.g. from DECLARE-ready or standard preprocessing), not FASTQ/count alignment

## Inputs

- Pre-processed single-cell gene expression matrix (N_cells x N_genes), e.g. the processed matrices stored in github.com/cliffzhou92/MuTrans-release/tree/main/Data
- Number of attractors K (user hyperparameter from Eigen-Peak Index or biological prior)
- Anndata object (Python/pyMuTrans path) or Matlab data/par structure

## Outputs

- Cell-cell scale rwTPM (e.g. rwtpm_cell_cell.npz), row-stochastic
- Cluster-cluster transition matrix Phat (e.g. rwtpm_cluster_cluster.csv) with attractor mutual conversion probabilities
- Soft membership matrix rho* (e.g. membership_matrix.csv), one column per attractor, rows non-negative summing to one
- Attractor basins, transition cell scores (TCS), transition entropy, and lineage trees (MPPT/MPFT)

## How to apply

Load the processed expression matrix (N_cells x N_genes) and compute the cell-cell rwTPM via a Gaussian-like affinity kernel, normalizing rows by the degree d(x) so p(x,y)=w(x,y)/d(x) satisfies detailed balance with stationary distribution mu(x)=d(x)/sum_z d(z). Choose the number of attractors K (e.g., par['K_cluster']=5 in pyMuTrans), guided by the Eigen-Peak Index eigen-gaps of the cell-cell rwTPM or prior clustering/biological knowledge. Solve the coarse-graining optimization min J[Sk, Phat_ij] = ||Phat_p - p||^2_mu over cluster assignments and cluster-cluster transition probabilities (Hilbert-Schmidt norm weighted by mu), then initialize the soft membership matrix with indicator functions rho_i^0(x)=1_{Si*}(x) and optimize rho_k by minimizing E[rho_k]=||ptilde_p[rho_k]-p||^2_mu subject to sum_k rho_k(x)=1 and rho_k(x)>=0 via quasi-Newton iteration. Optionally enable the DECLARE aggregation module (par['reduce_large_scale']=True, e.g. reduce_num_meta_cell=1500) for large datasets. Downstream, use transition path theory (MPPT or MPFT lineage trees), transition cell score (TCS) and transition entropy to distinguish stable from transient cells.

## Related tools

- **MuTrans** (Core method: runs the multi-scale rwTPM analysis (DynamicalAnalysis/pyMuTrans.dynamical_analysis) returning attractor basins, coarse-grained transition probabilities, and membership assignment) — https://github.com/cliffzhou92/MuTrans-release
- **DECLARE** (Optional pre-processing module that aggregates cells into micro-states to speed up the multi-scale calculation on large datasets (par['reduce_large_scale']=True)) — https://github.com/cliffzhou92/MuTrans-release

## Examples

```
import pyMuTrans as pm; par = {"K_cluster": 5.0}; par["reduce_large_scale"] = True; par["reduce_num_meta_cell"] = 1500; adata = pm.dynamical_analysis(adata, par); pm.infer_lineage(adata, si=2, sf=0, method="MPPT")
```

## Evaluation signals

- Each rwTPM is row-stochastic: verify all row sums fall within [1-1e-6, 1+1e-6]
- Membership matrix validation: one column per attractor K, all entries non-negative, and each row sums to 1
- Stationary distribution consistency: mu(x)=d(x)/sum_z d(z) satisfies detailed balance for the cell-cell rwTPM built from the Gaussian-like kernel
- Pipeline completes without error through DynamicalAnalysis -> InferLineage -> ConstructLandscape -> GeneAnalysis
- Number of identified attractor basins matches the chosen K_cluster hyperparameter

## Limitations

- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- A sufficiently large number of cells is required for high-resolution identification of transition cells; small datasets (e.g., myelopoiesis) need special care and further confirmation
- Detailed balance may be violated at the microscopic scale (Eq. 1); transitions rely on the estimated coarse-grained (mesoscopic) dynamics
- Continuum limit of the soft membership function in embedded over-damped Langevin systems remains an open problem; root-cell detection could be enhanced via entropy methods or RNA velocity (not currently integrated)

## Evidence

- [intro] MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel: "MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel"
- [intro] Coarse-graining and membership optimization: "the method performs coarse-graining on the cell–cell scale rwTPM to learn the dynamics on the cluster-cluster scale"
- [methods] EPI for choosing K: "We use the Eigen-Peak Index (EPI) to visualize the multiple eigen-gaps of cell-cell scale rwTPM (Supplementary Note 2)."
- [methods] DynamicalAnalysis returns: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [discussion] Small dataset caveat: "When the number of cells is relatively small, such as in the myelopoiesis dataset studied here, special care is needed"
- [readme] K_cluster parameter usage: "par = {"K_cluster": 5.0} # number of attractors, can plot EPI to give the rough estimate of clusters or refer to other clustering results"
- [abstract] MuTrans core capability: "By iteratively unifying transition dynamics across multiple scales, MuTrans constructs the cell-fate dynamical manifold that depicts progression of cell-state transitions"
