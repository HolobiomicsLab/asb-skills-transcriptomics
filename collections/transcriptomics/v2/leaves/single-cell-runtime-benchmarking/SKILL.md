---
name: single-cell-runtime-benchmarking
description: Use when a single-cell trajectory/dynamics method offers a pre-processing
  aggregation option for large datasets and you must quantify runtime speedup while
  verifying that downstream dynamical quantities (landscape, transition probabilities,
  path likelihoods) are preserved.
when_to_use_negative:
- The dataset is small (e.g., the Fluidigm C1 myelopoiesis data), where DECLARE aggregation
  is unnecessary and small-cell-count analyses need special care
- The scientific question requires cell-level resolution of transition cells rather
  than micro-state-level approximation, since DECLARE collapses cells into ~1500 meta-cells
- The method being benchmarked has no pre-processing/aggregation option, so no speedup-fidelity
  tradeoff can be measured
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: MuTrans
  role: Multi-scale stochastic dynamics method whose complete-data run provides the
    reference transition probabilities and landscape against which the DECLARE speedup
    is benchmarked
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: DECLARE
  role: Dynamics-preserving cell-aggregation pre-processing module (reduce_large_scale=True,
    reduce_num_meta_cell=1500) being benchmarked for runtime speedup and fidelity
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Used within MuTrans for transition path theory computation of transition path
    likelihoods compared across full-data and DECLARE runs
  repo: http://emma-project.org/latest/INSTALL.html
- name: Scanpy
  role: Loads the processed 10X Chromium bone-marrow gene expression matrix into the
    AnnData object passed to pm.dynamical_analysis
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:2fd7d006e12892db0fdd9c567783a02b8980543d7d5dd2e31453793416937ca6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - DECLARE
  - MuTrans
  - PyEMMA
  - Scanpy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-runtime-benchmarking

## Summary

Benchmarking protocol for measuring the wall-clock speedup and fidelity preservation of the DECLARE dynamics-preserving cell-aggregation pre-processing module in MuTrans on large single-cell datasets, using the full-data MuTrans run as ground truth.

## When to use

Use when a single-cell trajectory/dynamics method offers a pre-processing aggregation option for large datasets and you must quantify runtime speedup while verifying that downstream dynamical quantities (landscape, transition probabilities, path likelihoods) are preserved.

## When NOT to use

- The dataset is small (e.g., the Fluidigm C1 myelopoiesis data), where DECLARE aggregation is unnecessary and small-cell-count analyses need special care
- The scientific question requires cell-level resolution of transition cells rather than micro-state-level approximation, since DECLARE collapses cells into ~1500 meta-cells
- The method being benchmarked has no pre-processing/aggregation option, so no speedup-fidelity tradeoff can be measured

## Inputs

- Processed human hematopoietic progenitors (10X Chromium) gene expression matrix from MuTrans-release Data repository, loaded as an AnnData object
- MuTrans parameter dictionary (K_cluster, reduce_large_scale, reduce_num_meta_cell)
- Control aggregation setting using simple expression averaging

## Outputs

- Comparison of transition path likelihoods between complete-data, DECLARE, and expression-averaging runs
- Wall-clock runtime table and computed speedup factor (e.g., declare_speedup_comparison.csv)
- Coarse-grained transition probabilities and attractor basins from each run for manifold-structure comparison

## How to apply

Run the complete-data multi-scale MuTrans analysis (pyMuTrans dynamical_analysis, K_cluster set via Eigen-Peak Index or biological prior) on the full human hematopoietic progenitors (10X Chromium) bone-marrow matrix to obtain attractor basins, coarse-grained transition probabilities, and PyEMMA-based transition path likelihoods. Re-run with the DECLARE module enabled (par['reduce_large_scale']=True, par['reduce_num_meta_cell']=1500), which aggregates cells into micro-states via K-means/kNN partition before coarse-graining. Also run a control baseline in which aggregation is done by simple expression averaging of small cell groups. Compare transition path likelihoods and dynamical manifold structure (Fig. 6d-e style) across the three runs, record wall-clock times, and compute the speedup factor; validate that DECLARE approximates the no-DECLARE transition probabilities better than expression averaging and achieves at least ~5x (reported roughly one order of magnitude) speedup.

## Related tools

- **MuTrans** (Multi-scale stochastic dynamics method whose complete-data run provides the reference transition probabilities and landscape against which the DECLARE speedup is benchmarked) — https://github.com/cliffzhou92/MuTrans-release
- **DECLARE** (Dynamics-preserving cell-aggregation pre-processing module (reduce_large_scale=True, reduce_num_meta_cell=1500) being benchmarked for runtime speedup and fidelity) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Used within MuTrans for transition path theory computation of transition path likelihoods compared across full-data and DECLARE runs) — http://emma-project.org/latest/INSTALL.html
- **Scanpy** (Loads the processed 10X Chromium bone-marrow gene expression matrix into the AnnData object passed to pm.dynamical_analysis) — https://scanpy.readthedocs.io/en/stable/installation.html

## Examples

```
import pyMuTrans as pm; par = {"K_cluster": 5.0, "reduce_large_scale": True, "reduce_num_meta_cell": 1500}; adata = pm.dynamical_analysis(adata, par)
```

## Evaluation signals

- DECLARE run yields transition path likelihoods consistent with the complete-data run (matching reported Fig. 6d-e consistency)
- DECLARE preserves the dynamical landscape/manifold structure better than the simple expression-averaging baseline
- Measured speedup factor is at least 5x (paper reports roughly one order of magnitude)
- Coarse-grained transition probabilities from DECLARE closely approximate those computed without DECLARE
- Runtime table (declare_speedup_comparison.csv) records wall-clock times for both runs with a computed speedup ratio

## Limitations

- Speedup and fidelity figures are grounded in the human bone-marrow hematopoiesis dataset; behavior on other datasets/platforms is not guaranteed by this benchmark
- DECLARE aggregates cells into micro-states, trading cell-level resolution for runtime
- When the number of cells is relatively small, such as in the myelopoiesis dataset, special care is needed, and the aggregation benchmark is not the appropriate focus

## Evidence

- [intro] we provide an option to speed up calculation by introducing a pre-processing aggregation module DECLARE (dynamics-preserving cells aggregation): "we provide an option to speed up calculation by introducing a pre-processing aggregation module DECLARE (dynamics-preserving cells aggregation)"
- [readme] par["reduce_large_scale"] = True: "par["reduce_large_scale"] = True # optional, to use DECLARE module speeding-up the calculation"
- [discussion] MuTrans introduces a pre-processing method (DECLARE) to aggregate the cells and speed up computation.: "MuTrans introduces a pre-processing method (DECLARE) to aggregate the cells and speed up computation."
- [readme] %% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
- [methods] Pre-Processing By Declare And Scalability To Large Datasets: "## Methods > Pre-Processing By Declare And Scalability To Large Datasets"
