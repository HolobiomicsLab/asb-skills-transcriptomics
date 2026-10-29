---
name: runtime-scalability-benchmarking-of-single-cell-methods
description: Use when reproducing or extending runtime scalability comparisons of
  single-cell correction methods across increasing subsample sizes of a large dataset
  (e.g., ~500,000-cell Heart Cell Atlas).
when_to_use_negative:
- The goal is to assess filtering quality or biological signal preservation rather
  than computational scaling — use integration metrics or ground-truth-based evaluation
  instead.
- The dataset is small enough that runtime differences among methods are negligible
  and the benchmark is uninformative.
- Methods operate at fundamentally different levels (e.g., joint latent representation
  vs. gene expression level) and the goal is accuracy comparison, not timing.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SiFT (sift-sc)
  role: Primary method under benchmark; filters the 10 covariates via a KnnKernel
    built from the covariate mappings, run on GPU with wall-clock time recorded per
    subsample
  repo: https://github.com/nitzanlab/sift-sc
- name: scVI (scvi-tools)
  role: Baseline method trained per subsample for the same covariate-removal setting
    on GPU; runtime recorded per subsample
  repo: https://github.com/scverse/scvi-tools
- name: Python
  role: Implementation language for the benchmark harness, timing capture, table assembly
    (fig2c_runtimes.csv) and Fig. 2c plot
- name: pip
  role: Installs the sift-sc package (pip install sift-sc) used for the SiFT runtime
    runs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:1380423e53d092834a3bbc8419f4c03c19aca27cab6794e5288c5daedb59491f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# runtime-scalability-benchmarking-of-single-cell-methods

## Summary

Benchmarking skill for measuring how the wall-clock runtime of single-cell filtering/correction methods (SiFT, scVI, scanpy regress_out) scales with the number of cells, using the Heart Cell Atlas subsample series to verify the reported ordering SiFT < scVI < regress_out.

## When to use

Use when reproducing or extending runtime scalability comparisons of single-cell correction methods across increasing subsample sizes of a large dataset (e.g., ~500,000-cell Heart Cell Atlas).

## When NOT to use

- The goal is to assess filtering quality or biological signal preservation rather than computational scaling — use integration metrics or ground-truth-based evaluation instead.
- The dataset is small enough that runtime differences among methods are negligible and the benchmark is uninformative.
- Methods operate at fundamentally different levels (e.g., joint latent representation vs. gene expression level) and the goal is accuracy comparison, not timing.

## Inputs

- 8 Heart Cell Atlas subsamples ranging from 5,000 to 486,134 cells (full dataset ~500,000 cells)
- 10 continuous random covariates defined as nuisance signals to be filtered/regressed out
- GPU-supported environment for SiFT and scVI runs

## Outputs

- fig2c_runtimes.csv: per-method, per-subsample wall-clock runtime table
- Runtime-vs-number-of-cells plot reproducing Fig. 2c
- Full-dataset runtime comparison against the reported ~12 min (SiFT) < ~36 min (scVI) < ~1 h 26 min (regress_out) ordering

## How to apply

Load the 8 Heart Cell Atlas subsamples (5,000 up to 486,134 cells) together with the 10 continuous random covariates to be removed as nuisance signals. Run SiFT (sift-sc, e.g. sift.SiFT with a KnnKernel built from the covariate mappings) on GPU, train scVI via scvi-tools on GPU for the same covariate-removal setting, and run scanpy regress_out on CPU, recording wall-clock time per subsample and per method. Rationale: SiFT performs correction at the gene expression level and was engineered to scale to large datasets, so the benchmark tests whether its measured runtime stays below scVI (~36 min) and regress_out (~1 h 26 min) on the full ~500,000-cell dataset, where SiFT was reported at ~12 min. Assemble per-method, per-subsample timings into fig2c_runtimes.csv and plot runtime versus number of cells as in Fig. 2c.

## Related tools

- **SiFT (sift-sc)** (Primary method under benchmark; filters the 10 covariates via a KnnKernel built from the covariate mappings, run on GPU with wall-clock time recorded per subsample) — https://github.com/nitzanlab/sift-sc
- **scVI (scvi-tools)** (Baseline method trained per subsample for the same covariate-removal setting on GPU; runtime recorded per subsample) — https://github.com/scverse/scvi-tools
- **Python** (Implementation language for the benchmark harness, timing capture, table assembly (fig2c_runtimes.csv) and Fig. 2c plot)
- **pip** (Installs the sift-sc package (pip install sift-sc) used for the SiFT runtime runs)

## Examples

```
pip install sift-sc; kernel = sift.kernels.KnnKernel(mapping=covariates); sift.SiFT(kernel=kernel).filter(adata)
```

## Evaluation signals

- Full-dataset runtimes reproduce the ordering SiFT (~12 min) < scVI (~36 min) < regress_out (~1 h 26 min)
- Wall-clock time is recorded for every (method, subsample) pair across all 8 subsamples (5,000–486,134 cells)
- fig2c_runtimes.csv contains one runtime row per method per subsample with no missing timing entries
- Runtime-versus-cells curve is monotonically non-decreasing for each method, as expected for scaling behavior
- SiFT and scVI runs are verified to use GPU support, matching the reported benchmark configuration

## Limitations

- SiFT and scVI were benchmarked with GPU support whereas regress_out was not, so the comparison mixes hardware configurations
- Runtime results are hardware- and version-dependent and may not transfer to other CPUs/GPUs or package versions
- SiFT lacks a ground truth reference for evaluating filtering results; this skill validates only runtime, not filtering quality

## Evidence

- [abstract] scaling claim: "SiFT performs the correction at the gene expression level, can scale to large datasets, and compares favorably to state-of-the-art methods"
- [discussion] package availability and scaling: "We have made SiFT available as an open-source python package along with documentation and tutorials and ensured it can efficiently scale"
- [methods] installation via pip: "Through **pip**::

    pip install sift-sc"
- [readme] README installation: "Install SiFT via PyPI by running::

    pip install sift-sc"
- [readme] scvi-tools GPU capability: "includes standard save/load functions, GPU
acceleration, etc."
