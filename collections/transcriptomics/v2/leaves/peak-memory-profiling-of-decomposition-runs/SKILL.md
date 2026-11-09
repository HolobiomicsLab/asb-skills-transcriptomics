---
name: peak-memory-profiling-of-decomposition-runs
description: Use when two alternative matrix-decomposition or joint-dimensionality-reduction
  implementations must be compared for computational efficiency on the same scenario,
  and runtime, memory, and rank-selection agreement are the metrics of interest.
when_to_use_negative:
- The goal is statistical accuracy or batch-effect correction quality rather than
  computational cost — use downstream benchmarking against tools like Seurat v5, Harmony,
  LIGER, or ComBat-seq instead
- The implementations cannot be run on identical input data, making runtime and memory
  measurements incomparable
- The analysis targets small datasets where JIVE's original implementation is tractable,
  so profiling overhead adds little value
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: AJIVE (Python mvdr library)
  role: Fast joint-and-individual decomposition implementation whose wall-clock runtime
    and peak memory are measured on the shared simulation scenario
- name: r.jive
  role: Reference JIVE implementation in R; its jive() function with default permutation-based
    rank selection is run on the identical scenario for the baseline timing and memory
    measurement
- name: R
  role: Runtime environment hosting r.jive and the baseline decomposition run
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:e001d7a890c84996f847e0732cbe586d700c9318a7939b012a5288373fdc661a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AJIVE (Python mvdr library)
  - R
  - r.jive
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# peak-memory-profiling-of-decomposition-runs

## Summary

Benchmarks matrix-decomposition implementations (e.g., AJIVE from the Python mvdr library vs the R r.jive jive() function) on identical multi-view input by recording wall-clock runtime, peak memory usage, and the selected joint rank, enabling head-to-head computational comparisons such as the reported ~18 s / 601 MB vs ~2 h / 1180 MB result.

## When to use

Use when two alternative matrix-decomposition or joint-dimensionality-reduction implementations must be compared for computational efficiency on the same scenario, and runtime, memory, and rank-selection agreement are the metrics of interest.

## When NOT to use

- The goal is statistical accuracy or batch-effect correction quality rather than computational cost — use downstream benchmarking against tools like Seurat v5, Harmony, LIGER, or ComBat-seq instead
- The implementations cannot be run on identical input data, making runtime and memory measurements incomparable
- The analysis targets small datasets where JIVE's original implementation is tractable, so profiling overhead adds little value

## Inputs

- A single packaged simulation scenario (e.g., SimData from the r.jive vignette) shared identically by both implementations
- Python environment with the mvdr library providing AJIVE
- R environment with the r.jive package installed

## Outputs

- A comparison record containing runtime_s, memory_mb, and selected_joint_rank for each implementation
- Permutation-based joint rank selection outcome from each run (e.g., AJIVE joint rank 0)

## How to apply

Load a shared, packaged simulation dataset (e.g., the SimData scenario used in the JIVE vignette) so both implementations receive identical input. Run the first implementation — the Python AJIVE from the mvdr library, which uses permutation-based joint rank selection — while measuring wall-clock runtime and peak memory. Run the second implementation — the R r.jive function jive() with its default permutation-based rank selection — on the identical scenario under the same instrumentation. Compile the measurements into a single comparison record with three fields: runtime_s, memory_mb, and selected_joint_rank. Validation then checks that the record exists with all three fields and that cross-implementation rank selections are consistent (e.g., AJIVE selecting joint rank 0 as reported). This matters because JIVE's original implementation is documented as computationally infeasible for large single-cell datasets, motivating faster alternatives.

## Related tools

- **AJIVE (Python mvdr library)** (Fast joint-and-individual decomposition implementation whose wall-clock runtime and peak memory are measured on the shared simulation scenario)
- **r.jive** (Reference JIVE implementation in R; its jive() function with default permutation-based rank selection is run on the identical scenario for the baseline timing and memory measurement)
- **R** (Runtime environment hosting r.jive and the baseline decomposition run)

## Examples

```
install.packages("r.jive"); library(r.jive); res <- jive(data)  # measure runtime/peak memory around this call vs mvdr AJIVE
```

## Evaluation signals

- The output comparison file exists and contains all three required fields: runtime_s, memory_mb, and selected_joint_rank
- The AJIVE-selected joint rank matches the expected value (0 in the reported comparison), confirming consistent rank selection behavior
- Runtime ratio is consistent with the reported comparison (AJIVE ~18 s vs JIVE slightly over 2 h, roughly a 400x difference)
- Memory measurements fall near the reported values (AJIVE ~601 MB vs JIVE ~1180 MB, AJIVE roughly half of JIVE)
- Both runs consumed byte-identical input data (the same SimData scenario) so the measurements are directly comparable

## Limitations

- Results are specific to the chosen simulation scenario; timings and memory do not necessarily generalize to other data sizes or structures
- JIVE's original implementation is designed for bulk sequencing data and is computationally infeasible for large-scale single-cell sequencing datasets, so profiling it on such data may be impractical
- Permutation-based rank selection adds stochasticity, so selected_joint_rank may vary between runs and should be checked for consistency rather than assumed fixed

## Evidence

- [other] On a scenario from the simulation study, AJIVE fitted the model in 18 s versus slightly over 2 h for JIVE, and was more memory efficient, using 601 MB compared to the 1180 MB used by JIVE.: [evidence span withheld — anchor status: not_found]
- [other] Run the Python AJIVE implementation from the mvdr library on the selected scenario, measuring wall-clock runtime and peak memory usage, and capture its permutation-based joint rank selection outcome.: [evidence span withheld — anchor status: not_found]
- [other] Compile the two runs' measurements into a single record with fields runtime_s, memory_mb, and selected_joint_rank.: [evidence span withheld — anchor status: not_found]
- [abstract] However, its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets.: "its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets"
- [methods] The default function to estimate the JIVE decomposition is `jive`: "The default function to estimate the JIVE decomposition is `jive`"
