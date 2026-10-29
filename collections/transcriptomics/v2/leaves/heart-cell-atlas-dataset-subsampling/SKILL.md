---
name: heart-cell-atlas-dataset-subsampling
description: 'Use when reproducing or extending the runtime scalability benchmark
  on the Heart Cell Atlas: measuring how SiFT, scVI, and regress_out runtimes scale
  with subsample size from 5,000 to ~486,134 cells.'
when_to_use_negative:
- The goal is to validate the biological correctness of filtered output rather than
  runtimes — the article notes no ground truth reference exists to evaluate computational
  filtering results, so metrics must be application-specific.
- The dataset is small or unrelated to the Heart Cell Atlas scale; runtime ordering
  on ~500,000-cell subsamples will not generalize.
- The task is defining or applying a new filtering signal (kernel choice) rather than
  benchmarking scalability of existing covariate-removal runs.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SiFT (sift-sc)
  role: 'Fastest benchmarked method: filters the 10 covariates from each Heart Cell
    Atlas subsample on GPU using sift.SiFT with a kernel built from the covariate
    mappings; installed via pip.'
  repo: https://github.com/nitzanlab/sift-sc
- name: scVI (scvi-tools)
  role: 'Comparison method: trained on each subsample for the same covariate-removal
    setting with GPU support, its wall-clock time recorded per subsample.'
  repo: https://github.com/scverse/scvi-tools
- name: scanpy regress_out
  role: 'Baseline comparison method: regresses the 10 covariates out of the expression
    matrix of each subsample (CPU), and was the slowest method in the reported benchmark.'
- name: pip
  role: 'Package installation channel for SiFT: ''pip install sift-sc'' per the methods
    section and README.'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:5181cdf209aff7aeae33985c8b414c6477e4b5b01ab62f3eaa036a98cf634437
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - pip
  - scVI (scvi-tools)
  - scanpy regress_out
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# heart-cell-atlas-dataset-subsampling

## Summary

A runtime scalability benchmark that subsamples the Heart Cell Atlas (~500,000 cells) at 8 increasing sizes (5,000 to 486,134 cells) and measures wall-clock time for removing 10 continuous covariates with SiFT, scVI, and scanpy regress_out, testing the reported ordering SiFT (12 min) < scVI (36 min) < regress_out (1 h 26 min).

## When to use

Use when reproducing or extending the runtime scalability benchmark on the Heart Cell Atlas: measuring how SiFT, scVI, and regress_out runtimes scale with subsample size from 5,000 to ~486,134 cells.

## When NOT to use

- The goal is to validate the biological correctness of filtered output rather than runtimes — the article notes no ground truth reference exists to evaluate computational filtering results, so metrics must be application-specific.
- The dataset is small or unrelated to the Heart Cell Atlas scale; runtime ordering on ~500,000-cell subsamples will not generalize.
- The task is defining or applying a new filtering signal (kernel choice) rather than benchmarking scalability of existing covariate-removal runs.

## Inputs

- Heart Cell Atlas single-cell dataset (~500,000 cells) and its 8 subsamples of 5,000, 10,000, ..., 486,134 cells
- 10 continuous random covariates to be removed as nuisance signals (covariate mappings for the SiFT kernel)
- Expression count matrix / single-cell data object compatible with SiFT, scVI (scvi-tools), and scanpy

## Outputs

- fig2c_runtimes.csv — per-method, per-subsample wall-clock runtime measurements
- Runtime versus number-of-cells plot reproducing Fig. 2c
- Full-dataset runtime comparison against the reported ordering: SiFT 12 min < scVI 36 min < regress_out 1 h 26 min

## How to apply

Load the 8 Heart Cell Atlas subsamples (5,000, 10,000, ..., 486,134 cells) together with the 10 continuous random covariates to be removed as nuisance signals. Run SiFT via sift.SiFT with a KnnKernel built from the covariate mappings (one of the kernels RBFKernel/LaplacianKernel/KnnKernel/MappingKernel in sift.kernels) to filter the covariates from each subsample on GPU, recording wall-clock time per subsample. Train scVI (scverse scvi-tools) on each subsample under the same covariate-removal setting with GPU support, and run scanpy regress_out on the expression matrix of each subsample, timing both. Assemble per-method, per-subsample runtimes into fig2c_runtimes.csv and plot runtime versus number of cells as in Fig. 2c. Decision point: on the full dataset (~500,000 cells), the expected result is SiFT ~12 min < scVI ~36 min < regress_out ~1 h 26 min, with SiFT and scVI running on GPU.

## Related tools

- **SiFT (sift-sc)** (Fastest benchmarked method: filters the 10 covariates from each Heart Cell Atlas subsample on GPU using sift.SiFT with a kernel built from the covariate mappings; installed via pip.) — https://github.com/nitzanlab/sift-sc
- **scVI (scvi-tools)** (Comparison method: trained on each subsample for the same covariate-removal setting with GPU support, its wall-clock time recorded per subsample.) — https://github.com/scverse/scvi-tools
- **scanpy regress_out** (Baseline comparison method: regresses the 10 covariates out of the expression matrix of each subsample (CPU), and was the slowest method in the reported benchmark.)
- **pip** (Package installation channel for SiFT: 'pip install sift-sc' per the methods section and README.)

## Examples

```
pip install sift-sc
```

## Evaluation signals

- fig2c_runtimes.csv contains one runtime entry for each of the 3 methods across each of the 8 subsamples
- Per-method runtime increases monotonically with the number of cells in the subsample
- Full-dataset (~486,134 cells) runtimes reproduce the ordering SiFT (~12 min) < scVI (~36 min) < regress_out (~1 h 26 min)
- SiFT and scVI runs are confirmed to execute with GPU support, matching the reported benchmark conditions
- Each subsample is processed with the same 10 continuous covariates removed by all three methods, so the comparison is like-for-like

## Limitations

- Reported runtimes are hardware-dependent: SiFT and scVI were run with GPU support while regress_out runs on CPU, so reproduction requires a comparable GPU setup for the ordering to hold.
- SiFT lacks a ground truth reference to evaluate filtering results; integration metrics are application-specific.
- SiFT strongly relies on prior knowledge used to define the cell-cell similarity kernel — an inaccurate or coarse-grained mapping may lead to a loss of signal.
- SiFT modifies gene expression so the filtered data are no longer properly log-transformed, which matters for some downstream tasks.

## Evidence

- [abstract] SiFT performs the correction at the gene expression level, can scale to large datasets, and compares favorably to state-of-the-art methods: "SiFT performs the correction at the gene expression level, can scale to large datasets, and compares favorably to state-of-the-art methods"
- [intro] we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter: "we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter"
- [methods] Install `SiFT` in one of the following ways: Through **pip**:: pip install sift-sc: "Install `SiFT` in one of the following ways: Through **pip**:: pip install sift-sc"
- [discussion] We have made SiFT available as an open-source python package along with documentation and tutorials and ensured it can efficiently scale: "We have made SiFT available as an open-source python package along with documentation and tutorials and ensured it can efficiently scale"
- [readme] Signal FilTering is a tool for uncovering hidden biological processes in single-cell data.: "Signal FilTering is a tool for uncovering hidden biological processes in single-cell data."
