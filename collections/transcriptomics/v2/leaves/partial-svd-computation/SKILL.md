---
name: partial-svd-computation
description: Use when a decomposition algorithm (e.g. JIVE) needs only the 1–10 singular
  components of a large dense matrix and full SVD is the runtime bottleneck.
when_to_use_negative:
- All singular values/vectors or the full rank decomposition are required, not just
  the leading 1–10 components.
- The matrix is small enough that base R svd runtime (~1.67 s at 1000×1000) is not
  a bottleneck.
- The data are sparse or too large for dense storage, where specialized sparse iterative
  solvers would be more appropriate.
edam_operation: http://edamontology.org/operation_0409
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: RSpectra
  role: Provides svds for computing a specified number of leading singular components,
    replacing base R's full svd as the efficient kernel
- name: R
  role: Runtime environment in which the benchmark timing loops and svd/svds calls
    are executed
- name: svd
  role: Base R full SVD used as the baseline timing reference for the runtime comparison
- name: JIVE (scJIVE)
  role: Downstream decomposition algorithm whose computational efficiency is boosted
    by substituting partial SVD for full SVD
  repo: https://github.com/oconnell-statistics-lab/scJIVE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:642e235eff70b75fb5f7cc88845183b3e80c360710fc5c8a22c01aebffefe359
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oconnell-statistics-lab/scJIVE
  tools:
  - JIVE (scJIVE)
  - R
  - RSpectra
  - svd
  edam_operation: http://edamontology.org/operation_0409
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# partial-svd-computation

## Summary

Benchmarks and applies RSpectra's partial SVD (svds) against base R's full svd to compute leading singular components, the core linear-algebra kernel behind JIVE's computational-efficiency boost. On 1000×1000 matrices it cuts runtime by 91–95% for 1–10 components.

## When to use

Use when a decomposition algorithm (e.g. JIVE) needs only the leading 1–10 singular components of a large dense matrix and full SVD is the runtime bottleneck.

## When NOT to use

- All singular values/vectors or the full rank decomposition are required, not just the leading 1–10 components.
- The matrix is small enough that base R svd runtime (~1.67 s at 1000×1000) is not a bottleneck.
- The data are sparse or too large for dense storage, where specialized sparse iterative solvers would be more appropriate.

## Inputs

- dense numeric matrix (benchmark: 1000x1000 iid standard-normal entries in R, one per repetition over 100 reps)
- number of leading singular components to compute (1, 5, or 10 in the benchmark)

## Outputs

- leading singular values/vectors for the requested component count from svds
- runtime measurements per method (base svd vs svds) and per component count
- aggregated comparison table (method, number of components, mean runtime, percent reduction) written to svd_benchmark.csv

## How to apply

Generate or load the square/dense matrix (the benchmark used 1000×1000 iid standard-normal matrices, one per repetition across 100 repetitions). Time base R svd computing 1, 5, and 10 singular components, then time RSpectra::svds for the same component counts on the same matrices. Aggregate mean runtimes per method per component count (~1.67 s for full svd vs 0.08/0.12/0.15 s for partial SVD at 1/5/10 components) and compute percent reductions (95.4%, 93.1%, 91.0%). Rationale: JIVE's original implementation was computationally infeasible for large-scale single-cell data, and computing only the leading components suffices for the joint/individual low-rank structure it estimates. Write the comparison as a table (method, number of components, mean runtime, percent reduction) e.g. to svd_benchmark.csv. Treat runtime scaling with the requested number of components (0.08 → 0.15 s from 1 → 10) as the expected cost profile.

## Related tools

- **RSpectra** (Provides svds for computing a specified number of leading singular components, replacing base R's full svd as the efficient kernel)
- **R** (Runtime environment in which the benchmark timing loops and svd/svds calls are executed)
- **svd** (Base R full SVD used as the baseline timing reference for the runtime comparison)
- **JIVE (scJIVE)** (Downstream decomposition algorithm whose computational efficiency is boosted by substituting partial SVD for full SVD) — https://github.com/oconnell-statistics-lab/scJIVE

## Examples

```
library(RSpectra); s <- svds(X, k=10); Rscript svd_benchmark.R  # times base svd vs RSpectra::svds on 100x 1000x1000 matrices, writes svd_benchmark.csv
```

## Evaluation signals

- Percent runtime reduction falls in the ~91–95% band reported for 1/5/10 components (95.4%, 93.1%, 91.0%).
- Partial-SVD mean runtime is roughly proportional to component count: ~0.08 s at k=1, ~0.12 s at k=5, ~0.15 s at k=10.
- Leading singular values and singular vectors from svds match those from base svd to numerical tolerance.
- svd_benchmark.csv contains exactly 3 rows (one per component count) with columns method, number of components, mean runtime, percent reduction.
- Same 100 generated 1000×1000 matrices are reused for both methods so the comparison is paired.

## Limitations

- Benchmark figures were measured on 1000×1000 standard-normal matrices; other sizes, conditioning, or sparsity patterns may shift the 91–95% reduction.
- Only 1, 5, and 10 leading components were evaluated; runtime advantages may shrink as the requested rank grows.
- JIVE's original implementation remains computationally infeasible for large-scale single-cell data without this efficiency boost, so full-SVD fallback is not viable at scRNA-seq scale.

## Evidence

- [methods] base R svd averaged ~1.67 s while partial SVD took 0.08, 0.12, and 0.15 s for 1, 5, and 10 components: [evidence span withheld — anchor status: not_found]
- [abstract] JIVE computational infeasibility: "However, its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets."
- [intro] efficiency motivation: "We enhance JIVE for large-scale scRNA-seq data by boosting its computational efficiency and tailoring it to the single-cell context."
- [methods] benchmark setup: [evidence span withheld — anchor status: not_found]
