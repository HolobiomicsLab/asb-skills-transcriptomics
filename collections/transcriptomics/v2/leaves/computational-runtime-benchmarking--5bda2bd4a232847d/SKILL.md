---
name: computational-runtime-benchmarking
description: Use when a new method must be timed against baseline competitors on the
  same single-cell dataset, across data-size subsamples, and on a scaled-up simulated
  dataset to quantify speedup factors.
when_to_use_negative:
- The question is about statistical accuracy, sensitivity, or false-positive control
  of the method rather than computational cost
- The competing methods cannot be run on identical inputs (different data formats
  or normalization requirements), so the timing comparison would be invalid
tools:
- name: scDist
  role: Method under benchmark; timed against Augur and nDEG on COVID-19 data, subsamples,
    and 1M-cell simulated data
  repo: https://github.com/phillipnicol/scDist
- name: Augur
  role: Baseline competitor timed for comparison (~5 min vs scDist ~50 s on 13-sample
    COVID-19 data)
- name: nDEG
  role: Gene-wise differential testing baseline; pseudobulk aggregation followed by
    gene-wise testing, >60x slower than scDist
- name: simData
  role: scDist function used to generate the 1,000,000-cell simulated benchmark dataset
    (nct=10, N1=5, N2=5)
  repo: https://github.com/phillipnicol/scDist
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:1a52b99281dc3b1e29482a41fb75e55bfa6a12158cc37c4ab3208ae1b223a94c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/phillipnicol/scDist
  tools:
  - Augur
  - Seurat
  - ggplot2
  - irlba
  - lme4
  - lmerTest
  - nDEG
  - scDist
  - simData
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# computational-runtime-benchmarking

## Summary

A protocol for rigorously benchmarking the wall-clock runtime and speedup of a computational method (here scDist, a mixed-effects tool for detecting perturbed cell types in scRNA-seq) against competing methods (Augur, nDEG) on a fixed dataset and on controlled subsamples of increasing size.

## When to use

Use when a new method must be timed against baseline competitors on the same single-cell dataset, across data-size subsamples, and on a scaled-up simulated dataset to quantify speedup factors.

## When NOT to use

- The question is about statistical accuracy, sensitivity, or false-positive control of the method rather than computational cost
- The competing methods cannot be run on identical inputs (different data formats or normalization requirements), so the timing comparison would be invalid

## Inputs

- Normalized scRNA-seq expression matrix (genes x cells) with per-cell metadata (condition, patient, cluster) from a 13-sample COVID-19 dataset
- Size-varied subsamples of the same dataset for scaling benchmarks
- Simulated dataset from simData(nct=10, N1=5, N2=5) up to 1,000,000 cells
- Baseline method implementations: Augur and nDEG pseudobulk differential testing

## Outputs

- runtime_benchmark.csv with per-method wall-clock timings and speedup ratios
- Reported speedup factors: scDist ~50 s vs Augur ~5 min, ~5-fold average speedup across subsamples, ~10 min for 1M cells, >60x over nDEG

## How to apply

Install the method from GitHub (`devtools::install_github("phillipnicol/scDist")`) and run all competing methods on identical inputs — the 13-sample COVID-19 scRNA-seq dataset with fixed.effects = condition, random.effects = patient, clusters = cell type. Wrap each run in a wall-clock timer and record timings in a single table (e.g. runtime_benchmark.csv). Then generate size-varied subsamples of the same dataset, rerun all methods on each, and compute the average speedup (~5-fold for scDist over Augur). For scale-up, generate a 1,000,000-cell simulated dataset with simData(nct=10, N1=5, N2=5) and time the method again. Speedup validation compares the recorded ratios: scDist ~50 s vs Augur ~5 min on full data, ~10 min for 1M cells, and >60x faster than gene-wise nDEG pseudobulk differential testing.

## Related tools

- **scDist** (Method under benchmark; timed against Augur and nDEG on COVID-19 data, subsamples, and 1M-cell simulated data) — https://github.com/phillipnicol/scDist
- **Augur** (Baseline competitor timed for comparison (~5 min vs scDist ~50 s on 13-sample COVID-19 data))
- **nDEG** (Gene-wise differential testing baseline; pseudobulk aggregation followed by gene-wise testing, >60x slower than scDist)
- **simData** (scDist function used to generate the 1,000,000-cell simulated benchmark dataset (nct=10, N1=5, N2=5)) — https://github.com/phillipnicol/scDist

## Examples

```
library(scDist); system.time(out <- scDist(sim$Y, sim$meta.data, fixed.effects="response", random.effects="patient", clusters="clusters"))
```

## Evaluation signals

- Timing table shows scDist at ~50 seconds vs Augur at ~5 minutes on the full 13-sample COVID-19 dataset
- Average speedup of scDist over Augur across size-varied subsamples is approximately 5-fold
- scDist runtime on the 1,000,000-cell simData dataset is approximately 10 minutes
- scDist runtime is at least 60x shorter than the nDEG gene-wise approach on the same data
- All wall-clock timings and speedup ratios are compiled into a single runtime_benchmark.csv

## Limitations

- Runtime results depend on the normalization used; the article recommends scTransform Pearson residuals, and performance could be affected if the data were normalized by a sub-optimal approach
- nDEG runtime scales with cells per cell type (nDEG counts grow confounded when resampling 100-10,000 cells), so timing comparisons must hold the subsample grid fixed and preserve patient-level grouping
- Reported speedups are specific to the COVID-19 13-sample dataset and simulated data sizes and may not transfer to other cohort sizes or hardware

## Evidence

- [discussion] computationally fast method: "We present scDist, a statistically rigorous and computationally fast method for detecting cell-type specific differences across multiple groups or conditions."
- [readme] install and simulate: "From the R console, `devtools::install_github("phillipnicol/scDist")`."
- [readme] simulated benchmark data: "Generate simulated data with 10 cell types and 5 patients in each group"
- [methods] scDist model invocation: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
- [results] nDEG confounding rationale: [evidence span withheld — anchor status: not_found]
