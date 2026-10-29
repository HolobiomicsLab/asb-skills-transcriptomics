---
name: cell-fate-dynamical-manifold-construction
description: Use when you have a cell-cell rwTPM and DynamicalAnalysis output (attractor
  basins, coarse-grained P_ij, soft membership rho*(x)) from MuTrans and need to visualize
  the manifold, place attractor basins and saddle/transition cells on it, or sanity-check
  basin structure before lineage (MPPT) or.
when_to_use_negative:
- Dataset has too few cells (myelopoiesis-scale) for high-resolution transition-cell
  identification — the paper states special care is needed there before trusting the
  manifold.
- Non-stationary dynamics dominated by cell cycle or proliferation effects, which
  the current MuTrans method does not consider.
- You only need a static clustering (SC3/SIMLR-style) rather than attractor-basin
  structure and transition paths — high-accuracy clustering enforces distinct states
  and loses transient cells.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0602
tools:
- name: MuTrans
  role: Provides the multiscale rwTPM analysis (DynamicalAnalysis) whose Output object
    ConstructLandscape takes as input to fit the dynamical manifold.
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: pyMuTrans
  role: Python API (pm.dynamical_analysis, pm.infer_lineage) run on AnnData objects
    to produce the membership matrix and lineage overlay plotted on the manifold.
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: Scanpy
  role: Loads the processed gene expression matrices from the MuTrans-release Data
    folder into the AnnData object consumed by pyMuTrans.
- name: PyEMMA
  role: Implements the MPPT lineage inference whose transition trajectories are overlaid
    on the constructed manifold.
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:b36dd57b7001e749c232ecca3d04930030a1a38a55269a699835c7f84e5111b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - DECLARE
  - MuTrans
  - NumPy
  - Pandas
  - PyEMMA
  - Scanpy
  - pyMuTrans
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cell-fate-dynamical-manifold-construction

## Summary

Construct and visualize the MuTrans cell-fate dynamical manifold: a Gaussian-mixture landscape fit with attractor centers Yk and per-cell embedding xi(x) in which stable cells reside in attractor basins (E/M/hybrid, Pro-B/Pre-B, etc.) and transition cells are assigned along transition paths between basins.

## When to use

Use when you have a cell-cell rwTPM and DynamicalAnalysis output (attractor basins, coarse-grained P_ij, soft membership rho*(x)) from MuTrans and need to visualize the manifold, place attractor basins and saddle/transition cells on it, or sanity-check basin structure before lineage (MPPT) or Transcendental gene analysis.

## When NOT to use

- Dataset has too few cells (myelopoiesis-scale) for high-resolution transition-cell identification — the paper states special care is needed there before trusting the manifold.
- Non-stationary dynamics dominated by cell cycle or proliferation effects, which the current MuTrans method does not consider.
- You only need a static clustering (SC3/SIMLR-style) rather than attractor-basin structure and transition paths — high-accuracy clustering enforces distinct states and loses transient cells.

## Inputs

- Processed single-cell gene expression matrix (MuTrans-release Data folder) loaded into an AnnData object via Scanpy
- MuTrans parameter dictionary par with K_cluster (e.g. 5.0) and optional DECLARE settings (reduce_large_scale=True, reduce_num_meta_cell=1500)
- Output object from DynamicalAnalysis containing attractor basins S_k, coarse-grained KxK rwTPM P_ij, and optimized membership rho_k*(x)

## Outputs

- Cell-fate dynamical manifold / landscape (attractor centers Yk and per-cell embedding xi(x)) rendered as a figure
- Per-cell coordinates on the manifold colored by basin membership, transition entropy H(x) = -sum_k rho*_k(x) log rho*_k(x), or TCS
- Visualization confirming which cells lie in basins versus along transition paths (e.g. Pro-B/Pre-B boundary, E-ICS-M routes)

## How to apply

