---
name: runtime-performance-profiling
description: Use when a pipeline step (e.g., SVD inside JIVE) is suspected to be a
  computational bottleneck on large data and you must quantify how much a partial/approximate
  solver reduces runtime before adopting it.
when_to_use_negative:
- The downstream method requires the full singular spectrum or exact decomposition
  — a partial SVD (svds) only approximates leading components and would change results,
  not just speed.
- The input matrices are so small that baseline runtime is negligible (benchmarking
  overhead dominates and no adoption decision hinges on the measurement).
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: R
  role: environment for generating test matrices and timing both solvers
- name: RSpectra
  role: partial SVD (svds) candidate solver timed against the baseline for 1, 5, and
    10 components
- name: svd
  role: base R full SVD used as the runtime baseline (~1.67 s mean on 1000x1000)
- name: r.jive
  role: JIVE implementation whose SVD-based core motivates the runtime profiling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:57f4ff06db406f2d2df2419249a4d4b2e9105e577acc3e7648c4d58fb5dc28b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RSpectra
  - r.jive
  - svd
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# runtime-performance-profiling

## Summary

A benchmarking skill that times a baseline implementation against a faster partial approximation (e.g., base R svd vs RSpectra svds) across increasing component counts and quantifies percent runtime reduction, as done to justify speeding up JIVE for large-scale single-cell data.

## When to use

Use when a pipeline step (e.g., SVD inside JIVE) is suspected to be a computational bottleneck on large data and you must quantify how much a partial/approximate solver reduces runtime before adopting it.

## When NOT to use

- The downstream method requires the full singular spectrum or exact decomposition — a partial SVD (svds) only approximates leading components and would change results, not just speed.
- The input matrices are so small that baseline runtime is negligible (benchmarking overhead dominates and no adoption decision hinges on the measurement).

## Inputs

- 100 replicate 1000x1000 matrices with iid standard-normal entries (or the real data dimensionality of interest)
- List of component counts to test (e.g., 1, 5, 10 singular components)
- Baseline solver (base R svd) and candidate solver (RSpectra::svds) callable on the same inputs

## Outputs

- svd_benchmark.csv: 3-row table with columns method, number of components, mean runtime, percent reduction
- Per-repetition raw timings for each method/component-count combination

## How to apply

Generate replicate test inputs of fixed size (e.g., 100 repetitions of 1000×1000 matrices with iid standard-normal entries) so timings are comparable across methods. Time the baseline full decomposition (base R svd) and the candidate partial solver (RSpectra::svds) on the identical inputs for the same component counts actually needed downstream — here 1, 5, and 10 singular components, since JIVE only needs leading joint/individual structure. Aggregate mean runtime per method per component count, compute percent reduction relative to baseline (~1.67 s baseline vs 0.08/0.12/0.15 s for partial SVD, i.e. 95.4%/93.1%/91.0% reductions), and write a tidy comparison table (svd_benchmark.csv) with columns method, number of components, mean runtime, and percent reduction. The rationale: a partial SVD is acceptable only if it delivers the leading components the method consumes (JIVE's joint/individual low-rank estimates) at a large fraction of the cost; measured savings of >90% justify replacing the base solver in the scJIVE implementation for large-scale scRNA-seq data.

## Related tools

- **R** (environment for generating test matrices and timing both solvers)
- **RSpectra** (partial SVD (svds) candidate solver timed against the baseline for 1, 5, and 10 components)
- **svd** (base R full SVD used as the runtime baseline (~1.67 s mean on 1000x1000))
- **r.jive** (JIVE implementation whose SVD-based core motivates the runtime profiling)

## Examples

```
Rscript -e 'X<-matrix(rnorm(1e6),1000);t1<-system.time(svd(X)[1]);t2<-system.time(RSpectra::svds(X,k=5))'
```

## Evaluation signals

- Percent reductions fall in a plausible band relative to component count (here 95.4% for 1, 93.1% for 5, 91.0% for 10 — reduction shrinks as more components are requested).
- Baseline mean runtime is stable across the 100 repetitions and consistent with published magnitude (~1.67 s for 1000x1000 full SVD).
- Partial-SVD runtime increases monotonically with requested component count (0.08 < 0.12 < 0.15 s).
- svd_benchmark.csv has exactly the agreed schema: method, number of components, mean runtime, percent reduction, one row per component count.
- Both solvers are timed on identical input matrices, not regenerated ones, so differences are attributable to the solver.

## Limitations

- Timings are from synthetic 1000×1000 standard-normal matrices; runtimes may differ on the sparse, high-dimensional matrices typical of scRNA-seq data.
- Runtime reduction declines as more components are requested (95.4% → 91.0% from 1 to 10 components), so gains must be re-measured for the rank actually used.
- Partial SVD accuracy was not assessed here — the benchmark covers runtime only; approximation error must be validated separately against JIVE's joint/individual estimates.

## Evidence

- [methods] base R svd vs RSpectra partial SVD benchmark: [evidence span withheld — anchor status: not_found]
- [abstract] motivation for speeding up JIVE: "its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets"
- [intro] objective to boost computational efficiency: "We enhance JIVE for large-scale scRNA-seq data by boosting its computational efficiency and tailoring it to the single-cell context."
- [readme] install scJIVE: "You can install the development version of scJIVE from GitHub with:

``` r
# install.packages("devtools")
devtools::install_github("oconnell-statistics-lab/scJIVE")"
