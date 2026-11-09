---
name: random-walk-transition-matrix-construction
description: Use when you have a pre-processed N_cells x N_genes single-cell expression
  matrix and need to model cell-fate transition dynamics as a Markov random walk before
  attractor/coarse-graining analysis.
when_to_use_negative:
- The input is not a pre-processed expression matrix but raw counts or already-clustered/feature-summarized
  data lacking per-cell gene expression
- The dataset has very few cells where high-resolution identification of transition
  cells is unreliable (e.g., the myelopoiesis dataset required special care)
- The goal is static clustering or pseudo-time ordering alone (e.g., DPT/Slingshot/Monocle)
  without any stochastic dynamical modeling of transitions
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MuTrans
  role: Core method that learns the cell-cell rwTPM via a Gaussian-like kernel and
    supports multiscale reduction to attractor basins and transition paths
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: pyMuTrans
  role: Python API whose dynamical_analysis entry point builds the rwTPM on an AnnData
    object and exposes DECLARE acceleration
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: Scanpy
  role: Used to load and pre-process the expression matrix into an AnnData object
    before rwTPM construction
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
- name: PyEMMA
  role: Dependency package used by the Python MuTrans workflow for transition-path
    machinery operating on the learned rwTPM
  repo: http://emma-project.org/latest/INSTALL.html
- name: DECLARE
  role: Optional dynamics-preserving cell aggregation pre-processing module that speeds
    up rwTPM computation by aggregating cells into micro-states
  repo: https://github.com/cliffzhou92/MuTrans-release
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:9dd333c49198c53113294a213a6dbd5090b1e5c5e99e177efc3a78c7b5ec1ca7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - DECLARE
  - MuTrans
  - PyEMMA
  - Scanpy
  - Transcendental
  - pyMuTrans
  - transition cell score (TCS)
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# random-walk-transition-matrix-construction

## Summary

Constructs the cell-cell scale random walk transition probability matrix (rwTPM) from a pre-processed single-cell gene expression matrix using a Gaussian-like affinity kernel with degree normalization, as the first step of MuTrans's multiscale stochastic dynamics framework.

## When to use

Use when you have a pre-processed N_cells x N_genes single-cell expression matrix and need to model cell-fate transition dynamics as a Markov random walk before attractor/coarse-graining analysis.

## When NOT to use

- The input is not a pre-processed expression matrix but raw counts or already-clustered/feature-summarized data lacking per-cell gene expression
- The dataset has very few cells where high-resolution identification of transition cells is unreliable (e.g., the myelopoiesis dataset required special care)
- The goal is static clustering or pseudo-time ordering alone (e.g., DPT/Slingshot/Monocle) without any stochastic dynamical modeling of transitions

## Inputs

- Pre-processed single-cell gene expression matrix (N_cells x N_genes), e.g., loaded as an AnnData object with Scanpy or as a Matlab matrix
- Optional DECLARE parameters (reduce_large_scale=True, reduce_num_meta_cell) for large datasets

## Outputs

- Row-stochastic cell-cell rwTPM (e.g., rwtpm_cell_cell.npz) with stationary distribution mu(x) = d(x)/sum_z d(z) satisfying detailed balance
- Basis for downstream attractor basins, cluster-cluster transition matrix, and membership matrix rho*

## How to apply

Load the pre-processed gene expression matrix (N_cells x N_genes) and compute pairwise affinities via a Gaussian-like kernel w(x,y). Normalize each row by the degree d(x) = sum_z w(x,z) so that p(x,y) = w(x,y)/d(x), yielding a row-stochastic transition matrix whose stationary distribution mu(x) = d(x)/sum_z d(z) satisfies detailed balance. Optionally apply the DECLARE aggregation module (e.g., reduce_num_meta_cell=1500) to accelerate computation on large datasets. This rwTPM is the substrate for downstream steps: choosing the number of attractors K via Eigen-Peak Index eigen-gaps, coarse-graining to the cluster-cluster scale, and optimizing the soft membership matrix rho_k via quasi-Newton minimization. Save the matrix in machine-readable form (e.g., rwtpm_cell_cell.npz) for reuse by pyMuTrans or Matlab MuTrans routines.

## Related tools

- **MuTrans** (Core method that learns the cell-cell rwTPM via a Gaussian-like kernel and supports multiscale reduction to attractor basins and transition paths) — https://github.com/cliffzhou92/MuTrans-release
- **pyMuTrans** (Python API whose dynamical_analysis entry point builds the rwTPM on an AnnData object and exposes DECLARE acceleration) — https://github.com/cliffzhou92/MuTrans-release
- **Scanpy** (Used to load and pre-process the expression matrix into an AnnData object before rwTPM construction) — https://scanpy.readthedocs.io/en/stable/installation.html
- **PyEMMA** (Dependency package used by the Python MuTrans workflow for transition-path machinery operating on the learned rwTPM) — http://emma-project.org/latest/INSTALL.html
- **DECLARE** (Optional dynamics-preserving cell aggregation pre-processing module that speeds up rwTPM computation by aggregating cells into micro-states) — https://github.com/cliffzhou92/MuTrans-release

## Examples

```
import pyMuTrans as pm; par={"K_cluster":5.0,"reduce_large_scale":True,"reduce_num_meta_cell":1500}; adata=pm.dynamical_analysis(adata,par)
```

## Evaluation signals

- Every row of the computed rwTPM is row-stochastic: all row sums fall within [1-1e-6, 1+1e-6]
- The stationary distribution mu(x) = d(x)/sum_z d(z) derived from the degrees satisfies detailed balance under the constructed kernel
- Eigen-Peak Index (EPI) plot of the rwTPM shows interpretable eigen-gaps that guide the choice of K attractors (e.g., K=5 for SCC EMT, K=9 for iPSC data)
- The construction script runs to completion without error and the downstream DynamicalAnalysis returns attractor basins, coarse-grained transition probabilities, and membership assignment
- The saved matrix file (e.g., rwtpm_cell_cell.npz) is machine-readable and matches the dimensions of the input expression matrix (N_cells x N_cells)

## Limitations

- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- The number of cells needs to be sufficiently large for high-resolution identification of transition cells; small datasets (e.g., myelopoiesis) require special care
- Detailed balance may be violated at the microscopic scale (Eq. 1); the coarse-grained mesoscopic dynamics are relied upon to recover transitions
- Matlab usage requires R2019b/R2020a with the optimization toolbox (and Curve Fitting toolbox for Transcendental smoothing)

## Evidence

- [intro] MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel: "MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel"
- [other] so that p(x,y) = w(x,y)/d(x) yields a stationary distribution mu(x) = d(x)/sum_z d(z) satisfying detailed balance: [evidence span withheld — anchor status: not_found]
- [intro] we provide an option to speed up calculation by introducing a pre-processing aggregation module DECLARE (dynamics-preserving cells aggregation): "we provide an option to speed up calculation by introducing a pre-processing aggregation module DECLARE (dynamics-preserving cells aggregation)"
- [readme] %% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [methods] We use the Eigen-Peak Index (EPI) to visualize the multiple eigen-gaps of cell-cell scale rwTPM (Supplementary Note 2).: "We use the Eigen-Peak Index (EPI) to visualize the multiple eigen-gaps of cell-cell scale rwTPM (Supplementary Note 2)."
