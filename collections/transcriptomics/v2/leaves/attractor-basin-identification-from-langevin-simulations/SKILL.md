---
name: attractor-basin-identification-from-langevin-simulations
description: Use when you have snapshot data (simulated or single-cell) from a known
  multi-stable stochastic system and need to verify that a dynamical method recovers
  the true attractor-basin structure and highlights transition cells near saddles.
when_to_use_negative:
- Data is a single-cell gene expression matrix where the number of cell states is
  unknown a priori and needs clustering (Louvain/Leiden or EPI) before setting K_cluster
- Small datasets (e.g., myelopoiesis-like cell counts) where the article states special
  care is needed to confirm transition-cell analysis
- Systems with non-stationary cell-cycle or proliferation dynamics, which the method
  does not consider
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3316
tools:
- name: MuTrans
  role: Performs the multi-scale rwTPM coarse-graining (DynamicalAnalysis) that returns
    attractor basins, coarse-grained transition probabilities and membership assignment
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Required dependency package (>=2.5.6) for running pyMuTrans analysis in Python
  repo: http://emma-project.org/latest/INSTALL.html
- name: Scanpy
  role: Provides the Anndata object passed to pm.dynamical_analysis and manifold visualization
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:75916afdb9b9d08129198f5b62f8d50ca6a833fce7636bf629c9923d6ec4a64b
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# attractor-basin-identification-from-langevin-simulations

## Summary

Runs MuTrans multiscale stochastic dynamics on snapshots of an overdamped Langevin system (e.g., N=2,001 Euler-Maruyama samples of a triple-well potential) to recover the number and structure of attractor basins, coarse-grained transition probabilities, and transition entropy at saddle points.

## When to use

Use when you have snapshot data (simulated or single-cell) from a known multi-stable stochastic system and need to verify that a dynamical method recovers the true attractor-basin structure and highlights transition cells near saddles.

## When NOT to use

- Data is a single-cell gene expression matrix where the number of cell states is unknown a priori and needs clustering (Louvain/Leiden or EPI) before setting K_cluster
- Small datasets (e.g., myelopoiesis-like cell counts) where the article states special care is needed to confirm transition-cell analysis
- Systems with non-stationary cell-cycle or proliferation dynamics, which the method does not consider

## Inputs

- Simulated snapshot data: N=2,001 Euler-Maruyama samples of the overdamped Langevin equation (triple-well potential) from the MuTrans-release Example/Data folder
- K_cluster parameter (number of attractors, here 3, informed by the Eigen-Peak Index eigen-gap)

## Outputs

- Attractor basin assignments S_k (exactly 3 basins for the triple-well system)
- 3x3 coarse-grained rwTPM of mutual conversion probabilities between basins
- Cell-cluster membership matrix rho*(x) and per-cell transition entropy H(x)
- Dynamical manifold visualization with cells colored by transition entropy

## How to apply

Load the Euler-Maruyama snapshots of the overdamped Langevin equation (e.g., the triple-well dataset in the MuTrans-release Data folder, following example_triple_well.mlx). Build the cell-cell random walk transition probability matrix (rwTPM) via a Gaussian-like affinity kernel and inspect the Eigen-Peak Index (EPI) to confirm the eigen-gap at K=3 attractors. Run DynamicalAnalysis with K_cluster=3 (Matlab: Output = DynamicalAnalysis(data,par); Python: pm.dynamical_analysis(adata,par)) to iteratively optimize attractor basins S_k, the cell-cluster membership matrix rho*, and the 3x3 coarse-grained rwTPM. Compute the transition entropy H(x) = -sum_k rho*_k(x) log rho*_k(x) per cell and visualize cells on the dynamical manifold (ConstructLandscape), coloring by entropy. Rationale: MuTrans's coarse-grained transition probability is consistent with Kramers' reaction rate theory for overdamped Langevin dynamics under steady-state and detailed-balance conditions, so simulated wells are a ground-truth validation case.

## Related tools

- **MuTrans** (Performs the multi-scale rwTPM coarse-graining (DynamicalAnalysis) that returns attractor basins, coarse-grained transition probabilities and membership assignment) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Required dependency package (>=2.5.6) for running pyMuTrans analysis in Python) — http://emma-project.org/latest/INSTALL.html
- **Scanpy** (Provides the Anndata object passed to pm.dynamical_analysis and manifold visualization) — https://scanpy.readthedocs.io/en/stable/installation.html

## Examples

```
import pyMuTrans as pm; par = {"K_cluster": 3.0}; adata = pm.dynamical_analysis(adata, par); pm.infer_lineage(adata, si=2, sf=0, method="MPPT")
```

## Evaluation signals

- Number of inferred attractor basins must equal 3 for the triple-well simulation
- Coarse-grained rwTPM must be 3x3 (K_cluster x K_cluster)
- Eigen-Peak Index must show a clear eigen-gap at K=3 in the cell-cell rwTPM
- Transition entropy must be markedly higher near saddle/maximum points between wells than for stable cells (entropy near zero at basins)
- Optimized basin assignments S_k should place stable cells inside wells and transition cells along transition paths

## Limitations

- Coarse-grained dynamics relies on steady-state and detailed-balance conditions; detailed balance may be violated at the microscopic scale
- Requires a sufficiently large number of samples/cells for high-resolution identification of transition cells
- Continuum limit of the soft membership function in embedded overdamped Langevin systems remains an open problem

## Evidence

- [methods] MuTrans coarse-graining on simulated wells: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [discussion] Kramers consistency rationale: [evidence span withheld — anchor status: not_found]
- [intro] rwTPM construction via Gaussian kernel: "MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel"
- [methods] EPI eigen-gap diagnostic: "We use the Eigen-Peak Index (EPI) to visualize the multiple eigen-gaps of cell-cell scale rwTPM (Supplementary Note 2)."
- [methods] Transition entropy definition: "## Methods > Transition Entropy"
- [readme] README parameter usage: "par = {"K_cluster": 5.0} # number of attractors, can plot EPI to give the rough estimate of clusters"
- [readme] README example notebook for triple well: "[Triple-well Potential: example_triple_well.mlx](https://github.com/cliffzhou92/MuTrans-release/blob/main/Example/example_triple_well.mlx)"
- [methods] Simulation data generation: "The simulation data was generated by the Euler-Maruyama method to solve the overdamped Langevin equations"
