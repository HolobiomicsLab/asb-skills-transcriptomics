---
name: multiscale-coarse-graining-attractor-identification
description: Use when single-cell RNA-seq data must be decomposed into attractor basins
  and transition cells via stochastic dynamics — e.g., recovering EMT epithelial/mesenchymal/hybrid
  states or bifurcating iPSC lineages from a processed gene expression matrix.
when_to_use_negative:
- The dataset has very few cells (e.g., the myelopoiesis Fluidigm C1 dataset), where
  transition-cell identification requires special care and confirmation
- The goal is static clustering of distinct states only — high-accuracy clusterers
  like SC3 or SIMLR enforce distinct states and do not model transient cells
- Root cell state detection via RNA velocity or entropy methods is required, which
  MuTrans does not currently integrate
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MuTrans / pyMuTrans
  role: 'Core method: learns cell-cell rwTPM via Gaussian-like kernel, coarse-grains
    into attractor basins, and optimizes cell-cluster membership function rho_k(x)'
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: Scanpy
  role: Loads the processed EMT/iPSC gene expression matrices into an AnnData object
    for pyMuTrans analysis
- name: DECLARE
  role: Optional pre-processing dynamics-preserving cell aggregation module (reduce_num_meta_cell=1500)
    that speeds up calculation on large datasets
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Dependency package used for transition path analyses (e.g., MPPT lineage inference)
    on the coarse-grained dynamics
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:ec587a9207e03ed5b43f67648916a51af6e83d09610d314c68d415438a6ed9b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - DECLARE
  - MuTrans
  - MuTrans / pyMuTrans
  - PyEMMA
  - Scanpy
  - pyMuTrans
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# multiscale-coarse-graining-attractor-identification

## Summary

Runs MuTrans' multi-scale analysis of the cell-cell random-walk transition probability matrix (rwTPM) to coarse-grain single-cell transcriptome dynamics into K attractor basins with soft cell membership, recovering meta-stable states and transition cells. Validated on EMT (Smart-Seq2, K=5) and iPSC induction (single-cell RT-qPCR, K=9) datasets.

## When to use

Use when single-cell RNA-seq data must be decomposed into attractor basins and transition cells via stochastic dynamics — e.g., recovering EMT epithelial/mesenchymal/hybrid states or bifurcating iPSC lineages from a processed gene expression matrix.

## When NOT to use

- The dataset has very few cells (e.g., the myelopoiesis Fluidigm C1 dataset), where transition-cell identification requires special care and confirmation
- The goal is static clustering of distinct states only — high-accuracy clusterers like SC3 or SIMLR enforce distinct states and do not model transient cells
- Root cell state detection via RNA velocity or entropy methods is required, which MuTrans does not currently integrate

## Inputs

- Pre-processed single-cell gene expression matrix (N_cells x N_genes), e.g., from MuTrans-release/Data folder, loaded into a Scanpy AnnData object
- Parameter dictionary (par) with K_cluster number of attractors; optional reduce_large_scale=True and reduce_num_meta_cell=1500 for DECLARE

## Outputs

- Attractor basin assignments per cell
- Coarse-grained (cluster-cluster) transition probabilities
- Optimized cell-cluster soft membership matrix rho_k(x)
- Cell-fate dynamical manifold / landscape visualization

## How to apply

Load the pre-processed gene expression matrix (e.g., from MuTrans-release/Data) into an AnnData object with Scanpy. Choose K_cluster guided by the Eigen-Peak Index (EPI) of the cell-cell rwTPM or biological prior (K=5 for SCC Smart-Seq2 EMT; K=9 for iPSC induction). Run pyMuTrans dynamical_analysis, which learns the rwTPM via a Gaussian-like kernel, coarse-grains it into K attractor basins with coarse-grained transition probabilities, and optimizes the cell-cluster membership matrix rho_k(x) via quasi-Newton minimization of the coarse-grained rwTPM distance to the original rwTPM. Optionally enable the DECLARE module (reduce_large_scale=True, reduce_num_meta_cell=1500) to aggregate cells into micro-states for large datasets. Visualize the dynamical manifold with ConstructLandscape and confirm stable cells sit in attractor basins while transition cells lie along connecting paths.

## Related tools

- **MuTrans / pyMuTrans** (Core method: learns cell-cell rwTPM via Gaussian-like kernel, coarse-grains into attractor basins, and optimizes cell-cluster membership function rho_k(x)) — https://github.com/cliffzhou92/MuTrans-release
- **Scanpy** (Loads the processed EMT/iPSC gene expression matrices into an AnnData object for pyMuTrans analysis)
- **DECLARE** (Optional pre-processing dynamics-preserving cell aggregation module (reduce_num_meta_cell=1500) that speeds up calculation on large datasets) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Dependency package used for transition path analyses (e.g., MPPT lineage inference) on the coarse-grained dynamics)

## Examples

```
import pyMuTrans as pm; par = {"K_cluster": 5.0, "reduce_large_scale": True, "reduce_num_meta_cell": 1500}; adata = pm.dynamical_analysis(adata, par)
```

## Evaluation signals

- Recovered attractor count equals the specified K_cluster (5 for SCC EMT Smart-Seq2; 9 for iPSC induction)
- Eigen-Peak Index (EPI) of the rwTPM shows eigen-gaps consistent with the chosen number of attractors
- Dynamical manifold (ConstructLandscape) recovers epithelial, mesenchymal, and hybrid attractor basins with transition cells along connecting paths
- Coarse-grained rwTPM distance to the original rwTPM is minimized by the quasi-Newton-optimized membership function
- Per-cell transition entropy H(x) = -sum_k rho_k*(x) log rho_k*(x) is low inside attractor basins and elevated along transition paths

## Limitations

- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- The number of cells needs to be sufficiently large for high-resolution identification of transition cells; small datasets (e.g., myelopoiesis) require special care
- Detailed balance may be violated at the microscopic scale (Eq. 1); recovery of transitions relies on the coarse-grained mesoscopic dynamics
- The continuum limit of the soft membership function in embedded over-damped Langevin systems remains an open problem

## Evidence

- [intro] MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel: "MuTrans first learns the cellular random walk transition probability matrix (rwTPM) on the cell-cell scale through a Gaussian-like kernel"
- [intro] coarse-graining and membership optimization: "the method performs coarse-graining on the cell–cell scale rwTPM to learn the dynamics on the cluster-cluster scale"
- [intro] membership matrix optimization: "constructed via optimizing a cell-cluster membership matrix"
- [discussion] DECLARE speed-up: "MuTrans introduces a pre-processing method (DECLARE) to aggregate the cells and speed up computation."
- [readme] README basic usage with K_cluster=5 and DECLARE parameters: "par = {"K_cluster": 5.0} # number of attractors, can plot EPI to give the rough estimate of clusters"
- [readme] README DynamicalAnalysis outputs: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
