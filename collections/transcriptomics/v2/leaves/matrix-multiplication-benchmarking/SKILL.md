---
name: matrix-multiplication-benchmarking
description: Use when comparing the runtime of dense matrix multiplication across
  R backends (%*%, RcppArmadillo, RcppEigen) with varying OpenMP core counts, or when
  justifying a faster backend for a computationally heavy R linear-algebra routine.
when_to_use_negative:
- The analysis goal is batch-effect correction or decomposition of scRNA-seq data
  (e.g., running jive/bic.jive with cores set for the JIVE estimator) rather than
  timing a linear-algebra kernel itself.
- The matrices are not dense matrix multiplication workloads (e.g., I/O-bound tasks
  or sparse operations), where the %*%-vs-C++ speedup metric is not meaningful.
- OpenMP is unavailable, so the 1/2/4/8-core parallel configurations cannot be compiled
  or evaluated.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Host environment for generating seeded 1000x1000 test matrices, running the
    %*% baseline, and aggregating mean runtimes into runtime_benchmark.csv
- name: Rcpp
  role: Compilation bridge used to build the RcppArmadillo and RcppEigen matrix-multiplication
    backends callable from R
- name: RcppEigen
  role: C++ matrix-multiplication backend benchmarked with OpenMP at 1, 2, 4, and
    8 cores; produced the 67.5%-92.9% speedups
- name: RcppArmadillo
  role: Alternative C++ matrix-multiplication backend benchmarked under the same OpenMP
    core configurations for comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:78bc1d1ceb07f6408f190c7e0d8ba2f278887f6b9ff6eae7b888c03392a0b358
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Rcpp
  - RcppArmadillo
  - RcppEigen
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# matrix-multiplication-benchmarking

## Summary

A reproducible R benchmarking procedure that times R's built-in %*% operator against RcppArmadillo and RcppEigen matrix-multiplication backends compiled via Rcpp with OpenMP at 1, 2, 4, and 8 cores, quantifying percentage speedup over the single-threaded baseline. This is the kind of computational-efficiency validation used to justify fast C++ backends for heavy linear algebra such as the JIVE decomposition.

## When to use

Use when comparing the runtime of dense matrix multiplication across R backends (%*%, RcppArmadillo, RcppEigen) with varying OpenMP core counts, or when justifying a faster backend for a computationally heavy R linear-algebra routine.

## When NOT to use

- The analysis goal is batch-effect correction or decomposition of scRNA-seq data (e.g., running jive/bic.jive with cores set for the JIVE estimator) rather than timing a linear-algebra kernel itself.
- The matrices are not dense matrix multiplication workloads (e.g., I/O-bound tasks or sparse operations), where the %*%-vs-C++ speedup metric is not meaningful.
- OpenMP is unavailable, so the 1/2/4/8-core parallel configurations cannot be compiled or evaluated.

## Inputs

- 1000x1000 standard-normal dense matrices generated in R with a fixed random seed
- C++ matrix-multiplication backends (RcppArmadillo, RcppEigen) compiled via Rcpp with OpenMP enabled

## Outputs

- runtime_benchmark.csv with one column/field per implementation and core configuration, holding mean runtimes and percentage speedups relative to the %*% baseline

## How to apply

Generate 1000x1000 standard-normal matrices in R with a fixed random seed so the benchmark is reproducible, and time R's built-in %*% operator as the single-threaded baseline over 100 repetitions. Then time the RcppArmadillo and RcppEigen C++ matrix-multiplication backends (compiled via Rcpp) with OpenMP parallelism set to 1, 2, 4, and 8 cores, 100 repetitions per configuration. Compute the mean runtime per implementation per core count and the percentage speedup relative to the %*% baseline. Write the mean runtimes and speedup percentages to runtime_benchmark.csv with one column/field per implementation and core configuration. The reference result is a monotone speedup with core count, from 67.5% shorter runtime on one core to 92.9% shorter runtime on eight cores.

## Related tools

- **R** (Host environment for generating seeded 1000x1000 test matrices, running the %*% baseline, and aggregating mean runtimes into runtime_benchmark.csv)
- **Rcpp** (Compilation bridge used to build the RcppArmadillo and RcppEigen matrix-multiplication backends callable from R)
- **RcppEigen** (C++ matrix-multiplication backend benchmarked with OpenMP at 1, 2, 4, and 8 cores; produced the 67.5%-92.9% speedups)
- **RcppArmadillo** (Alternative C++ matrix-multiplication backend benchmarked under the same OpenMP core configurations for comparison)

## Examples

```
Rscript -e 'set.seed(42); A <- matrix(rnorm(1e6),1000); system.time(replicate(100, A %*% A))'
```

## Evaluation signals

- runtime_benchmark.csv exists and contains a field for each implementation (R %*%, RcppArmadillo, RcppEigen) and core configuration (1, 2, 4, 8 cores).
- The reported 8-core RcppEigen speedup lies within the 90-95% range (paper reports 92.9%).
- Speedup is monotonically non-decreasing with core count, from 67.5% shorter runtime on one core up to 92.9% on eight cores.
- Re-running the benchmark with the same fixed random seed reproduces comparable mean runtimes over 100 repetitions per configuration.

## Limitations

- Benchmark results are specific to dense 1000x1000 standard-normal matrices; absolute speedups will differ for other matrix sizes, sparsity, or BLAS settings underlying %*%.
- Achieved speedups depend on the number of available CPU cores and OpenMP configuration of the compiled backends.
- Timing over only 100 repetitions per configuration may leave residual measurement noise in the mean runtimes.

## Evidence

- [other] Speedups increase with core count: [evidence span withheld — anchor status: not_found]
- [other] Benchmark protocol: [evidence span withheld — anchor status: not_found]
- [readme] Motivation for fast backends: "We enhance JIVE for large-scale scRNA-seq data by boosting its computational efficiency and tailoring it to the single-cell context."
- [abstract] Computational infeasibility of original JIVE: "its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets."