First run pyMuTrans pm.dynamical_analysis on an AnnData object loaded from the processed expression matrix (e.g. K_cluster=5 with DECLARE reduce_num_meta_cell=1500 for SCC Smart-Seq2 EMT; K_cluster=3 for the triple-well simulation; K_cluster=9 for iPSC RT-qPCR), choosing K from the Eigen-Peak Index (EPI) eigen-gaps or biological priors. Then call ConstructLandscape (Matlab) on the returned Output object, or the equivalent landscape construction in pyMuTrans, which fits the dynamical manifold as a mixture distribution with attractor centers Yk and per-cell embedding xi(x). Verify that stable cells sit inside attractor basins while transition cells fall along the connecting paths (e.g. transition cells along the saddle between Pro-B and Pre-B basins; E->ICS->M routes in EMT). Optionally overlay pm.infer_lineage(adata, si, sf, method='MPPT') to check that inferred lineage paths thread the saddle regions of the fitted manifold. The rationale is that the manifold renders the multiscale rwTPM coarse-graining interpretable: basins map to meta-stable attractors and path-connected cells to transient states.

## Related tools

- **MuTrans** (Provides the multiscale rwTPM analysis (DynamicalAnalysis) whose Output object ConstructLandscape takes as input to fit the dynamical manifold.) — https://github.com/cliffzhou92/MuTrans-release
- **pyMuTrans** (Python API (pm.dynamical_analysis, pm.infer_lineage) run on AnnData objects to produce the membership matrix and lineage overlay plotted on the manifold.) — https://github.com/cliffzhou92/MuTrans-release
- **Scanpy** (Loads the processed gene expression matrices from the MuTrans-release Data folder into the AnnData object consumed by pyMuTrans.)
- **PyEMMA** (Implements the MPPT lineage inference whose transition trajectories are overlaid on the constructed manifold.)

## Examples

```
Land = ConstructLandscape(Output,par)
```

## Evaluation signals

- Number of recovered attractor basins equals the EPI-indicated K and matches validation targets (5 for SCC EMT, 3 for the triple-well simulation with a 3x3 coarse-grained rwTPM, 9 for iPSC).
- Stable cells reside inside attractor basins while transition cells localize along the saddle/transition paths connecting basins (e.g. between Pro-B and Pre-B).
- Transition entropy is markedly higher near saddle/maximum points between wells than inside basins (entropy near zero for stable cells).
- ConstructLandscape only accepts the Output object of DynamicalAnalysis — the pipeline order (dynamical_analysis -> landscape) must be preserved.
- Reproduced basin assignments agree with Supplementary annotations (e.g. LP, Pro-B, Pre-B) and reported figure values (Fig. 3f-g for EMT).

## Limitations

- A sufficiently large number of cells is required for high-resolution identification of transition cells; small datasets (e.g. myelopoiesis) need special care and further confirmation.
- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method.
- Detailed balance may be violated at the microscopic (Eq. 1) scale; the coarse-grained mesoscopic dynamics is relied upon to recover transitions.
- Root cell state detection could be enhanced via entropy methods or RNA velocity, which are not currently integrated.

## Evidence

- [intro] MuTrans fits a dynamical manifold using a mixture distribution to make stable cells reside in the attractor basins while assign transition cells along the transition paths: "MuTrans fits a dynamical manifold using a mixture distribution to make stable cells reside in the attractor basins while assign transition cells along the transition paths"
- [readme] Construct and visualize the dynamical manifold: "%% Construct and visualize the dynamical manifold"
- [methods] We use the Eigen-Peak Index (EPI) to visualize the multiple eigen-gaps of cell-cell scale rwTPM: "We use the Eigen-Peak Index (EPI) to visualize the multiple eigen-gaps of cell-cell scale rwTPM (Supplementary Note 2)."
- [intro] Coarse-grain the cell-cell scale rwTPM to learn cluster-cluster scale dynamics, acquiring attractor basins and mutual conversion probabilities: "the method performs coarse-graining on the cell–cell scale rwTPM to learn the dynamics on the cluster-cluster scale"
