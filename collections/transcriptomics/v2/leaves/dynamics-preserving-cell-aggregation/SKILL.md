---
name: dynamics-preserving-cell-aggregation
description: Use when running MuTrans dynamical_analysis on large datasets (e.g.,
  10x Chromium human bone-marrow hematopoiesis with tens of thousands of cells) where
  full cell-cell rwTPM computation is too slow, and approximate transition probabilities
  are acceptable.
when_to_use_negative:
- Dataset is small (e.g., the Fluidigm C1 myelopoiesis data), where special care is
  already needed and the full-data MuTrans run is tractable without aggregation
- Exact cell-level rwTPM transition probabilities are required rather than an approximation,
  since DECLARE trades exactness for speed
- The analysis goal is simple gene-expression clustering rather than cell-fate dynamical
  modeling, where expression averaging would suffice
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MuTrans
  role: 'Host framework: DECLARE is a pre-processing module of MuTrans that feeds
    aggregated micro-states into the multi-scale dynamical analysis (pyMuTrans dynamical_analysis)'
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: DECLARE
  role: The dynamics-preserving cell aggregation module itself, aggregating cells
    into micro-states via K-means or kNN partition
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Used for transition path theory computation of transition path likelihoods
    used to validate that DECLARE preserves transition probabilities
  repo: http://emma-project.org/latest/INSTALL.html
- name: Scanpy
  role: Used to load the processed 10x bone-marrow gene expression matrix into an
    AnnData object consumed by pyMuTrans
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:84a9a82a3f6cb31ab56077d5404aa46f3578e7e41865878130b7178706de2ebc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - DECLARE
  - MuTrans
  - PyEMMA
  - Scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# dynamics-preserving-cell-aggregation

## Summary

DECLARE is the pre-processing aggregation module of MuTrans that compresses large single-cell datasets into hundreds to thousands of micro-states via K-means or kNN partition while preserving the stochastic dynamical landscape, enabling scalable multi-scale analysis with roughly one order-of-magnitude speedup.

## When to use

Use when running MuTrans dynamical_analysis on large datasets (e.g., 10x Chromium human bone-marrow hematopoiesis with tens of thousands of cells) where full cell-cell rwTPM computation is too slow, and approximate transition probabilities are acceptable.

## When NOT to use

- Dataset is small (e.g., the Fluidigm C1 myelopoiesis data), where special care is already needed and the full-data MuTrans run is tractable without aggregation
- Exact cell-level rwTPM transition probabilities are required rather than an approximation, since DECLARE trades exactness for speed
- The analysis goal is simple gene-expression clustering rather than cell-fate dynamical modeling, where expression averaging would suffice

## Inputs

- Processed single-cell gene expression matrix (N_cells x N_genes) as an AnnData object, e.g., human hematopoietic progenitors 10X Chromium bone-marrow data from the MuTrans-release Data repository
- K_cluster parameter (number of attractors) selected via Eigen-Peak Index or biological prior
- reduce_num_meta_cell parameter (e.g., 1500 micro-states)

## Outputs

- Attractor basins and cell-cluster membership assignment aggregated at the micro-state level
- Coarse-grained transition probabilities among micro-states approximating the complete-data transition probabilities
- Transition path likelihoods (via PyEMMA-based transition path theory) comparable to the full-data run
- Runtime measurements supporting the speedup comparison (e.g., declare_speedup_comparison.csv)

## How to apply

Load the processed gene expression matrix (e.g., the Setty et al. human bone-marrow 10x data) into an AnnData object with Scanpy and first select the number of attractors K_cluster via the Eigen-Peak Index (EPI) or biological/Louvain prior. Enable DECLARE in pyMuTrans by setting par['reduce_large_scale']=True and par['reduce_num_meta_cell']=1500, which aggregates cells into micro-states before the multi-scale reduction of the random walk transition probability matrix (rwTPM), then run pm.dynamical_analysis(adata,par). Rationale: unlike naive averaging of gene expression profiles of small cell groups, dynamics-preserving aggregation better retains the attractor-basin structure and gives a good approximation to the coarse-grained transition probabilities and transition path likelihoods (PyEMMA-based transition path theory) computed on the complete data. Validate by comparing the dynamical manifold and transition path likelihoods between the complete-data run and the DECLARE run, and against an expression-averaging baseline; record wall-clock times to confirm a speedup of at least 5x (reported as roughly one order of magnitude).

## Related tools

- **MuTrans** (Host framework: DECLARE is a pre-processing module of MuTrans that feeds aggregated micro-states into the multi-scale dynamical analysis (pyMuTrans dynamical_analysis)) — https://github.com/cliffzhou92/MuTrans-release
- **DECLARE** (The dynamics-preserving cell aggregation module itself, aggregating cells into micro-states via K-means or kNN partition) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Used for transition path theory computation of transition path likelihoods used to validate that DECLARE preserves transition probabilities) — http://emma-project.org/latest/INSTALL.html
- **Scanpy** (Used to load the processed 10x bone-marrow gene expression matrix into an AnnData object consumed by pyMuTrans) — https://scanpy.readthedocs.io/en/stable/installation.html

## Examples

```
par = {"K_cluster": 5.0, "reduce_large_scale": True, "reduce_num_meta_cell": 1500}; adata = pm.dynamical_analysis(adata, par)
```

## Evaluation signals

- Transition path likelihoods from the DECLARE run are consistent with the complete-data run (matching the reported Fig. 6d-e consistency)
- DECLARE preserves the dynamical landscape / manifold structure better than the simple expression-averaging baseline
- Coarse-grained transition probabilities with DECLARE approximate those computed without DECLARE
- Wall-clock runtime shows a speedup of at least 5x, reported as roughly one order of magnitude
- The number of micro-states equals reduce_num_meta_cell (e.g., 1500) and each cell is assigned to exactly one micro-state

## Limitations

- Approximation quality depends on the number of micro-states; very small reduce_num_meta_cell may distort the landscape
- MuTrans/DECLARE results on relatively small datasets (e.g., myelopoiesis) require special care and further confirmation
- Detailed balance may be violated at the microscopic scale (Eq. 1); recovery of transitions relies on the coarse-grained dynamics
- A sufficiently large number of cells is needed for high-resolution identification of transition cells

## Evidence

- [intro] we provide an option to speed up calculation by introducing a pre-processing aggregation module DECLARE (dynamics-preserving cells aggregation): "we provide an option to speed up calculation by introducing a pre-processing aggregation module DECLARE (dynamics-preserving cells aggregation)"
- [discussion] MuTrans introduces a pre-processing method (DECLARE) to aggregate the cells and speed up computation.: "MuTrans introduces a pre-processing method (DECLARE) to aggregate the cells and speed up computation."
- [readme] par["reduce_large_scale"] = True # optional, to use DECLARE module speeding-up the calculation: "par["reduce_large_scale"] = True # optional, to use DECLARE module speeding-up the calculation
par["reduce_num_meta_cell"] = 1500 # optional, to set the number of microsopic meta-stable states in"
- [methods] with the processed gene expression matrices that could be loaded directly in MuTrans analysis stored at https://github.com/cliffzhou92/MuTrans-release/tree/main/Data.: "with the processed gene expression matrices that could be loaded directly in MuTrans analysis stored at https://github.com/cliffzhou92/MuTrans-release/tree/main/Data."
