---
name: runtime-scaling-analysis
description: Use when comparing computational scalability of imputation or similar
  single-cell methods on datasets of hundreds of thousands of cells, where runtime-not
  accuracy-is the question and single-core timings must be isolated.
when_to_use_negative:
- The question is imputation accuracy rather than computational cost — use a downstream
  cell-type/marker-recovery evaluation instead.
- The dataset is small (thousands of cells), where runtime differences between methods
  are negligible and the scaling question does not arise.
- Methods cannot be pinned to a single core or have incompatible runtimes omitted;
  mixing single-core and multi-core timings would invalidate the comparison.
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: ALRA
  role: 'Fastest benchmarked method: randomized SVD rank-k approximation (k=48 on
    full Hrvatin data, rsvd package, q=10 power iterations) run per subset with wall-clock
    timing; ~40 min on 50000 cells'
  repo: https://github.com/KlugerLab/ALRA
- name: MAGIC
  role: Baseline imputation method timed on each subset; among the fastest, ~40 min
    on 50000 cells
- name: DCA
  role: Baseline imputation method timed on each subset; about 4 h on 50000 cells
- name: SAVER
  role: Baseline imputation method timed with the do.fast option set to true; exceeds
    10 h at 10000 cells and was not attempted on larger matrices
- name: scImpute
  role: Baseline imputation method timed on each subset; exceeds 10 h at 10000 cells
    and was not attempted on larger matrices
- name: rsvd
  role: R package providing randomized SVD used by ALRA in the benchmarked configuration
- name: rpca-mkl
  role: Optional MKL-accelerated randomized SVD backend selectable via ALRA's use.mkl
    flag for dramatic speedups over the default rpca-based version
  repo: https://github.com/KlugerLab/rpca-mkl
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:49a5d1477913aa113c9d2405b0997e75a6b5cc1139d436739699aee1f3bf606a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - DCA
  - MAGIC
  - SAVER
  - rpca-mkl
  - rsvd
  - scImpute
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# runtime-scaling-analysis

## Summary

Benchmark the wall-clock runtime of scRNA-seq imputation methods (ALRA, MAGIC, DCA, scImpute, SAVER) on nested cell-count subsets of a large dataset, each restricted to a single core, to quantify how runtime scales with the number of cells.

## When to use

Use when comparing computational scalability of imputation or similar single-cell methods on datasets of hundreds of thousands of cells, where runtime-not accuracy-is the question and single-core timings must be isolated.

## When NOT to use

- The question is imputation accuracy rather than computational cost — use a downstream cell-type/marker-recovery evaluation instead.
- The dataset is small (thousands of cells), where runtime differences between methods are negligible and the scaling question does not arise.
- Methods cannot be pinned to a single core or have incompatible runtimes omitted; mixing single-core and multi-core timings would invalidate the comparison.

## Inputs

- Filtered scRNA-seq expression matrix (genes x cells), e.g. Hrvatin et al. 2018 mouse visual cortex with 19155 genes and 65539 cells
- List of imputation methods to benchmark (ALRA, MAGIC, DCA, scImpute, SAVER)
- Subset sizes to sample (e.g., 5000, 10000, 20000, 30000, 40000, 50000 cells)

## Outputs

- runtime_table.csv containing wall-clock runtime for each method at each subset size
- Runtime-vs-cell-count scaling comparison across the five imputation methods

## How to apply

Load a large filtered expression matrix (e.g., Hrvatin et al. 2018 mouse visual cortex: 19155 genes x 65539 cells, after dropping genes with mean expression < 0.00003 or non-zero expression in fewer than 4 cells). Sample nested random subsets at fixed cell counts (5000, 10000, 20000, 30000, 40000, 50000 cells) so all methods see identical inputs. Run each imputation method — ALRA (rank-k=48 for full data, randomized SVD via the rsvd R package, q=10 power iterations), MAGIC, DCA, scImpute, and SAVER with do.fast=TRUE — pinned to a single core, recording wall-clock time per run. Assemble times into a runtime-vs-cell-count table (runtime_table.csv) and validate against the published scaling: ALRA and MAGIC finish 50000 cells in roughly 40 minutes, DCA takes about 4 hours on the same subset, while scImpute and SAVER exceed 10 hours at only 10000 cells. Single-core execution is the key design choice: it removes multi-threading as a confounder when comparing implementations.

## Related tools

- **ALRA** (Fastest benchmarked method: randomized SVD rank-k approximation (k=48 on full Hrvatin data, rsvd package, q=10 power iterations) run per subset with wall-clock timing; ~40 min on 50000 cells) — https://github.com/KlugerLab/ALRA
- **MAGIC** (Baseline imputation method timed on each subset; among the fastest, ~40 min on 50000 cells)
- **DCA** (Baseline imputation method timed on each subset; about 4 h on 50000 cells)
- **SAVER** (Baseline imputation method timed with the do.fast option set to true; exceeds 10 h at 10000 cells and was not attempted on larger matrices)
- **scImpute** (Baseline imputation method timed on each subset; exceeds 10 h at 10000 cells and was not attempted on larger matrices)
- **rsvd** (R package providing randomized SVD used by ALRA in the benchmarked configuration)
- **rpca-mkl** (Optional MKL-accelerated randomized SVD backend selectable via ALRA's use.mkl flag for dramatic speedups over the default rpca-based version) — https://github.com/KlugerLab/rpca-mkl

## Examples

```
system.time(result.completed <- alra(A_norm, k = 48, q = 10))  # single-core timing on a 50000-cell Hrvatin subset
```

## Evaluation signals

- runtime_table.csv contains timings for all five methods (ALRA, MAGIC, DCA, scImpute, SAVER) across all six subset sizes (5000–50000 cells)
- ALRA and MAGIC runtimes at 50000 cells are approximately 40 minutes
- DCA runtime at 50000 cells is approximately 4 hours
- scImpute and SAVER exceed 10 hours at 10000 cells and are absent for larger subset sizes
- All runs were executed on a single core, so recorded times are comparable across methods

## Limitations

- SAVER and scImpute are extremely slow on datasets over 100000 cells — they exceed 10 h on just 10000 cells and were not attempted on larger matrices, so their scaling beyond that size is unmeasured.
- Runtimes are hardware- and single-core-dependent; absolute times do not transfer across machines even though relative scaling does.
- Optionally, rpca-mkl (MKL-accelerated randomized SVD) is still under development and not on CRAN, and its pre-compiled MKL libraries are only included for Linux and OS X.

## Evidence

- [results] ALRA and MAGIC are the fastest imputation methods: [evidence span withheld — anchor status: not_found]
- [intro] scImpute and SAVER slow on large datasets: "SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells"
- [intro] First step of ALRA is SVD: "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
- [readme] rsvd prerequisite and MKL flag: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
- [readme] MKL speedup flag: "The functions now have a flag `use.mkl` for users who have installed [rpca-mkl](https://github.com/KlugerLab/rpca-mkl), which allows for dramatic speedups"
