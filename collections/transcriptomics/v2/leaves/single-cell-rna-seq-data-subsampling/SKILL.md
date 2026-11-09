---
name: single-cell-rna-seq-data-subsampling
description: 'Use when benchmarking scalability of a single-cell method: you have
  a cell x gene expression matrix (e.g., PBMC Multiome from the Human Blood Atlas)
  and need runtime vs. cell-number measurements on nested subsamples.'
when_to_use_negative:
- The goal is accuracy/performance benchmarking of GRN inference rather than runtime
  scaling — use the method's accuracy evaluation workflow instead
- Data are true longitudinal time series or already pseudo-time-ordered trajectories;
  this skill targets time-stamped cross-sectional (TSCS) single-cell data where cells
  at different times come from independent batches
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: COSLIR (Python version)
  role: Method whose gene regulatory network inference is executed on each subsample
    to obtain wall-clock timings
  repo: https://github.com/Ge-lab-pku/COSLIR
- name: NumPy
  role: Loads the expression matrix and generates random subsamples of cells at increasing
    cell numbers
- name: Python
  role: Runtime environment for the subsampling loop and timing measurements
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:c0e1baf70c9149a5fcc00f5de652c1e8cddf37627d24f26aa89387c886eca183
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ge-lab-pku/COSLIR
  tools:
  - COSLIR (Python version)
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# single-cell-rna-seq-data-subsampling

## Summary

Generates subsampled versions of a single-cell expression matrix at a series of increasing cell counts and times a method (here, the Python version of COSLIR) on each to characterize how running time scales with the number of cells, as done to reproduce the near-flat runtime curve of Fig. 5C.

## When to use

Use when benchmarking scalability of a single-cell method: you have a cell x gene expression matrix (e.g., PBMC Multiome from the Human Blood Atlas) and need runtime vs. cell-number measurements on nested subsamples.

## When NOT to use

- The goal is accuracy/performance benchmarking of GRN inference rather than runtime scaling — use the method's accuracy evaluation workflow instead
- Data are true longitudinal time series or already pseudo-time-ordered trajectories; this skill targets time-stamped cross-sectional (TSCS) single-cell data where cells at different times come from independent batches

## Inputs

- Single-cell expression matrix (e.g., PBMC Multiome / Peripheral Blood Mononuclear Cell Multiome from the Human Blood Atlas), loadable with NumPy
- Series of target cell counts for subsampling

## Outputs

- Per-subsample wall-clock running times recorded against cell counts
- runtime_vs_cells.csv (table of cell count vs. runtime)
- Scalability plot of runtime vs. number of cells (corresponding to Fig. 5C)

## How to apply

Load the single-cell RNA-seq/multiome expression matrix (e.g., the Peripheral Blood Mononuclear Cell Multiome dataset from the Human Blood Atlas) into a NumPy array. Generate subsampled versions at a series of increasing cell numbers, e.g., by randomly selecting row indices without replacement for each target count. For each subsample, run the method under test — COSLIR (Python version, tested on Python v3.6.4 with NumPy v1.14) — to infer the gene regulatory network, and record wall-clock running time. Aggregate cell count vs. runtime into a table (runtime_vs_cells.csv) and plot runtime against number of cells. The expected result, and the validation of the analysis, is a nearly flat curve: for COSLIR, running time remained nearly independent of the number of cells because the method relies only on the first and second moments of samples between two consecutive time points rather than per-cell trajectories.

## Related tools

- **COSLIR (Python version)** (Method whose gene regulatory network inference is executed on each subsample to obtain wall-clock timings) — https://github.com/Ge-lab-pku/COSLIR
- **NumPy** (Loads the expression matrix and generates random subsamples of cells at increasing cell numbers)
- **Python** (Runtime environment for the subsampling loop and timing measurements)

## Examples

```
import numpy as np; X = np.load('pbmc_multiome.npy'); idx = np.random.choice(X.shape[0], n_cells, replace=False); X_sub = X[idx, :]
```

## Evaluation signals

- The runtime-vs-cell-count curve remains nearly flat, matching the reported property that COSLIR's running time remained nearly independent of the number of cells
- runtime_vs_cells.csv contains one runtime entry per subsample, keyed by the exact cell count used
- Subsample sizes form a monotonically increasing series and each subsample is drawn without replacement from the full expression matrix
- The Python environment matches the tested configuration (Python v3.6.4, NumPy v1.14) so timings are comparable to the reported Fig. 5C setup

## Limitations

- COSLIR is explicitly a beta version; 'Some bugs may still remain, so please use COSLIR with caution at present', so timings may vary across versions
- Subsampling introduces sampling noise, so each timing reflects a particular random draw rather than the full atlas density
- The reported near-independence of runtime applies to COSLIR's moment-based computation (first and second moments), not necessarily to other GRN methods

## Evidence

- [abstract] runtime_independence_finding: "Notably, its running time remained nearly independent of the number of cells."
- [readme] runtime_environment: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)."
- [abstract] moment_basis: "utilizing only the first and second moments of samples between two consecutive time points"
- [methods] usage_docs: "Just go to the corresponding folder for your preferred version and check the DESCRIPTION files there for the details of usage"
