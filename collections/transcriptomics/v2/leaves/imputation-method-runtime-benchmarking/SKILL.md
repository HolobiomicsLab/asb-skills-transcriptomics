---
name: imputation-method-runtime-benchmarking
description: Use when you need to compare the computational scalability of scRNA-seq
  imputation methods across increasing cell counts (e.g. 5000–50000 cells) and report
  a runtime-vs-cell-count table.
when_to_use_negative:
- The goal is imputation accuracy rather than computational cost — this skill only
  measures runtime scaling and does not evaluate zero-imputation quality
- The dataset has far fewer than 10000 cells, where the reported method differences
  (e.g. scImpute/SAVER exceeding 10 hours) do not manifest
- Methods are run with multi-core/parallel execution — the benchmark's findings are
  specific to single-core wall-clock comparisons
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ALRA
  role: Imputation method under benchmark; the fast low-rank-approximation baseline
    whose runtime scaling is the focal result, run with default parameters and randomized
    SVD via the rsvd package
  repo: https://github.com/KlugerLab/ALRA
- name: MAGIC
  role: Baseline imputation method timed alongside ALRA; tied for fastest (~40 min
    on 50000 cells)
- name: DCA
  role: Baseline imputation method timed on each subset; takes about 4 h on 50000
    cells
- name: SAVER
  role: Baseline imputation method timed with its 'do.fast' option set to true; exceeds
    10 h at 10000 cells and was not attempted on larger matrices
- name: scImpute
  role: Baseline imputation method timed on each subset; exceeds 10 h at 10000 cells
    and was not attempted on larger matrices
- name: rsvd
  role: R package providing the randomized SVD used by ALRA during timing runs
  repo: https://github.com/KlugerLab/ALRA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:895e4513288e438566cffe18d67f3fe6221ecb8411d3d9ed0baca1d7ce4bcd2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - DCA
  - MAGIC
  - SAVER
  - rsvd
  - scImpute
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# imputation-method-runtime-benchmarking

## Summary

A benchmarking skill for measuring how the wall-clock runtime of scRNA-seq imputation methods (ALRA, MAGIC, DCA, scImpute, SAVER) scales with cell count on a single core, using the Hrvatin mouse visual cortex dataset. It quantifies the practical scalability claim that motivated ALRA's design.

## When to use

Use when you need to compare the computational scalability of scRNA-seq imputation methods across increasing cell counts (e.g. 5000–50000 cells) and report a runtime-vs-cell-count table.

## When NOT to use

- The goal is imputation accuracy rather than computational cost — this skill only measures runtime scaling and does not evaluate zero-imputation quality
- The dataset has far fewer than 10000 cells, where the reported method differences (e.g. scImpute/SAVER exceeding 10 hours) do not manifest
- Methods are run with multi-core/parallel execution — the benchmark's findings are specific to single-core wall-clock comparisons

## Inputs

- Hrvatin et al. (2018) mouse visual cortex scRNA-seq expression matrix filtered to 19155 genes x 65539 cells (genes with mean expression < 0.00003 or non-zero expression in fewer than 4 cells excluded)
- Random cell subsets of sizes 5000, 10000, 20000, 30000, 40000, and 50000 cells drawn from the full matrix

## Outputs

- runtime_table.csv: wall-clock times for ALRA, MAGIC, DCA, scImpute, and SAVER across the six subset sizes
- Per-method-per-subset wall-clock timing records from single-core execution

## How to apply

Load the Hrvatin et al. (2018) mouse visual cortex expression matrix filtered to 19155 genes and 65539 cells (excluding genes with mean expression < 0.00003 or non-zero expression in fewer than 4 cells). Sample random subsets of 5000, 10000, 20000, 30000, 40000, and 50000 cells. Run ALRA on each subset with default parameters (k=48 for full Hrvatin data, randomized SVD via the rsvd package, q=10 power iterations) on a single core, recording wall-clock time, then run MAGIC, DCA, scImpute, and SAVER (with SAVER's 'do.fast' set to true) under the same single-core restriction. Assemble all runtimes into a runtime-vs-cell-count table (runtime_table.csv). The rationale is that an imputation method 'should be able to process, in a reasonable time, large-scale scRNA-seq datasets that contain hundreds of thousands to millions of cells', so runtime scaling is a first-class evaluation criterion alongside imputation accuracy.

## Related tools

- **ALRA** (Imputation method under benchmark; the fast low-rank-approximation baseline whose runtime scaling is the focal result, run with default parameters and randomized SVD via the rsvd package) — https://github.com/KlugerLab/ALRA
- **MAGIC** (Baseline imputation method timed alongside ALRA; tied for fastest (~40 min on 50000 cells))
- **DCA** (Baseline imputation method timed on each subset; takes about 4 h on 50000 cells)
- **SAVER** (Baseline imputation method timed with its 'do.fast' option set to true; exceeds 10 h at 10000 cells and was not attempted on larger matrices)
- **scImpute** (Baseline imputation method timed on each subset; exceeds 10 h at 10000 cells and was not attempted on larger matrices)
- **rsvd** (R package providing the randomized SVD used by ALRA during timing runs) — https://github.com/KlugerLab/ALRA

## Examples

```
library(ALRA); A_norm <- normalize_data(expr_matrix); t0 <- Sys.time(); result.completed <- alra(A_norm, k=48); print(Sys.time() - t0)
```

## Evaluation signals

- runtime_table.csv reproduces the reported scaling pattern: ALRA and MAGIC take roughly 40 minutes on 50000 cells
- DCA takes about 4 hours on the 50000-cell subset
- scImpute and SAVER exceed 10 hours at 10000 cells and are therefore not attempted on larger subsets
- All five methods were restricted to a single core for each run, so timings are directly comparable
- The six subset sizes (5000–50000 cells) are all drawn from the same 19155-gene filtered Hrvatin matrix, so runtime differences reflect method cost, not input dimensionality changes

## Limitations

- scImpute and SAVER are extremely slow on large data — the article notes 'SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells' — so their runtimes at 20000+ cells could not be measured in this benchmark
- Timings are single-core wall-clock times; results will differ on multi-core or MKL-accelerated setups (e.g. ALRA's use.mkl flag via rpca-mkl gives 'dramatic speedups over the default rpca-based version')
- ALRA was run with k=48 fixed for the full Hrvatin data rather than the automatically chosen k, which may differ from other datasets' optimal rank

## Evidence

- [results] Runtime of ALRA vs baselines scales with cell count: [evidence span withheld — anchor status: not_found]
- [intro] Motivation: methods must scale to large datasets: "it should be able to process, in a reasonable time, large-scale scRNA-seq datasets that contain hundreds of thousands to millions of cells"
- [intro] scImpute and SAVER scalability limits: "SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells"
- [readme] ALRA dependency and MKL speedup option: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
- [intro] ALRA algorithm structure underlying its speed: "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
