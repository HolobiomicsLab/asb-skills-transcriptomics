---
name: multicore-parallel-scaling-analysis
description: Use when a compute-bound step (matrix multiplication, JIVE decomposition/rank
  estimation via bic.jive or jive.perm with a cores argument) dominates runtime and
  you need to quantify speedup across core counts 1/2/4/8 against a single-threaded
  baseline.
when_to_use_negative:
- The routine is not compute-bound (e.g., small matrices or few iterations), so OpenMP
  thread-spawn overhead can exceed gains and the benchmark measures noise rather than
  scaling.
- The goal is statistical accuracy or decomposition quality (joint/individual ranks)
  rather than runtime — use the JIVE decomposition skill instead.
- OpenMP or a multicore BLAS is unavailable on the target system, making a cores sweep
  meaningless.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Benchmark harness environment; generates fixed-seed inputs and times the baseline
    %*% operator
- name: Rcpp
  role: Compiles the C++ matrix-multiplication backends used in the timed sweep
- name: RcppEigen
  role: Parallel matrix-multiplication backend whose OpenMP core counts (1, 2, 4,
    8) are swept to measure scaling
- name: RcppArmadillo
  role: Alternative C++ matrix-multiplication backend timed for comparison against
    RcppEigen and %*%
- name: scJIVE
  role: Target routine whose computationally efficient multicore implementation exposes
    cores arguments (bic.jive, jive.perm) that can be scaled and benchmarked
  repo: github.com/oconnell-statistics-lab/scJIVE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:6847638060039670e7d918debd7d1e348cb42aa41dc32f26ccf558b8d480b0d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Rcpp
  - RcppArmadillo
  - RcppEigen
  - scJIVE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# multicore-parallel-scaling-analysis

## Summary

Benchmarks the runtime of a compute-bound R/C++ routine (e.g., matrix multiplication via %*%, RcppArmadillo, RcppEigen, or rank-selection calls like bic.jive/jive.perm that accept a cores argument) as OpenMP core count varies, reporting mean runtimes and percentage speedups relative to a single-threaded baseline.

## When to use

Use when a compute-bound step (matrix multiplication, JIVE decomposition/rank estimation via bic.jive or jive.perm with a cores argument) dominates runtime and you need to quantify speedup across core counts 1/2/4/8 against a single-threaded baseline.

## When NOT to use

- The routine is not compute-bound (e.g., small matrices or few iterations), so OpenMP thread-spawn overhead can exceed gains and the benchmark measures noise rather than scaling.
- The goal is statistical accuracy or decomposition quality (joint/individual ranks) rather than runtime — use the JIVE decomposition skill instead.
- OpenMP or a multicore BLAS is unavailable on the target system, making a cores sweep meaningless.

## Inputs

- 1000x1000 standard-normal matrices generated in R with a fixed random seed (reproducible benchmark input)
- Multi-source data list accepted by bic.jive/jive.perm (e.g., BRCA_data or SimData) for timing the cores parameter sweep
- OpenMP core-count grid: 1, 2, 4, 8 cores

## Outputs

- runtime_benchmark.csv with one column/field per implementation and core configuration
- Mean runtime per implementation per core count (over 100 repetitions)
- Percentage speedup (runtime reduction) relative to the %*% or cores=1 baseline

## How to apply

Generate reproducible inputs with a fixed random seed (e.g., 1000x1000 standard-normal matrices in R, or the SimData/BRCA_data objects accepted by bic.jive). Time the single-threaded baseline first — R's built-in %*% operator, or the same call with cores=1 — over 100 repetitions to control for JIT/IO noise. Then sweep OpenMP parallelism at 1, 2, 4, and 8 cores (100 repetitions per configuration), either for C++ matrix-multiplication backends compiled via Rcpp or for r.jive/scJIVE functions that expose a cores parameter (bic.jive, jive.perm). Compute mean runtime per implementation per core count and percentage runtime reduction relative to the baseline; the reference result is a monotone speedup rising from 67.5% runtime reduction at one core to 92.9% at eight cores. Write mean runtimes and speedup percentages to runtime_benchmark.csv with one column/field per implementation and core configuration so the scaling curve is machine-checkable.

## Related tools

- **R** (Benchmark harness environment; generates fixed-seed inputs and times the baseline %*% operator)
- **Rcpp** (Compiles the C++ matrix-multiplication backends used in the timed sweep)
- **RcppEigen** (Parallel matrix-multiplication backend whose OpenMP core counts (1, 2, 4, 8) are swept to measure scaling)
- **RcppArmadillo** (Alternative C++ matrix-multiplication backend timed for comparison against RcppEigen and %*%)
- **scJIVE** (Target routine whose computationally efficient multicore implementation exposes cores arguments (bic.jive, jive.perm) that can be scaled and benchmarked) — github.com/oconnell-statistics-lab/scJIVE

## Examples

```
bic.jive(SimData, showProgress=TRUE, cores=8)  # sweep cores in {1,2,4,8} and record mean timings to runtime_benchmark.csv
```

## Evaluation signals

- runtime_benchmark.csv exists and contains a field for each implementation (baseline, RcppArmadillo, RcppEigen) and each core configuration (1, 2, 4, 8).
- The reported 8-core speedup value lies within the 90-95% range (paper reports 92.9% shorter runtime).
- Speedup is monotonically non-decreasing with core count (67.5% at 1 core up to 92.9% at 8 cores).
- Each configuration was timed over 100 repetitions, and per-cell values are means rather than single observations.
- Inputs were generated with a fixed random seed, so re-running the benchmark reproduces the same matrices and comparable timings.

## Limitations

- Baseline JIVE was originally designed for bulk sequencing data and is computationally infeasible for large-scale single-cell datasets, so single-threaded reference timings for such data may be unobtainable.
- Reported speedups are specific to the OpenMP parallelism levels tested (1, 2, 4, 8 cores); results will differ with different hardware, BLAS backends, or thread counts.
- Speedup percentages are runtime reductions relative to R's %*% baseline, not absolute throughputs, and depend on the 1000x1000 benchmark size.

## Evidence

- [readme] We enhance JIVE for large-scale scRNA-seq data by boosting its computational efficiency and tailoring it to the single-cell context.: "We enhance JIVE for large-scale scRNA-seq data by boosting its computational efficiency and tailoring it to the single-cell context."
- [methods] bic.jive exposes a cores argument: "bic.jive(data, n, d, conv, maxiter, orthIndiv, showProgress, cores)"
- [methods] jive.perm exposes a cores argument: "jive.perm(data, nperms, alpha, est, conv, maxiter, orthIndiv, showProgress, cores)"
- [other] RcppEigen speedups increased with core count from 67.5% to 92.9% runtime reduction: [evidence span withheld — anchor status: not_found]
- [abstract] Original JIVE implementation infeasible for large-scale single-cell data: "its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets"
