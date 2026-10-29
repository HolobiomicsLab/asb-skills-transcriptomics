---
name: interval-overlap-equivalence-testing
description: Use when a Python interval/feature data structure (e.g. HTSeq StepVector)
  is a measured bottleneck in read-to-feature assignment and a compiled (Cython) replacement
  must be proven query-equivalent before adoption.
when_to_use_negative:
- You are changing query semantics (e.g. different overlap rules or returned feature
  sets) — equivalence testing asserts exact matching against HTSeq StepVector behavior,
  not redesigned behavior
- The interval data structure is not on the measured critical path — the skill justifies
  a Cython rewrite only via a demonstrated average runtime reduction on the read-assignment
  workload
- Features are already a pre-built count matrix; this skill operates on interval-overlap
  resolution, not downstream counting outputs
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Cython
  role: C language extension used to implement the piecewise-constant StepVector replacement
    compiled alongside the tiny-count Python codebase
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: tiny-count
  role: Read-assignment workload on which both StepVector implementations are benchmarked
  repo: https://anaconda.org/bioconda/tiny-count
- name: Python
  role: Host language in which the Cython extension is compiled and the equivalence
    tests and benchmarks are run
  repo: https://github.com/MontgomeryLab/tinyRNA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:b59fdb0995b0e3785d8dbac95a34ab96077a9419030775edd4cb416f6baac15c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - Cython
  - Python
  - tiny-count
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# interval-overlap-equivalence-testing

## Summary

A validation and benchmarking method for a custom Cython piecewise-constant StepVector that replaces HTSeq's StepVector for resolving features overlapping an alignment interval in tiny-count. It establishes output equivalence via identical interval-overlap query results and quantifies the runtime gain (~30% average reduction) against the HTSeq baseline.

## When to use

Use when a Python interval/feature data structure (e.g. HTSeq StepVector) is a measured bottleneck in read-to-feature assignment and a compiled (Cython) replacement must be proven query-equivalent before adoption.

## When NOT to use

- You are changing query semantics (e.g. different overlap rules or returned feature sets) — equivalence testing asserts exact matching against HTSeq StepVector behavior, not redesigned behavior
- The interval data structure is not on the measured critical path — the skill justifies a Cython rewrite only via a demonstrated average runtime reduction on the read-assignment workload
- Features are already a pre-built count matrix; this skill operates on interval-overlap resolution, not downstream counting outputs

## Inputs

- Feature annotation sets used in tiny-count read-assignment (e.g. GFF3/GFF2/GTF reference annotations)
- Interval-overlap query intervals from the tiny-count read-assignment workload
- Tested small RNA-seq dataset for the benchmark runs

## Outputs

- Per-query equivalence flags (Cython vs HTSeq returned feature sets, asserted exact match)
- Per-run wall-clock times for each implementation on the read-assignment workload
- stepvector_benchmark.csv containing equivalence flags and timing measurements, and the average runtime reduction (~30%)

## How to apply

Implement the piecewise-constant interval data structure in Cython, compiled with the Cython compiler alongside the tiny-count Python codebase, supporting interval-overlap queries against stored feature intervals. Run equivalence tests that issue identical interval-overlap queries to both the original HTSeq StepVector and the Cython implementation over the same feature sets, asserting that returned feature sets match exactly for every query interval — accept the replacement only with zero mismatches across all test feature sets. Benchmark both implementations on the tiny-count read-assignment workload using the tested small RNA-seq dataset, recording per-run wall-clock times for each implementation. Compute the average runtime reduction of the Cython StepVector relative to the HTSeq StepVector across benchmark runs (expected on the order of 30%, hardware-dependent) and write the equivalence flags and timing measurements to stepvector_benchmark.csv. The rationale is that the HTSeq StepVector incurs inefficiencies in iteration and intermediate object creation that a compiled Cython version eliminates without changing query semantics.

## Related tools

- **Cython** (C language extension used to implement the piecewise-constant StepVector replacement compiled alongside the tiny-count Python codebase) — https://github.com/MontgomeryLab/tinyRNA
- **tiny-count** (Read-assignment workload on which both StepVector implementations are benchmarked) — https://anaconda.org/bioconda/tiny-count
- **Python** (Host language in which the Cython extension is compiled and the equivalence tests and benchmarks are run) — https://github.com/MontgomeryLab/tinyRNA

## Evaluation signals

- Zero mismatches: returned feature sets match exactly for every interval-overlap query issued to both implementations over the same feature sets
- Equivalence flags in stepvector_benchmark.csv are all true across the tested feature sets
- Average wall-clock time on the tiny-count read-assignment workload is reduced relative to the HTSeq StepVector baseline (expected on the order of 30%, hardware-dependent)
- Per-run timing entries exist for both implementations for each benchmark run, allowing average-reduction computation
- The Cython implementation compiles successfully with the Cython compiler alongside the tiny-count Python codebase

## Limitations

- The ~30% average runtime reduction is expected rather than guaranteed — it is hardware-dependent and must be re-measured on the target system
- Equivalence is only established for the feature sets and query intervals tested; untested feature-set configurations could expose divergences
- tiny-count and other tinyRNA tools are implemented in Python, C++, Cython, and R with CWL coordination, so the Cython extension adds a compilation step to installation (and the README notes installation issues may arise on Apple Silicon–based systems)

## Evidence

- [other] Cython custom StepVector motivation: [evidence span withheld — anchor status: not_found]
- [other] Equivalence testing procedure: [evidence span withheld — anchor status: not_found]
- [other] Benchmark output artifact: [evidence span withheld — anchor status: not_found]
- [abstract] Language stack from abstract: "tiny-count and other tinyRNA tools are implemented in Python, C++, Cython, and R, and the workflow is coordinated with CWL."
- [readme] tiny-count role in pipeline from README: [evidence span withheld — anchor status: external_doc]
