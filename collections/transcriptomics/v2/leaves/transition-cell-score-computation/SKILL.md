---
name: transition-cell-score-computation
description: Use when MuTrans DynamicalAnalysis has returned attractor basins and
  optimized membership functions rho*(x) for a dataset, and you need to order cells
  along a specific state transition i->j, distinguish stable (attractor) cells from
  transition-layer cells, or feed per-cell TCS into downstream.
when_to_use_negative:
- The dataset has few cells (e.g., the myelopoiesis Fluidigm C1 dataset), where high-resolution
  identification of transition cells requires special care and confirmation.
- You only have raw expression data and have not yet run MuTrans DynamicalAnalysis
  — TCS depends on the optimized membership function rho*(x) and attractor basins.
- Non-stationary effects such as cell cycle or cell proliferation dynamics dominate,
  which the current MuTrans method does not model.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: MuTrans
  role: Provides DynamicalAnalysis (or pyMuTrans pm.dynamical_analysis) producing
    the attractor basins and membership function rho*(x) from which the TCS is computed
    via the Transcendental/GeneAnalysis procedure.
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Used for transition path theory computations (effective transition-path currents
    fij+) on the coarse-grained dynamics that contextualize the transition cells scored
    by TCS.
- name: Scanpy
  role: Loads the processed gene expression matrix (e.g., Cel-Seq2 Herman et al. example_mpp
    data) into the AnnData object passed to pyMuTrans.
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:4bb7409bf8471dd196985bd4f0f595c1ebebdf8ba42ad9ed0bc5a5bc448cfbad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - MuTrans
  - PyEMMA
  - Scanpy
  - Transcendental
  - transition cell score (TCS)
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# transition-cell-score-computation

## Summary

Computation of the transition cell score (TCS) tau_ij(x) = rho_i*(x)/(rho_i*(x)+rho_j*(x)) in MuTrans's Transcendental procedure, which quantifies each cell's relative position between a starting attractor basin Si* and a targeting basin Sj* to separate stable cells from transition cells. It relies on the optimized soft membership function rho*(x) from MuTrans's multi-scale analysis of the rwTPM.

## When to use

Use when MuTrans DynamicalAnalysis has returned attractor basins and optimized membership functions rho*(x) for a dataset, and you need to order cells along a specific state transition i->j, distinguish stable (attractor) cells from transition-layer cells, or feed per-cell TCS into downstream TD/IH/MS gene classification.

## When NOT to use

- The dataset has few cells (e.g., the myelopoiesis Fluidigm C1 dataset), where high-resolution identification of transition cells requires special care and confirmation.
- You only have raw expression data and have not yet run MuTrans DynamicalAnalysis — TCS depends on the optimized membership function rho*(x) and attractor basins.
- Non-stationary effects such as cell cycle or cell proliferation dynamics dominate, which the current MuTrans method does not model.

## Inputs

- AnnData object (or Matlab data matrix N_cells x N_genes) after pm.dynamical_analysis / DynamicalAnalysis, containing attractor basins Sk*, coarse-grained transition matrix, and optimized membership rho*(x)
- Indices/labels of the starting attractor state i and targeting attractor state j for the transition of interest

## Outputs

- Per-cell TCS values tau_ij(x) (with cell IDs and membership fractions), e.g. tcs_saddle_node.csv or transcendental_tcs_proB_preB.csv
- Ordered transition cells separating stable cells (TCS ~0 or ~1) from intermediate-TCS transition cells (TCS in (0,1))
- Optionally per-cell transition entropy H(x), jointly exported with TCS in transition_cell_analysis.csv

## How to apply

First run pm.dynamical_analysis(adata, par) (Matlab: DynamicalAnalysis(data,par)) with K_cluster chosen from the Eigen-Peak Index (EPI) eigen-gaps or prior annotations, obtaining attractor basins Sk*, coarse-grained transition probabilities, and the optimized membership function rho*(x). Then, in the Transcendental GeneAnalysis(i,j,Output,par) step between starting state i and targeting state j, compute the transition cell score tau_ij(x) = rho_i*(x)/(rho_i*(x)+rho_j*(x)) for every cell selected by the membership function. Optionally also compute per-cell transition entropy H(x) = -sum_k rho_k*(x) log rho_k*(x). Stable cells should receive TCS near 0 or 1 while transition cells along the saddle path between basins form a continuum of intermediate values in (0,1); cells can then be ordered by TCS for downstream TD, IH, and MS gene classification. Export per-cell scores (e.g., tcs_saddle_node.csv or transcendental_tcs_proB_preB.csv) alongside membership fractions.

## Related tools

- **MuTrans** (Provides DynamicalAnalysis (or pyMuTrans pm.dynamical_analysis) producing the attractor basins and membership function rho*(x) from which the TCS is computed via the Transcendental/GeneAnalysis procedure.) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Used for transition path theory computations (effective transition-path currents fij+) on the coarse-grained dynamics that contextualize the transition cells scored by TCS.)
- **Scanpy** (Loads the processed gene expression matrix (e.g., Cel-Seq2 Herman et al. example_mpp data) into the AnnData object passed to pyMuTrans.)

## Examples

```
import pyMuTrans as pm; par={"K_cluster":5.0}; adata=pm.dynamical_analysis(adata,par); pm.gene_analysis(adata,si=0,sf=2)  # Transcendental GeneAnalysis computes TCS tau_ij = rho_i*/(rho_i*+rho_j*) per cell
```

## Evaluation signals

- All TCS values lie within [0,1] by construction of tau_ij(x) = rho_i*(x)/(rho_i*(x)+rho_j*(x)); verify this invariant in the output table.
- In the saddle-node bifurcation simulation (N=2,000), the fraction of stable cells with TCS < 0.05 or > 0.95 should be consistent with the reported Fig. 2a pattern.
- Computed TCS values should match the tutorial notebook (example_saddle_node.mlx) reference TCS output within tolerance.
- Intermediate-TCS transition cells should lie along the saddle path between basins on the dynamical manifold (e.g., between Pro-B and Pre-B attractors in the Herman et al. Cel-Seq2 lymphoid data), reproducing the reported fraction of transition cells within an expert-tolerated range.
- Output file schema checks: contains cell ID, membership fractions, and tcs (and optionally entropy) columns with one score per relevant cell.

## Limitations

- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method.
- A sufficiently large number of cells is needed for high-resolution identification of transition cells; small datasets (e.g., myelopoiesis) require special care and further confirmation.
- The continuum limit of the soft membership function in embedded over-damped Langevin dynamical systems remains an open problem.

## Evidence

- [methods] we define the transition cell score (TCS) to measure the relative position of cell x in different clusters.: [evidence span withheld — anchor status: not_found]
- [methods] MuTrans performs the Transcendental (transition cells and relevant analysis) procedure on each transition process to identify the transition cells from the stable cells: "MuTrans performs the Transcendental (transition cells and relevant analysis) procedure on each transition process to identify the transition cells from the stable cells"
- [readme] %% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [intro] the Transition Cell Score (TCS) and transition entropy allows us to distinguish between attractors and transition cells: "the Transition Cell Score (TCS) and transition entropy allows us to distinguish between attractors and transition cells"
- [discussion] When the number of cells is relatively small, such as in the myelopoiesis dataset studied here, special care is needed to further confirm the analysis of transition cells.: "When the number of cells is relatively small, such as in the myelopoiesis dataset studied here, special care is needed to further confirm the analysis of transition cells."
