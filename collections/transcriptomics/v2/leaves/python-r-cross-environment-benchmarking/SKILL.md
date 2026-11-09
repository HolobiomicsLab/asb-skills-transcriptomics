---
name: python-r-cross-environment-benchmarking
description: Use when comparing wall-clock execution times of rare-cell detection
  methods (FiRE, LOF, RaceID, GiniClust) across increasing scRNA-seq cell counts,
  e.g. 1k to ~68k profiles, on a single CPU core.
when_to_use_negative:
- The goal is biological rare-cell discovery accuracy (recall/precision) rather than
  runtime/scalability benchmarking — this skill measures only execution time and failure
  behavior
- Data is smaller than a few thousand cells where all compared methods complete quickly
  and runtime differences are not the decision factor
- Input is not an expression matrix that can be subset by cell count (e.g. precomputed
  embeddings or already-clustered results)
tools:
- name: FiRE
  role: Python/C++ rareness-scoring method being benchmarked; installed via './INSTALL
    --py' and run with L=100, M=50, H=1017881, seed=5489 on each subset
  repo: github.com/princewinner/FiRE
- name: LOF
  role: Python baseline outlier-detection method timed single-core on the same subsets
- name: RaceID
  role: R-based rare-cell method timed single-core on the same subsets (observed ~79
    h for 5k cells)
- name: GiniClust
  role: R-based rare-cell method timed single-core on the same subsets (runtime error
    beyond ~45k profiles)
- name: Python
  role: Environment in which FiRE is installed and the timing harness is run
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:fea95464335729aaad65c9e05a0b945ece7aa0bbff70a19f8ab51da330404bda
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# python-r-cross-environment-benchmarking

## Summary

Benchmark the single-core runtime of FiRE (installed as a Python/C++ module via ./INSTALL --py) against LOF, RaceID and GiniClust on nested subsets of the ~68k-cell PBMC scRNA-seq dataset (GSE94820), scaling from 1k to the full ~68k expression profiles to quantify scalability differences.

## When to use

Use when comparing wall-clock execution times of rare-cell detection methods (FiRE, LOF, RaceID, GiniClust) across increasing scRNA-seq cell counts, e.g. 1k to ~68k profiles, on a single CPU core.

## When NOT to use

- The goal is biological rare-cell discovery accuracy (recall/precision) rather than runtime/scalability benchmarking — this skill measures only execution time and failure behavior
- Data is smaller than a few thousand cells where all compared methods complete quickly and runtime differences are not the decision factor
- Input is not an expression matrix that can be subset by cell count (e.g. precomputed embeddings or already-clustered results)

## Inputs

- Human PBMC scRNA-seq expression matrix from GEO GSE94820 (~68k cells)
- Nested cell-count subsets (1k, 5k, 10k, 15k, ... ~68k profiles) of the expression matrix
- FiRE Python module (requires python 2.7/3, cython >= 0.23.4, g++ >= 4.8.4, boost >= 1.54.0)
- LOF implementation (Python), RaceID and GiniClust R packages

## Outputs

- Per-run wall-clock times for each method on each subset size
- Log of failures (e.g. GiniClust runtime error beyond ~45k cells, RaceID ~79 h runtime at 5k cells)
- Single comparison table with one row per method per subset size, cross-checked against Supplementary Table 1

## How to apply

Download the ~68k-cell human PBMC expression matrix (GEO accession GSE94820) and construct nested cell-count subsets (1k, 5k, 10k, 15k, ... up to ~68k profiles). Install FiRE in the Python environment with './INSTALL --py' (and optionally in R with './INSTALL --R'), then run FiRE from Python with L=100 estimators, M=50 sampled features, H=1017881 hash bins, seed=5489 on each subset, recording wall-clock time per run on a single core. Run LOF (Python/sklearn) and the R-based methods RaceID and GiniClust under identical single-core conditions, logging completion times and any failures. Compile all measured runtimes into one comparison table with one row per method per subset size; FiRE should finish the full ~68k data in seconds (~26 s measured, paper reports ~31 s) while RaceID can take ~79 h for just 5k cells and GiniClust can hit a runtime error beyond ~45k profiles.

## Related tools

- **FiRE** (Python/C++ rareness-scoring method being benchmarked; installed via './INSTALL --py' and run with L=100, M=50, H=1017881, seed=5489 on each subset) — github.com/princewinner/FiRE
- **LOF** (Python baseline outlier-detection method timed single-core on the same subsets)
- **RaceID** (R-based rare-cell method timed single-core on the same subsets (observed ~79 h for 5k cells))
- **GiniClust** (R-based rare-cell method timed single-core on the same subsets (runtime error beyond ~45k profiles))
- **Python** (Environment in which FiRE is installed and the timing harness is run)

## Examples

```
[sudo] ./INSTALL --py && python example/jurkat_simulation_small.py
```

## Evaluation signals

- FiRE completes the full ~68k-cell PBMC dataset in roughly 26 s (paper reports ~31 s for ~68k profiles); materially longer runs indicate a configuration or environment problem
- GiniClust fails with a runtime error when the subset exceeds ~45k expression profiles
- RaceID does not complete in reasonable time at 5k cells (~79 h reported); flag any faster recorded value as suspect
- Runtime table has one row per method per subset size, with no missing cells for FiRE/LOF across all 1k-to-68k subsets
- FiRE runtimes grow sub-linearly/consistently across subsets, and FiRE is orders of magnitude faster than RaceID and GiniClust at every comparable size

## Limitations

- Benchmark reflects single-core wall-clock time on a specific machine class; absolute times are not portable across hardware
- GiniClust cannot be benchmarked at large sizes because it terminates beyond ~45k profiles, so the comparison table is incomplete at the high end
- RaceID's ~79 h runtime at 5k cells makes full-scaling runs for that method effectively infeasible

## Evidence

- [discussion] FiRE ~31 s on 68k profiles: "FiRE took ~31 s to analyze a scRNA-seq dataset containing ~68 k expression profiles."
- [abstract] Existing methods scale slowly or terminate: "Existing algorithms to find rare cells scale unbearably slowly or terminate, as the sample size grows to the order of tens of thousands."
- [readme] Install FiRE in python environment: "--py                       : python        : Install FiRE in python environment."
- [abstract] FiRE assigns rareness scores in seconds: "We propose Finder of Rare Entities (FiRE), an algorithm that, in a matter of seconds, assigns a rareness score to every individual expression profile under study."
