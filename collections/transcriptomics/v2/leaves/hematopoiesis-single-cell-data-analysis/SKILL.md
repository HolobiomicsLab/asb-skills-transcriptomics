---
name: hematopoiesis-single-cell-data-analysis
description: Use when analyzing large-scale single-cell RNA-seq hematopoiesis data
  (e.g., human bone marrow from 10X Chromium) where cell-fate dynamics must be reconstructed
  efficiently and aggregation is needed to make multiscale transition analysis tractable.
when_to_use_negative:
- Dataset has relatively few cells (e.g., the myelopoiesis dataset studied here) —
  the paper notes special care is needed to confirm transition-cell analysis in small
  datasets, and DECLARE aggregation is unnecessary overhead
- Non-stationary effects such as cell cycle or cell proliferation dynamics are central
  to the question, since these were not considered in the current method
- The input is already aggregated or is not a single-cell expression matrix (e.g.,
  a feature table or pre-computed pseudotime)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MuTrans
  role: Core multi-scale stochastic dynamics method (pyMuTrans dynamical_analysis)
    that learns the cell-cell rwTPM, coarse-grains to attractor basins, and quantifies
    transition paths
  repo: github.com/cliffzhou92/MuTrans-release
- name: DECLARE
  role: Pre-processing dynamics-preserving cell aggregation module (reduce_large_scale=True,
    reduce_num_meta_cell=1500) that aggregates cells into micro-states via K-means
    or kNN to speed up MuTrans on large datasets
  repo: github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Dependency (>=2.5.6) used for transition path theory computation of transition
    path likelihoods
  repo: http://emma-project.org/latest/INSTALL.html
- name: Scanpy
  role: Used to load the processed bone-marrow expression matrix into an AnnData object
    for pyMuTrans analysis
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:0cb2d51923156a3ab3a69fee43b9e0c4ca4e54e45d69a62c1fd48f6a82ef8a8b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DECLARE
  - MuTrans
  - PyEMMA
  - Scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hematopoiesis-single-cell-data-analysis

## Summary

Applies the MuTrans multi-scale stochastic dynamics framework, with the DECLARE dynamics-preserving aggregation pre-processing module, to the human bone-marrow hematopoiesis (10X Chromium) dataset to reconstruct attractor basins, coarse-grained transition probabilities, and transition path likelihoods while speeding up computation by roughly an order of magnitude.

## When to use

Use when analyzing large-scale single-cell RNA-seq hematopoiesis data (e.g., human bone marrow from 10X Chromium) where cell-fate dynamics must be reconstructed efficiently and aggregation is needed to make multiscale transition analysis tractable.

## When NOT to use

- Dataset has relatively few cells (e.g., the myelopoiesis dataset studied here) — the paper notes special care is needed to confirm transition-cell analysis in small datasets, and DECLARE aggregation is unnecessary overhead
- Non-stationary effects such as cell cycle or cell proliferation dynamics are central to the question, since these were not considered in the current method
- The input is already aggregated or is not a single-cell expression matrix (e.g., a feature table or pre-computed pseudotime)

## Inputs

- Processed human hematopoietic progenitors (10X Chromium) gene expression matrix (cells x genes) from MuTrans-release Data repository
- AnnData object compatible with Scanpy/pyMuTrans
- K_cluster value (number of attractor basins) chosen via EPI, Louvain/Leiden, or biological prior

## Outputs

- Attractor basins, coarse-grained transition probabilities, and cell-cluster membership assignment from dynamical_analysis
- Cell lineage / most probable transition trajectories via MPPT or MPFT (PyEMMA-based transition path theory)
- Dynamical manifold/landscape visualization (ConstructLandscape)
- declare_speedup_comparison.csv with transition-likelihood comparison and runtime/speedup table

## How to apply

Load the processed human hematopoietic progenitors gene expression matrix from the MuTrans-release Data repository (github.com/cliffzhou92/MuTrans-release/tree/main/Data) into an AnnData object with Scanpy. Choose K_cluster for pyMuTrans dynamical_analysis using the Eigen-Peak Index (EPI), Louvain/Leiden clustering, or biological prior, then run the full multi-scale analysis to obtain attractor basins, coarse-grained transition probabilities, and PyEMMA-based transition path likelihoods. For large datasets, enable the DECLARE module with par['reduce_large_scale']=True and par['reduce_num_meta_cell']=1500 so cells are first aggregated into hundreds/thousands of micro-states via K-means or kNN partition before coarse-graining. Compare the resulting dynamical landscape and transition path likelihoods against the complete-data run and against a simple expression-averaging aggregation baseline — DECLARE should preserve the landscape structure better than expression averaging and approximate no-DECLARE transition probabilities well. Record wall-clock runtimes of both runs and compute the speedup factor, writing the comparison to declare_speedup_comparison.csv.

## Related tools

- **MuTrans** (Core multi-scale stochastic dynamics method (pyMuTrans dynamical_analysis) that learns the cell-cell rwTPM, coarse-grains to attractor basins, and quantifies transition paths) — github.com/cliffzhou92/MuTrans-release
- **DECLARE** (Pre-processing dynamics-preserving cell aggregation module (reduce_large_scale=True, reduce_num_meta_cell=1500) that aggregates cells into micro-states via K-means or kNN to speed up MuTrans on large datasets) — github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Dependency (>=2.5.6) used for transition path theory computation of transition path likelihoods) — http://emma-project.org/latest/INSTALL.html
- **Scanpy** (Used to load the processed bone-marrow expression matrix into an AnnData object for pyMuTrans analysis) — https://scanpy.readthedocs.io/en/stable/installation.html

## Examples

```
import pyMuTrans as pm; par = {"K_cluster": 5.0, "reduce_large_scale": True, "reduce_num_meta_cell": 1500}; adata = pm.dynamical_analysis(adata, par)
```

## Evaluation signals

- DECLARE and complete-data runs yield consistent transition path likelihoods (matching the reported Fig. 6d-e consistency)
- DECLARE preserves the dynamical landscape/transition path structure better than the simple expression-averaging aggregation baseline
- DECLARE speedup factor is at least 5x (reported as roughly one order of magnitude) in the wall-clock runtime comparison
- Coarse-grained transition probabilities from the DECLARE run are a good approximation to those computed without DECLARE
- Attractor basin count equals the chosen K_cluster and membership matrix assigns each cell with sensible attractor positions

## Limitations

- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- A sufficiently large number of cells is required for high-resolution identification of transition cells; small datasets like the myelopoiesis data require special care
- Detailed balance may be violated at the microscopic scale, relying on the estimated coarse-grained (mesoscopic) dynamics to recover transitions

## Evidence

- [intro] we provide an option to speed up calculation by introducing a pre-processing aggregation module DECLARE (dynamics-preserving cells aggregation): "we provide an option to speed up calculation by introducing a pre-processing aggregation module DECLARE (dynamics-preserving cells aggregation)"
- [discussion] MuTrans introduces a pre-processing method (DECLARE) to aggregate the cells and speed up computation.: "MuTrans introduces a pre-processing method (DECLARE) to aggregate the cells and speed up computation."
- [readme] par["reduce_large_scale"] = True # optional, to use DECLARE module speeding-up the calculation: "par["reduce_large_scale"] = True # optional, to use DECLARE module speeding-up the calculation"
- [readme] %% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [methods] with the processed gene expression matrices that could be loaded directly in MuTrans analysis stored at https://github.com/cliffzhou92/MuTrans-release/tree/main/Data.: "with the processed gene expression matrices that could be loaded directly in MuTrans analysis stored at https://github.com/cliffzhou92/MuTrans-release/tree/main/Data."
