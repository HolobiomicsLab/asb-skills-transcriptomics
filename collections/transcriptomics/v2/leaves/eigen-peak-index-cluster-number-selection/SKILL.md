---
name: eigen-peak-index-cluster-number-selection
description: 'Use when running MuTrans and the number of attractors (K_cluster) is
  unknown: compute the EPI spectrum on the Gaussian-kernel cell-cell rwTPM and pick
  K at the clearest eigen-gap before DynamicalAnalysis.'
when_to_use_negative:
- When K_cluster is already fixed by a biological prior or by other clustering results
  (Louvain/Leiden), which the README lists as acceptable alternatives to EPI
- When the input is not a cell-level snapshot/expression matrix from which a cell-cell
  rwTPM can be constructed (EPI is defined on the rwTPM, not on raw features)
- When analyzing very small datasets where the discussion notes high-resolution inference
  requires a sufficiently large number of cells and 'special care is needed'
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: MuTrans
  role: Provides the cell-cell rwTPM (Gaussian-like kernel) whose eigen-gaps the EPI
    visualizes; K_cluster feeds DynamicalAnalysis multi-scale coarse-graining
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: pyMuTrans
  role: Python interface where the estimated K is set as par['K_cluster'] before pm.dynamical_analysis
    on the AnnData object
  repo: https://github.com/cliffzhou92/MuTrans-release
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:25fa39357554aa8d8de172b69db95dbf68e862d4098bf144354fdcb50173016e
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
  - pyMuTrans
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# eigen-peak-index-cluster-number-selection

## Summary

Selects the number of attractor basins (K_cluster) for MuTrans DynamicalAnalysis by inspecting the Eigen-Peak Index (EPI), which visualizes multiple eigen-gaps of the cell-cell scale random walk transition probability matrix (rwTPM).

## When to use

Use when running MuTrans and the number of attractors (K_cluster) is unknown: compute the EPI spectrum on the Gaussian-kernel cell-cell rwTPM and pick K at the clearest eigen-gap before DynamicalAnalysis.

## When NOT to use

- When K_cluster is already fixed by a biological prior or by other clustering results (Louvain/Leiden), which the README lists as acceptable alternatives to EPI
- When the input is not a cell-level snapshot/expression matrix from which a cell-cell rwTPM can be constructed (EPI is defined on the rwTPM, not on raw features)
- When analyzing very small datasets where the discussion notes high-resolution inference requires a sufficiently large number of cells and 'special care is needed'

## Inputs

- Pre-processed single-cell gene expression or simulation snapshot matrix (N_cells x N_genes), e.g. the triple-well Euler-Maruyama snapshots
- Cell-cell scale random walk transition probability matrix (rwTPM) built with the Gaussian-like affinity kernel

## Outputs

- Eigen-Peak Index (EPI) spectrum/plot over candidate K values showing eigen-gaps
- Selected number of attractors K_cluster (e.g., K=3 for the triple-well system)
- Validated K x K coarse-grained rwTPM and attractor basin assignments from downstream DynamicalAnalysis

## How to apply

First learn the cell-cell scale rwTPM from the snapshot/expression matrix via MuTrans' Gaussian-like kernel, since EPI is defined on this matrix. Compute and plot the Eigen-Peak Index to visualize the multiple eigen-gaps of the rwTPM and read off the K with the dominant gap — e.g., for the triple-well simulation (N=2,001 Euler-Maruyama snapshots of the over-damped Langevin system, example_triple_well.mlx) the eigen-gap confirms K=3 attractor basins. Set the chosen value as par['K_cluster'] in pyMuTrans (or pass it to the Matlab DynamicalAnalysis) and run the multi-scale coarse-graining. Decision point: the README notes EPI gives only a rough estimate, so cross-check against Louvain/Leiden clustering results or a biological prior; if they conflict, prefer the physically/biologically motivated K. Rationale: the eigen-gap marks the scale separation between cell-cell random-walk dynamics and cluster-cluster dynamics, so the basin count K chosen this way directly determines the K x K coarse-grained rwTPM returned by DynamicalAnalysis.

## Related tools

- **MuTrans** (Provides the cell-cell rwTPM (Gaussian-like kernel) whose eigen-gaps the EPI visualizes; K_cluster feeds DynamicalAnalysis multi-scale coarse-graining) — https://github.com/cliffzhou92/MuTrans-release
- **pyMuTrans** (Python interface where the estimated K is set as par['K_cluster'] before pm.dynamical_analysis on the AnnData object) — https://github.com/cliffzhou92/MuTrans-release

## Examples

```
import pyMuTrans as pm; par = {"K_cluster": 3.0}; adata = pm.dynamical_analysis(adata, par)  # K=3 chosen after inspecting the EPI eigen-gap (triple-well example_triple_well.mlx)
```

## Evaluation signals

- The EPI curve exhibits a clear dominant peak/eigen-gap at the chosen K (e.g., gap at K=3 for the triple-well simulation with N=2,001 snapshots)
- DynamicalAnalysis with the EPI-selected K_cluster infers exactly that number of attractor basins (validation: inferred basins must equal 3 in the triple-well case)
- The coarse-grained rwTPM returned by DynamicalAnalysis has dimensions K x K (e.g., 3x3)
- rwTPM rows are valid stochastic transition probabilities (each row sums to 1) before EPI computation
- Stable cells show near-zero transition entropy while entropy peaks at saddle/maximum points, confirming the basin structure implied by the selected K

## Limitations

- EPI provides only a rough estimate of the cluster number; the README recommends also consulting Louvain/Leiden clustering results or a biological prior
- EPI is defined on the cell-cell rwTPM, so its quality depends on the Gaussian-like kernel construction and a sufficiently large number of cells for high-resolution identification
- The number of clusters chosen here fixes the coarse-graining scale; for small datasets (e.g., myelopoiesis) the article warns special care is needed to confirm downstream analysis

## Evidence

- [methods] EPI visualizes rwTPM eigen-gaps: "We use the Eigen-Peak Index (EPI) to visualize the multiple eigen-gaps of cell-cell scale rwTPM (Supplementary Note 2)."
- [readme] README: EPI for rough cluster estimate: "# number of attractors, can plot EPI to give the rough estimate of clusters or refer to other clustering results such as Louvain/Leiden results"
- [intro] rwTPM construction basis for EPI: "MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel"
