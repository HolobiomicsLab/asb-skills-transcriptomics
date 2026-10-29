---
name: transition-entropy-quantification-from-membership
description: Use when a MuTrans DynamicalAnalysis output (attractor basins Sk* and
  optimized membership ρ*(x)) is available and you need to score each cell's transition
  propensity, e.g. to locate saddle points between attractors in a cell-fate dynamical
  manifold.
when_to_use_negative:
- You only need static clustering labels (e.g. Louvain/Leiden) without any notion
  of transition dynamics or transient cells
- The dataset has too few cells to resolve transition cells at high resolution (e.g.
  small myelopoiesis datasets, which the authors flag as needing special care)
- No membership/attractor-basin decomposition has been inferred yet — run DynamicalAnalysis
  first; the entropy cannot be computed directly from raw expression
tools:
- name: MuTrans
  role: Runs DynamicalAnalysis (multi-scale coarse-graining) that returns the attractor
    basins and optimized membership function ρ*(x) from which the transition entropy
    is computed; Transcendental performs downstream transition-cell analysis
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Dependency used in the Python MuTrans workflow (pyMuTrans) supporting the
    random-walk/coarse-grained dynamics analysis feeding the membership computation
  repo: http://emma-project.org/latest/INSTALL.html
- name: Scanpy
  role: Python dependency used to handle the AnnData object passed through pm.dynamical_analysis
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:5fc079ac3a8fe2829d5e98eff359aaa52d699ffd268a61e384470c4d2dab385c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - MuTrans
  - NumPy
  - Pandas
  - PyEMMA
  - Scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# transition-entropy-quantification-from-membership

## Summary

Computes MuTrans's per-cell transition entropy H(x) = −Σ_k ρk*(x) log ρk*(x) from the optimized cell-cluster membership function ρ*(x) returned by DynamicalAnalysis, to distinguish transition cells (high entropy, near saddle/maximum points between attractor basins) from meta-stable cells (entropy near zero).

## When to use

Use when a MuTrans DynamicalAnalysis output (attractor basins Sk* and optimized membership ρ*(x)) is available and you need to score each cell's transition propensity, e.g. to locate saddle points between attractors in a cell-fate dynamical manifold.

## When NOT to use

- You only need static clustering labels (e.g. Louvain/Leiden) without any notion of transition dynamics or transient cells
- The dataset has too few cells to resolve transition cells at high resolution (e.g. small myelopoiesis datasets, which the authors flag as needing special care)
- No membership/attractor-basin decomposition has been inferred yet — run DynamicalAnalysis first; the entropy cannot be computed directly from raw expression

## Inputs

- Processed single-cell gene expression matrix (N_cells x N_genes), e.g. from the MuTrans-release Data repository
- MuTrans DynamicalAnalysis output: attractor basins Sk*, coarse-grained rwTPM P̂ij, and optimized membership function ρ*(x)
- Number of attractors K_cluster (e.g. validated by the Eigen-Peak Index eigen-gap)

## Outputs

- Per-cell transition entropy H(x) values in [0,1]
- Per-cell transition cell score (TCS) values for specified attractor pairs Si*, Sj*
- Per-cell table such as transition_cell_analysis.csv with tcs and entropy columns
- Dynamical manifold plots colored by transition entropy

## How to apply

First run the MuTrans multi-scale analysis (DynamicalAnalysis with K_cluster set to the inferred number of attractors, e.g. K_cluster=3 confirmed by an eigen-gap in the Eigen-Peak Index for the triple-well simulation) to obtain the attractor basins S_k, coarse-grained transition matrix P̂ij, and optimized membership function ρ*(x). For each cell x, compute the transition entropy H(x) = −Σ_k ρk*(x) log ρk*(x) over the K attractor membership values, optionally alongside the transition cell score τij(x) = ρi*(x)/(ρi*(x)+ρj*(x)) for a specific Si*→Sj* transition. Write per-cell entropy (and TCS) values to per-cell output (e.g. transition_cell_analysis.csv) and visualize cells on the dynamical manifold colored by entropy. Because ρ* is a probability vector, entropies fall in [0,1] with the maximum at K-way ambiguity between attractors. The rationale is that stable cells sit inside attractor basins (entropy near zero) while transition cells occupy saddle regions where membership is mixed.

## Related tools

- **MuTrans** (Runs DynamicalAnalysis (multi-scale coarse-graining) that returns the attractor basins and optimized membership function ρ*(x) from which the transition entropy is computed; Transcendental performs downstream transition-cell analysis) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Dependency used in the Python MuTrans workflow (pyMuTrans) supporting the random-walk/coarse-grained dynamics analysis feeding the membership computation) — http://emma-project.org/latest/INSTALL.html
- **Scanpy** (Python dependency used to handle the AnnData object passed through pm.dynamical_analysis) — https://scanpy.readthedocs.io/en/stable/installation.html

## Examples

```
par = {"K_cluster": 3.0}; adata = pm.dynamical_analysis(adata,par)  # then compute H(x) = -sum_k rho_k*(x) log rho_k*(x) from adata membership
```

## Evaluation signals

- All per-cell entropy (and TCS) values lie within [0,1], since ρ*(x) is a probability vector over K attractors
- For the triple-well simulation with K_cluster=3, exactly 3 attractor basins are inferred and the coarse-grained rwTPM is 3x3
- Transition entropy is markedly higher for cells near saddle/maximum points between wells and near zero for cells inside attractor basins
- Output file (e.g. transition_cell_analysis.csv) exists and contains tcs and entropy columns for all analyzed cells
- The chosen K_cluster is corroborated by an eigen-gap in the Eigen-Peak Index (EPI) of the cell-cell scale rwTPM

## Limitations

- A sufficiently large number of cells is required for high-resolution identification of transition cells; small datasets (e.g. myelopoiesis) need special care and further confirmation
- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- Large datasets may require the DECLARE aggregation module (reduce_large_scale, reduce_num_meta_cell) to speed up the multi-scale analysis

## Evidence

- [intro] TCS and entropy distinguish attractors from transition cells: "the barrier height naturally quantifies the likelihood of cell-fate switch, and the Transition Cell Score (TCS) and transition entropy allows us to distinguish between attractors and transition cells"
- [readme] DynamicalAnalysis returns membership assignment: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [abstract] MuTrans manifold distinguishes stable and transition cells: "MuTrans constructs the cell-fate dynamical manifold that depicts progression of cell-state transitions, and distinguishes stable"
- [methods] Transition entropy listed as a Methods topic: "## Methods > Transition Entropy"
