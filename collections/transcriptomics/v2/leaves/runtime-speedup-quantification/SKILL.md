---
name: runtime-speedup-quantification
description: Use when you must quantify how much faster an optimized C++/OpenMP backend
  runs versus R's single-threaded baseline (e.g., %*%), across varying core counts,
  with reproducible timing over many repetitions.
when_to_use_negative:
- The question is correctness or biological quality of results (e.g., whether a method
  preserves cell-type effects), not computational runtime
- You have time for only a single untimed run — one-shot timings without repetitions
  (here 100 per configuration) are too noisy to support speedup claims
- The operation has no meaningful single-threaded baseline to compare against, so
  percentage speedup is undefined
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: R
  role: Host language; generates seeded test matrices and provides the single-threaded
    %*% baseline
- name: Rcpp
  role: Compiles and binds the C++ matrix-multiplication backends into R
- name: RcppEigen
  role: C++ Eigen matrix-multiplication backend timed under OpenMP at 1, 2, 4, and
    8 cores
- name: RcppArmadillo
  role: C++ Armadillo matrix-multiplication backend timed alongside RcppEigen for
    comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:11cb5a0dbfaa9541184390d8e02010843b65840f77f9372a4cfbee7d78df9444
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
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# runtime-speedup-quantification

## Summary

Quantifies runtime speedup of an optimized implementation (e.g., RcppEigen/RcppArmadillo C++ backends with OpenMP multi-core parallelism) relative to a single-threaded R baseline, reporting mean runtimes and percentage runtime reduction per core count. This is the benchmarking pattern behind claims that JIVE's computational efficiency was boosted for large-scale scRNA-seq data.

## When to use

Use when you must quantify how much faster an optimized C++/OpenMP backend runs versus R's single-threaded baseline (e.g., %*%), across varying core counts, with reproducible timing over many repetitions.

## When NOT to use

- The question is correctness or biological quality of results (e.g., whether a method preserves cell-type effects), not computational runtime
- You have time for only a single untimed run — one-shot timings without repetitions (here 100 per configuration) are too noisy to support speedup claims
- The operation has no meaningful single-threaded baseline to compare against, so percentage speedup is undefined

## Inputs

- Two dense numeric matrices (e.g., 1000x1000 standard-normal, generated with a fixed random seed in R)
- R baseline implementation of the operation (built-in %*% operator)
- C++ backend implementations compiled via Rcpp (RcppArmadillo and RcppEigen matrix multiplication with OpenMP)

## Outputs

- runtime_benchmark.csv with one column/field per implementation and core configuration (mean runtime and percentage speedup relative to baseline)
- Percentage speedup values per core count (e.g., 67.5% at 1 core up to 92.9% at 8 cores)

## How to apply

Generate the workload (e.g., 1000x1000 standard-normal matrices) with a fixed random seed so timings are reproducible, then time the single-threaded baseline — R's built-in %*% operator — over 100 repetitions as the reference point. Compile the C++ backends (RcppArmadillo and RcppEigen matrix multiplication via Rcpp) and time each with OpenMP parallelism set to 1, 2, 4, and 8 cores, again 100 repetitions per configuration. Compute the mean runtime per implementation per core count and the percentage speedup relative to the baseline; in the surfaced benchmark this ranged from 67.5% shorter runtime at one core to 92.9% at eight cores. Write mean runtimes and speedup percentages to runtime_benchmark.csv with one column/field per implementation and core configuration. The rationale is that efficiency claims (such as scJIVE's boosted computational efficiency over the original JIVE, which was designed for bulk data and is infeasible for large-scale single-cell datasets) must be backed by controlled, repeated timings against a baseline rather than anecdotal single runs.

## Related tools

- **R** (Host language; generates seeded test matrices and provides the single-threaded %*% baseline)
- **Rcpp** (Compiles and binds the C++ matrix-multiplication backends into R)
- **RcppEigen** (C++ Eigen matrix-multiplication backend timed under OpenMP at 1, 2, 4, and 8 cores)
- **RcppArmadillo** (C++ Armadillo matrix-multiplication backend timed alongside RcppEigen for comparison)

## Examples

```
times <- replicate(100, system.time(X %*% Y)[["elapsed"]]); write.csv(data.frame(mean = mean(times)), "runtime_benchmark.csv")
```

## Evaluation signals

- runtime_benchmark.csv exists and contains a field for each implementation and core configuration
- The reported 8-core speedup lies within the 90–95% range (paper reports 92.9%)
- Speedups increase monotonically with core count (67.5% at 1 core up to 92.9% at 8 cores)
- Each configuration was timed over the full 100 repetitions and means, not single runs, are reported
- Re-running with the same fixed random seed reproduces the same input matrices and comparable timings

## Limitations

- Benchmark uses synthetic 1000x1000 standard-normal matrices; timings may not transfer to large-scale single-cell data workloads
- Original JIVE was designed for bulk sequencing data and is computationally infeasible for large-scale scRNA-seq datasets, so speedups vs. it cannot be measured at single-cell scale without the enhanced implementation
- OpenMP core counts depend on available hardware; oversubscription or missing OpenMP support invalidates the 1/2/4/8-core comparison

## Evidence

- [intro] We enhance JIVE for large-scale scRNA-seq data by boosting its computational efficiency and tailoring it to the single-cell context.: "We enhance JIVE for large-scale scRNA-seq data by boosting its computational efficiency and tailoring it to the single-cell context."
- [methods] bic.jive(data, n, d, conv, maxiter, orthIndiv, showProgress, cores): "bic.jive(data, n, d, conv, maxiter, orthIndiv, showProgress, cores)"
- [abstract] its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets.: "its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets."
