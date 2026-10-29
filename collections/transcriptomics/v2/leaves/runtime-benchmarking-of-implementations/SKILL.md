---
name: runtime-benchmarking-of-implementations
description: Use when two implementations of the same decomposition (e.g. JIVE variants)
  must be compared for computational efficiency on identical data, and runtime/memory
  gains must be validated against equivalence of the fitted decomposition.
when_to_use_negative:
- The goal is accuracy or biological performance comparison (e.g. batch-effect correction
  quality across Seurat v5, Harmony, LIGER, ComBat-seq) rather than computational
  efficiency of equivalent implementations
- The implementations do not target the same decomposition/model, so equivalence of
  variance proportions or selected joint rank cannot be used as a validation criterion
- Only a single run per implementation is affordable and variance in wall-clock timing
  cannot be averaged over repeated runs (the benchmark protocol relies on ~20 repetitions)
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: r.jive
  role: Original R implementation of the JIVE decomposition whose runtime (jive()
    with permutation-based rank selection) is benchmarked as the baseline
- name: JIVE (scJIVE)
  role: Optimized JIVE implementation whose per-run wall-clock times are recorded
    over 20 repetitions and compared to r.jive on identical simulated data
  repo: github.com/oconnell-statistics-lab/scJIVE
- name: AJIVE (Python mvdr library)
  role: Python AJIVE implementation timed and memory-profiled against r.jive::jive()
    on the same simulation scenario, including its permutation-based joint rank selection
    outcome
- name: R
  role: Language in which both the r.jive baseline runs and scJIVE benchmark runs
    are executed and timed
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:108d1fdf07a6a8ea82e271f4833523b5024a92fd0eef56d21aee9ceb05e13517
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AJIVE (Python mvdr library)
  - JIVE (scJIVE)
  - R
  - r.jive
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# runtime-benchmarking-of-implementations

## Summary

Benchmark wall-clock runtime and memory usage of alternative implementations of the same statistical method (e.g. original r.jive::jive vs. the optimized scJIVE, or r.jive vs. Python AJIVE/mvdr) on identical simulated input, then verify that outputs (variance proportions, selected ranks) are equivalent before declaring a speedup valid.

## When to use

Use when two implementations of the same decomposition (e.g. JIVE variants) must be compared for computational efficiency on identical data, and runtime/memory gains must be validated against equivalence of the fitted decomposition.

## When NOT to use

- The goal is accuracy or biological performance comparison (e.g. batch-effect correction quality across Seurat v5, Harmony, LIGER, ComBat-seq) rather than computational efficiency of equivalent implementations
- The implementations do not target the same decomposition/model, so equivalence of variance proportions or selected joint rank cannot be used as a validation criterion
- Only a single run per implementation is affordable and variance in wall-clock timing cannot be averaged over repeated runs (the benchmark protocol relies on ~20 repetitions)

## Inputs

- SimData: two 200x1000 matrices with shared rank-1 joint structure, rank-1 individual structures per matrix, and standard-normal residual error
- A selected scenario from the packaged simulation study (shared input for all implementations)
- Number of repetitions per implementation (e.g. 20 runs)

## Outputs

- runtime_benchmark.csv: mean per-run wall-clock times per implementation and their ratio
- variance_proportions.csv: proportion of variation explained by joint, individual, and residual components under each implementation
- Comparison record with fields runtime_s, memory_mb, selected_joint_rank per implementation

## How to apply

Generate or load one identical input for all implementations — e.g. SimData: two 200x1000 matrices sharing a common joint structure of rank 1, each with a unique rank-1 individual structure plus standard-normal residual error, or one scenario from the packaged simulation study. Run each implementation the same number of times (20 repetitions in the scJIVE benchmark) recording per-run wall-clock time, and additionally record peak memory (MB) and the selected joint rank when comparing e.g. AJIVE (mvdr) against r.jive's permutation-based rank selection. Summarize mean runtimes and their ratio, and write comparison tables (runtime_benchmark.csv, variance_proportions.csv or a record with runtime_s, memory_mb, selected_joint_rank). Validate by checking that the runtime ratio falls in the expected range (~8-10x, ~35.8 s for r.jive vs ~4.1 s for scJIVE; ~18 s AJIVE vs ~2 h JIVE, 601 MB vs 1180 MB) AND that the decomposition outputs — joint/individual/residual variance proportions or the selected joint rank — are near-identical between implementations; a speedup with divergent results fails validation.

## Related tools

- **r.jive** (Original R implementation of the JIVE decomposition whose runtime (jive() with permutation-based rank selection) is benchmarked as the baseline)
- **JIVE (scJIVE)** (Optimized JIVE implementation whose per-run wall-clock times are recorded over 20 repetitions and compared to r.jive on identical simulated data) — github.com/oconnell-statistics-lab/scJIVE
- **AJIVE (Python mvdr library)** (Python AJIVE implementation timed and memory-profiled against r.jive::jive() on the same simulation scenario, including its permutation-based joint rank selection outcome)
- **R** (Language in which both the r.jive baseline runs and scJIVE benchmark runs are executed and timed)

## Examples

```
library(r.jive); times <- replicate(20, system.time(jive(SimData))['elapsed']); mean(times)
```

## Evaluation signals

- Runtime ratio between implementations falls in the ~8-10x range (mean ~35.8 s for r.jive vs ~4.1 s for scJIVE over 20 runs)
- Joint, individual, and residual variance proportions are near-identical between the two JIVE implementations
- AJIVE-vs-JIVE comparison record contains all three required fields (runtime_s, memory_mb, selected_joint_rank) and the AJIVE-selected joint rank is 0
- Expected AJIVE-vs-JIVE magnitudes reproduced: ~18 s vs ~2 h runtime and ~601 MB vs ~1180 MB peak memory
- Per-run wall-clock times are recorded for every repetition (20 runs per implementation) rather than a single timing

## Limitations

- The original JIVE implementation was designed for bulk sequencing data and is computationally infeasible for large-scale single-cell datasets — runtime benchmarks on small simulated matrices (200x1000) do not directly transfer to full scRNA-seq scale
- Validation depends on equivalence of the decomposition (variance proportions or selected joint rank); rank-selection differences such as AJIVE selecting joint rank 0 versus JIVE change what 'equivalent results' means for the comparison
- Benchmarks were performed on simulated data with rank-1 joint and individual structures; runtimes and memory on real multi-source data (e.g. the BRCA TCGA multi-omics data) may differ

## Evidence

- [results] r.jive vs scJIVE runtime ratio: [evidence span withheld — anchor status: not_found]
- [results] equivalence of decomposition: [evidence span withheld — anchor status: not_found]
- [results] AJIVE vs JIVE time and memory: [evidence span withheld — anchor status: not_found]
- [abstract] original JIVE infeasibility for scRNA-seq: "its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets"
- [supplementary] AJIVE joint rank outcome: [evidence span withheld — anchor status: not_found]
