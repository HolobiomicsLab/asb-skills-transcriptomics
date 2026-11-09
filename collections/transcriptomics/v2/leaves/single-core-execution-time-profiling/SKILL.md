---
name: single-core-execution-time-profiling
description: Use when comparing runtime scalability of scRNA-seq rare-cell detection
  algorithms as input cell count grows from ~1k to tens of thousands of expression
  profiles, under single-core execution.
when_to_use_negative:
- The analysis goal is rare-cell detection accuracy (recall/precision) rather than
  runtime scaling — FiRE also returns rareness scores, but this skill only measures
  execution time
- Data is already small (fewer than a few thousand cells), where runtime differences
  among methods are not the bottleneck
- Multi-core or parallel timing is required — the benchmark protocol is explicitly
  single-core
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FiRE
  role: Fast method being benchmarked; single-core runtimes recorded for each subset
    up to the full ~68k profiles (~26 s observed, ~31 s reported)
  repo: github.com/princethewinner/FiRE
- name: LOF
  role: Baseline density-based method benchmarked for runtime against FiRE
- name: RaceID
  role: R-based rare-cell method benchmarked; required ~79 h on the 5k-cell subset,
    showing poor scaling
- name: GiniClust
  role: R-based rare-cell method benchmarked; hit a runtime error beyond ~45k profiles
- name: Python
  role: Hosts the FiRE module (all manuscript results generated with the python module)
    and the timing harness used to record wall-clock times
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:3990b68fe10769de0d62442ca27c575da44543ec03958543bf02a1affbee944e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - GiniClust
  - LOF
  - Python
  - RaceID
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-core-execution-time-profiling

## Summary

Benchmarks single-core wall-clock execution time of rare-cell detection methods (FiRE, LOF, RaceID, GiniClust) across nested cell-count subsets of a scRNA-seq dataset, from 1k up to ~68k profiles, to expose runtime scaling differences.

## When to use

Use when comparing runtime scalability of scRNA-seq rare-cell detection algorithms as input cell count grows from ~1k to tens of thousands of expression profiles, under single-core execution.

## When NOT to use

- The analysis goal is rare-cell detection accuracy (recall/precision) rather than runtime scaling — FiRE also returns rareness scores, but this skill only measures execution time
- Data is already small (fewer than a few thousand cells), where runtime differences among methods are not the bottleneck
- Multi-core or parallel timing is required — the benchmark protocol is explicitly single-core

## Inputs

- scRNA-seq expression matrix with ~68k human PBMC cells (GEO accession GSE94820)
- Nested cell-count subsets (1k, 5k, 10k, 15k, ..., ~68k cells) derived from the expression matrix
- FiRE Python/C++ installation (via ./INSTALL --py, with g++ >= 4.8.4, boost >= 1.54.0)
- R environment with RaceID and GiniClust installed

## Outputs

- Wall-clock runtime per method per subset size (single-core)
- Comparison table with one row per method per cell-count subset
- Failure log for methods that terminate or error out (e.g. GiniClust beyond ~45k profiles)

## How to apply

Download a large scRNA-seq expression matrix (e.g. the ~68k-cell human PBMC dataset from GEO GSE94820) and construct nested subsets of 1k, 5k, 10k, 15k ... up to the full ~68k profiles. Install FiRE in the Python environment via the project's INSTALL script (`./INSTALL --py`, requires g++ >= 4.8.4 and boost >= 1.54.0), after preprocessing each subset with `ranger_preprocess` to keep the top 1000 variable genes. Run FiRE, LOF, and the R-based methods RaceID and GiniClust single-core on every subset, recording wall-clock time per run and any failures. Compile all runtimes into one table with a row per method per subset size, and validate against the paper's reference values: FiRE finishes the full ~68k data in ~26 s (paper reports ~31 s), GiniClust hits a runtime error beyond ~45k profiles, and RaceID needs ~79 h for just 5k cells.

## Related tools

- **FiRE** (Fast method being benchmarked; single-core runtimes recorded for each subset up to the full ~68k profiles (~26 s observed, ~31 s reported)) — github.com/princethewinner/FiRE
- **LOF** (Baseline density-based method benchmarked for runtime against FiRE)
- **RaceID** (R-based rare-cell method benchmarked; required ~79 h on the 5k-cell subset, showing poor scaling)
- **GiniClust** (R-based rare-cell method benchmarked; hit a runtime error beyond ~45k profiles)
- **Python** (Hosts the FiRE module (all manuscript results generated with the python module) and the timing harness used to record wall-clock times)

## Examples

```
python example/jurkat_simulation_small.py
```

## Evaluation signals

- FiRE completes the full ~68k-cell PBMC dataset in ~26 s single-core (paper reports ~31 s class runtimes); a materially larger value flags an error
- GiniClust terminates with a runtime error at subset sizes beyond ~45k cells
- RaceID requires ~79 h on the 5k-cell subset or times out
- The comparison table contains exactly one row per method per subset size, with no missing timing entries for runs that completed
- Every run is executed single-core so timing values are comparable across methods

## Limitations

- GiniClust cannot scale beyond ~45k profiles and errors out, so its row stops at that subset size
- RaceID is extremely slow (~79 h for 5k cells), making full-scaling benchmarks with it impractical
- Preprocessing of the full demo data may require a large amount of RAM to load before timing FiRE itself
- FiRE assigns a rareness score but does not discriminate between an outlier and cells representing minor cell types — timing results say nothing about detection quality

## Evidence

- [discussion] FiRE is remarkably faster than existing methods: "FiRE took ~31 s to analyze a scRNA-seq dataset containing ~68 k expression profiles."
- [abstract] Existing methods fail at scale: "Existing algorithms to find rare cells scale unbearably slowly or terminate, as the sample size grows to the order of tens of thousands."
- [readme] All benchmark timings produced with the Python module: [evidence span withheld — anchor status: external_doc]
- [other] Timing benchmark conditions from source card: [evidence span withheld — anchor status: not_found]
