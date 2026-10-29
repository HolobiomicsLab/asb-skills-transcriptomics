---
name: random-matrix-simulation
description: Use when you need a reproducible, size-controlled workload to benchmark
  linear-algebra implementations (SVD variants, matrix multiplication, core-count
  scaling) before applying them to real data.
when_to_use_negative:
- The goal is to time methods on real empirical data (e.g., actual scRNA-seq count
  matrices or TCGA BRCA_data) rather than on controlled synthetic workloads
- The matrices are much smaller than 1000×1000, where iterative partial SVD overhead
  can dominate and the reported speedup regime (90%+) will not hold
- The analysis requires matrices with realistic structure (sparsity, zero inflation,
  batch effects) — iid standard-normal entries deliberately strip all such structure
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Generates the 1000×1000 standard-normal matrices (set.seed + rnorm/matrix)
    and hosts the built-in %*% and svd baselines
- name: RSpectra
  role: Provides svds, the partial SVD whose runtimes for 1, 5, and 10 components
    are benchmarked against base svd
- name: RcppArmadillo
  role: C++ matrix-multiplication backend timed against the %*% baseline at each OpenMP
    core count
- name: RcppEigen
  role: C++ matrix-multiplication backend (compiled via Rcpp) whose 1–8 core OpenMP
    runtimes define the speedup curve
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:ab3b30d676985e76dc6048535aa11c798167bce52ca43fb4ff8df058c5c6eb1e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RSpectra
  - Rcpp
  - RcppArmadillo
  - RcppEigen
  - svd
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# random-matrix-simulation

## Summary

Generate synthetic 1000×1000 standard-normal matrices in R (with a fixed seed) as controlled benchmark inputs for timing linear-algebra routines such as base svd, RSpectra svds, %*%, and OpenMP-parallel RcppEigen/RcppArmadillo backends.

## When to use

Use when you need a reproducible, size-controlled workload to benchmark linear-algebra implementations (SVD variants, matrix multiplication, core-count scaling) before applying them to real data.

## When NOT to use

- The goal is to time methods on real empirical data (e.g., actual scRNA-seq count matrices or TCGA BRCA_data) rather than on controlled synthetic workloads
- The matrices are much smaller than 1000×1000, where iterative partial SVD overhead can dominate and the reported speedup regime (90%+) will not hold
- The analysis requires matrices with realistic structure (sparsity, zero inflation, batch effects) — iid standard-normal entries deliberately strip all such structure

## Inputs

- Random seed (integer) for reproducibility
- Matrix dimensions (1000 × 1000) and number of repetitions (100)
- Number of singular components to compute (1, 5, 10)
- OpenMP core counts (1, 2, 4, 8)

## Outputs

- svd_benchmark.csv with columns method, number of components, mean runtime, percent reduction
- runtime_benchmark.csv with one field per implementation and core configuration, containing mean runtimes and speedup percentages

## How to apply

Simulate one iid standard-normal 1000×1000 matrix per repetition (100 repetitions) in R, setting a fixed seed so the benchmark is reproducible. For the SVD benchmark, time base R svd and RSpectra svds computing 1, 5, and 10 singular components on the same matrices, then aggregate mean runtimes per method per component count (~1.67 s base vs 0.08/0.12/0.15 s partial) and compute percent reductions (95.4%/93.1%/91.0%), writing a 3-row svd_benchmark.csv with method, components, mean runtime, and percent reduction. For the matrix-multiplication benchmark, time the built-in %*% operator as the single-threaded baseline over 100 repetitions, then time RcppArmadillo and RcppEigen (compiled via Rcpp) with OpenMP at 1, 2, 4, and 8 cores, computing mean runtimes and percent speedups (67.5% at one core up to 92.9% at eight) relative to the baseline into runtime_benchmark.csv. The rationale is that the paper's motivation for computational efficiency — base JIVE being computationally infeasible at single-cell scale — is quantified by such synthetic-matrix timings before integrating into scJIVE.

## Related tools

- **R** (Generates the 1000×1000 standard-normal matrices (set.seed + rnorm/matrix) and hosts the built-in %*% and svd baselines)
- **RSpectra** (Provides svds, the partial SVD whose runtimes for 1, 5, and 10 components are benchmarked against base svd)
- **RcppArmadillo** (C++ matrix-multiplication backend timed against the %*% baseline at each OpenMP core count)
- **RcppEigen** (C++ matrix-multiplication backend (compiled via Rcpp) whose 1–8 core OpenMP runtimes define the speedup curve)

## Examples

```
set.seed(123); X <- matrix(rnorm(1000 * 1000), nrow = 1000, ncol = 1000); system.time(svd(X, nu = 5, nv = 5))
```

## Evaluation signals

- With the fixed seed, rerunning the simulation reproduces the same set of 100 matrices and benchmark means
- svd_benchmark.csv has exactly 3 rows and mean runtimes consistent with ~1.67 s (base svd) and 0.08/0.12/0.15 s (partial SVD, 1/5/10 components), i.e. ~95.4%/93.1%/91.0% reductions
- runtime_benchmark.csv exists and contains a field for every implementation × core configuration
- The reported 8-core RcppEigen speedup falls within the 90–95% range (paper reports 92.9%)
- Speedups monotonically increase with core count, starting near 67.5% at one core

## Limitations

- Results are for iid standard-normal 1000×1000 matrices; real single-cell data are sparse and structured, so timings may not transfer to scJIVE's actual inputs
- Speedups depend strongly on core count and implementation (67.5% at 1 core vs 92.9% at 8 cores for RcppEigen), so single-configuration timings should not be generalized
- The paper notes JIVE's original implementation was designed for bulk sequencing data and is computationally infeasible for large-scale single-cell datasets; synthetic benchmarks quantify but do not resolve this gap

## Evidence

- [other] Generate 1000x1000 matrices with iid standard-normal entries in R, one per repetition (100 repetitions).: [evidence span withheld — anchor status: not_found]
- [other] fixed random seed: [evidence span withheld — anchor status: not_found]
- [abstract] computationally infeasible: "its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets."
- [readme] install scJIVE: "You can install the development version of scJIVE from GitHub with:

``` r
# install.packages("devtools")
devtools::install_github("oconnell-statistics-lab/scJIVE")"
