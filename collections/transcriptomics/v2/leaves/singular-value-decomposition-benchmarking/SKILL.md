---
name: singular-value-decomposition-benchmarking
description: Use when a pipeline (e.g., JIVE) repeatedly computes only the 1–10 singular
  components on large matrices and you must decide whether a partial SVD replaces
  full svd without changing results.
when_to_use_negative:
- The analysis requires the full singular spectrum (all components), not just the
  leading 1–10 — partial SVD via svds is not a drop-in replacement for a full decomposition.
- The benchmark matrix dimensions or data distribution differ substantially from the
  target pipeline's inputs (e.g., tall-skinny or sparse single-cell matrices), so
  runtimes will not transfer.
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Language in which the svd/svds timing benchmark is implemented and runtimes
    aggregated
- name: RSpectra
  role: Provides svds for computing only the leading 1–10 singular components, the
    candidate fast replacement
- name: r.jive
  role: JIVE implementation whose svd calls are the motivation for the runtime benchmark
  repo: https://github.com/oconnell-statistics-lab/scJIVE
- name: scJIVE
  role: Enhanced JIVE whose computational-efficiency gains over original JIVE this
    benchmarking rationale supports
  repo: https://github.com/oconnell-statistics-lab/scJIVE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:1d528d25d3d05b6e6dafbb353d24528b66d320feece3e27112281181eb8919ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oconnell-statistics-lab/scJIVE
  tools:
  - R
  - RSpectra
  - r.jive
  - scJIVE
  - svd
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# singular-value-decomposition-benchmarking

## Summary

Benchmarks base R svd against RSpectra's partial SVD (svds) for computing leading singular components, quantifying runtime reductions to justify accelerating JIVE-style matrix decompositions on large datasets.

## When to use

Use when a pipeline (e.g., JIVE) repeatedly computes only the leading 1–10 singular components on large matrices and you must decide whether a partial SVD replaces full svd without changing results.

## When NOT to use

- The analysis requires the full singular spectrum (all components), not just the leading 1–10 — partial SVD via svds is not a drop-in replacement for a full decomposition.
- The benchmark matrix dimensions or data distribution differ substantially from the target pipeline's inputs (e.g., tall-skinny or sparse single-cell matrices), so runtimes will not transfer.

## Inputs

- 100 synthetic 1000x1000 matrices with iid standard-normal entries (one per repetition)
- Component counts to benchmark (e.g., 1, 5, and 10 singular components)

## Outputs

- Per-repetition runtime measurements for base svd and RSpectra svds at each component count
- svd_benchmark.csv: 3-row comparison table with columns for method, number of components, mean runtime, and percent reduction

## How to apply

Generate replicate test matrices matching the target analysis dimensions (here 1000×1000 with iid standard-normal entries, 100 repetitions). Time base R svd computing 1, 5, and 10 singular components on each matrix, then time RSpectra::svds computing the same component counts on the same matrices. Aggregate mean runtimes per method per component count and compute percent reductions (observed: base svd ~1.67 s vs 0.08/0.12/0.15 s for 1/5/10 components, i.e., 95.4%/93.1%/91.0% reductions). Write a comparison table (method, number of components, mean runtime, percent reduction) to svd_benchmark.csv. Rationale: JIVE's original implementation is computationally infeasible for large-scale single-cell datasets, so swapping full svd for a partial SVD inside the jive workflow (and dependencies like SVDmiss) is a key efficiency boost.

## Related tools

- **R** (Language in which the svd/svds timing benchmark is implemented and runtimes aggregated)
- **RSpectra** (Provides svds for computing only the leading 1–10 singular components, the candidate fast replacement)
- **r.jive** (JIVE implementation whose svd calls are the motivation for the runtime benchmark) — https://github.com/oconnell-statistics-lab/scJIVE
- **scJIVE** (Enhanced JIVE whose computational-efficiency gains over original JIVE this benchmarking rationale supports) — https://github.com/oconnell-statistics-lab/scJIVE

## Examples

```
library(RSpectra); system.time(svd(X)); system.time(svds(X, k=10))
```

## Evaluation signals

- Percent runtime reduction falls in the expected ~90–96% range for 1–10 components on 1000×1000 dense matrices (95.4%, 93.1%, 91.0% observed).
- Mean base svd runtime is on the order of seconds (~1.67 s) while partial SVD stays sub-second (0.08–0.15 s).
- svd_benchmark.csv contains exactly one row per (method, component count) combination with non-negative mean runtime and reduction values.
- Both methods are timed on identical input matrices and identical component counts so the comparison is paired, not confounded.

## Limitations

- Benchmark uses synthetic 1000×1000 standard-normal matrices; real scRNA-seq inputs are sparse, higher-dimensional, and may shift the relative advantage of svds.
- Partial SVD only computes requested leading components; accuracy of the truncated decomposition versus full svd is not itself quantified by the runtime benchmark.
- Original JIVE's implementation 'is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets' — the speedup addresses feasibility but per-component accuracy should be checked in the target pipeline.

## Evidence

- [abstract] JIVE's original implementation was designed for bulk sequencing data and is computationally infeasible for large-scale single-cell sequencing datasets.: "However, its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets."
- [intro] We enhance JIVE for large-scale scRNA-seq data by boosting its computational efficiency and tailoring it to the single-cell context.: "We enhance JIVE for large-scale scRNA-seq data by boosting its computational efficiency and tailoring it to the single-cell context."
- [methods] SVDmiss is a workflow dependency in the r.jive methods for handling missing values via SVD computations.: "SVDmiss(X, niter, ncomp, conv.reldiff)"
- [readme] The scJIVE implementation is distributed from the oconnell-statistics-lab GitHub repository.: [evidence span withheld — anchor status: not_found]
