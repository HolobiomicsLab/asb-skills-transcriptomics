---
name: runtime-benchmark-measurement
description: Use when you have replaced or re-implemented a performance-critical component
  (e.g. an interval query structure) and must prove both functional equivalence with
  the original and a measured runtime improvement on the real workload.
when_to_use_negative:
- The component has not been re-implemented or modified — benchmarking against the
  unmodified baseline provides no equivalence or improvement evidence.
- The goal is functional comparison alone without timing — use plain equivalence testing
  rather than a benchmark workflow.
- Timing cannot be compared fairly (e.g. differing hardware or non-deterministic workloads),
  since the 30% reduction figure is explicitly hardware-dependent.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Cython
  role: Language used to write the compiled piecewise-constant interval data structure
    (StepVector replacement) that is benchmarked against HTSeq's implementation
- name: Python
  role: Language of the tiny-count codebase into which the Cython StepVector is compiled
    and against which equivalence tests and benchmarks are run
- name: tiny-count
  role: Counting tool whose read-assignment workload provides the benchmark scenario;
    its custom StepVector is the implementation under test
  repo: https://github.com/MontgomeryLab/tinyRNA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:91c335ec52f7e00b8e946502842fe6a5ba3b19ef2fd21c369440f71b2f579c11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - Cython
  - Python
  - tiny-count
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# runtime-benchmark-measurement

## Summary

Benchmark a re-implemented data structure against the original by issuing identical queries to both implementations and recording per-run wall-clock times, then computing the average runtime reduction. Used here to validate that tiny-count's Cython StepVector matches HTSeq's StepVector exactly while running the read-assignment workload faster.

## When to use

Use when you have replaced or re-implemented a performance-critical component (e.g. an interval query structure) and must prove both functional equivalence with the original and a measured runtime improvement on the real workload.

## When NOT to use

- The component has not been re-implemented or modified — benchmarking against the unmodified baseline provides no equivalence or improvement evidence.
- The goal is functional comparison alone without timing — use plain equivalence testing rather than a benchmark workflow.
- Timing cannot be compared fairly (e.g. differing hardware or non-deterministic workloads), since the 30% reduction figure is explicitly hardware-dependent.

## Inputs

- Cython StepVector replacement supporting interval-overlap queries (compiled with the Cython compiler alongside the tiny-count Python codebase)
- Original HTSeq StepVector implementation
- Feature sets for equivalence-test query intervals
- Small RNA-seq dataset for the tiny-count read-assignment workload

## Outputs

- stepvector_benchmark.csv containing equivalence flags and per-run wall-clock timing measurements
- Average runtime reduction of the Cython StepVector relative to the HTSeq StepVector (~30%, hardware-dependent)
- Equivalence test results asserting identical returned feature sets for every query interval

## How to apply

First implement the replacement as a compiled extension — in this case a piecewise-constant interval data structure (StepVector) in Cython, supporting interval-overlap queries against stored feature intervals — and compile it alongside the existing Python codebase. Second, run equivalence tests that issue identical interval-overlap queries to both the original HTSeq StepVector and the Cython implementation over the same feature sets, asserting that returned feature sets match exactly for every query interval. Third, benchmark both implementations on the actual workload (tiny-count read-assignment on the tested small RNA-seq dataset), recording per-run wall-clock times for each. Finally, compute the average runtime reduction of the new implementation relative to the baseline across benchmark runs — expected on the order of 30%, hardware-dependent — and write the equivalence flags and timing measurements to a results file (stepvector_benchmark.csv).

## Related tools

- **Cython** (Language used to write the compiled piecewise-constant interval data structure (StepVector replacement) that is benchmarked against HTSeq's implementation)
- **Python** (Language of the tiny-count codebase into which the Cython StepVector is compiled and against which equivalence tests and benchmarks are run)
- **tiny-count** (Counting tool whose read-assignment workload provides the benchmark scenario; its custom StepVector is the implementation under test) — https://github.com/MontgomeryLab/tinyRNA

## Examples

```
python benchmark_stepvector.py --impl cython --impl htseq --features reference_data/ram1.gff3 --workload read-assignment --output stepvector_benchmark.csv
```

## Evaluation signals

- All interval-overlap query results are identical between the Cython StepVector and HTSeq StepVector — zero mismatches across test feature sets.
- The measured average runtime on the read-assignment workload is reduced relative to the HTSeq baseline (expected on the order of 30%, hardware-dependent).
- stepvector_benchmark.csv exists and contains both equivalence flags and per-run wall-clock times for each implementation.
- Benchmark comparisons are computed across multiple runs (per-run wall-clock times recorded) rather than a single timing measurement.

## Limitations

- The ~30% average runtime reduction is hardware-dependent and may not reproduce on other machines.
- Acceptance requires zero query mismatches — any single interval-overlap query returning a different feature set from HTSeq StepVector disqualifies the Cython implementation.
- Timing comparisons are only meaningful for the tested small RNA-seq dataset and the read-assignment workload; they may not generalize to other feature sets or workloads.

## Evidence

- [full_text] custom Cython StepVector motivation: [evidence span withheld — anchor status: not_found]
- [full_text] equivalence testing method: [evidence span withheld — anchor status: not_found]
- [full_text] benchmark and metric: [evidence span withheld — anchor status: not_found]
- [readme] language stack from README: "tiny-count and other tinyRNA tools are implemented in Python, C++, Cython, and R, and the workflow is coordinated with CWL."
