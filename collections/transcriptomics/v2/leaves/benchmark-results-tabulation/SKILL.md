---
name: benchmark-results-tabulation
description: Use when replacing a full SVD routine (e.g. base R svd) with a partial/truncated
  SVD (e.g. RSpectra::svds) inside an iterative algorithm like JIVE, and you need
  runtime evidence that the substitution preserves the computation while cutting cost.
when_to_use_negative:
- The matrices are small enough that full SVD runtime is not a bottleneck (partial
  SVD overhead cannot pay off at this scale)
- Accuracy of the truncated decomposition rather than runtime is the open question
  — this skill measures speed and does not validate that partial SVD recovers the
  same joint/individual ranks in the JIVE decomposition
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Environment in which the benchmark matrices are generated, svd/svds are timed,
    and the comparison table is written
- name: svd
  role: Baseline full SVD routine whose runtime on 1, 5, and 10 components is measured
    as the reference (~1.67 s mean on 1000x1000)
- name: RSpectra
  role: Provides svds (partial SVD) whose runtimes (0.08/0.12/0.15 s at 1/5/10 components)
    are benchmarked against base svd
- name: JIVE (scJIVE)
  role: Algorithm whose computational efficiency motivates the SVD-substitution benchmark;
    JIVE's original bulk-data implementation was computationally infeasible for large-scale
    single-cell data
  repo: https://github.com/oconnell-statistics-lab/scJIVE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:0bc6ffbb5aa7095db00214094fc87df0fd06aa96b2ec689fb69a2247ff1841e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oconnell-statistics-lab/scJIVE
  tools:
  - JIVE (scJIVE)
  - R
  - RSpectra
  - svd
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# benchmark-results-tabulation

## Summary

Benchmarking and tabulation skill for comparing runtime of the base R svd against RSpectra's partial SVD (svds) when computing leading singular components in JIVE, aggregating mean runtimes and percent reductions into a small CSV comparison table.

## When to use

Use when replacing a full SVD routine (e.g. base R svd) with a partial/truncated SVD (e.g. RSpectra::svds) inside an iterative algorithm like JIVE, and you need runtime evidence that the substitution preserves the computation while cutting cost.

## When NOT to use

- The matrices are small enough that full SVD runtime is not a bottleneck (partial SVD overhead cannot pay off at this scale)
- Accuracy of the truncated decomposition rather than runtime is the open question — this skill measures speed and does not validate that partial SVD recovers the same joint/individual ranks in the JIVE decomposition

## Inputs

- 1000x1000 matrices with iid standard-normal entries (one per repetition, 100 repetitions)
- Component counts to evaluate (1, 5, and 10 singular components)
- Baseline SVD implementation (base R svd) and candidate implementation (RSpectra::svds)

## Outputs

- svd_benchmark.csv — a 3-row table with columns: method, number of components, mean runtime, percent reduction
- Per-repetition runtime records for base svd and partial SVD at 1, 5, and 10 components

## How to apply

Generate the workload matrices matching the target algorithm's inputs — in the JIVE benchmark, 1000x1000 matrices with iid standard-normal entries, one per repetition (100 repetitions). Time the baseline (base R svd) computing 1, 5, and 10 singular components on each matrix, then time RSpectra::svds for the same component counts on the same matrices, recording runtimes for both. Aggregate mean runtimes per method per component count (baseline ~1.67 s vs 0.08/0.12/0.15 s for partial SVD at 1/5/10 components) and compute percent reductions (95.4%, 93.1%, 91.0%). Write the aggregated comparison as a small table (here a 3-row svd_benchmark.csv) with columns for method, number of components, mean runtime, and percent reduction, so the substitution decision can be documented in the paper's computational-efficiency argument.

## Related tools

- **R** (Environment in which the benchmark matrices are generated, svd/svds are timed, and the comparison table is written)
- **svd** (Baseline full SVD routine whose runtime on 1, 5, and 10 components is measured as the reference (~1.67 s mean on 1000x1000))
- **RSpectra** (Provides svds (partial SVD) whose runtimes (0.08/0.12/0.15 s at 1/5/10 components) are benchmarked against base svd)
- **JIVE (scJIVE)** (Algorithm whose computational efficiency motivates the SVD-substitution benchmark; JIVE's original bulk-data implementation was computationally infeasible for large-scale single-cell data) — https://github.com/oconnell-statistics-lab/scJIVE

## Examples

```
Rscript -e 'X <- matrix(rnorm(1e6),1000); t1 <- system.time(svd(X)); t2 <- system.time(RSpectra::svds(X, k=5)); write.csv(data.frame(method,components,mean_runtime,pct_reduction), "svd_benchmark.csv")'
```

## Evaluation signals

- The output CSV has exactly the expected rows — one per (method, component count) pair — with columns method, number of components, mean runtime, and percent reduction
- Mean baseline runtime lands near the reference value (~1.67 s for base svd on 1000x1000 across 100 repetitions)
- Percent reductions fall in the observed ranges: ~95.4% (1 component), ~93.1% (5 components), ~91.0% (10 components)
- Partial SVD runtimes scale monotonically with the number of components (0.08 -> 0.12 -> 0.15 s), as expected for truncated decomposition
- The same matrices are used for both methods in each repetition, so runtimes are paired and comparable

## Limitations

- Results are specific to the benchmark configuration (1000x1000 standard-normal matrices, 100 repetitions); percent reductions may differ for the real multi-matrix JIVE input structure
- The benchmark measures runtime only — it does not establish that the partial SVD yields identical joint and individual rank estimates (e.g. the joint rank 2 / individual ranks 27, 26, 25 reported for the BRCA data)
- Runtime reductions shrink as more components are requested (95.4% at 1 component down to 91.0% at 10), so gains may be smaller at higher ranks

## Evidence

- [full_text] On 1000×1000 standard-normal matrices, base R svd averaged ~1.67 s while partial SVD took 0.08, 0.12, and 0.15 s for 1, 5, and 10 components, i.e. runtime reductions of 95.4%, 93.1%, and 91.0%.: [evidence span withheld — anchor status: not_found]
- [abstract] However, its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets.: "its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets"
- [readme] We enhance JIVE for large-scale scRNA-seq data by boosting its computational efficiency and tailoring it to the single-cell context.: "We enhance JIVE for large-scale scRNA-seq data by boosting its computational efficiency and tailoring it to the single-cell context"
- [methods] The default function to estimate the JIVE decomposition is `jive`: "The default function to estimate the JIVE decomposition is `jive`"
