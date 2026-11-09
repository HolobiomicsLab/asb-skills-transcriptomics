---
name: scrnaseq-runtime-benchmarking
description: Use when comparing computational scalability of rare-cell identification
  methods on scRNA-seq data, or when judging whether a method can process datasets
  in the tens-of-thousands-of-cells range.
when_to_use_negative:
- The goal is accuracy of rare-cell detection (precision/recall) rather than runtime
  scalability — this skill measures execution time only
- The dataset is small (hundreds of cells) where scalability differences between methods
  are negligible
- The candidate method cannot be run single-core or in a controlled environment, making
  wall-clock comparisons unfair
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: FiRE
  role: Benchmarked method; rareness-score algorithm whose single-core runtime is
    measured across subset sizes using its Python/C++ build
  repo: github.com/princethewinner/FiRE
- name: RaceID
  role: Baseline R-based rare-cell method whose runtime (e.g., ~79 h for 5k cells)
    is logged at each subset size
- name: GiniClust
  role: Baseline R-based rare-cell method whose runtime is logged until it terminates
    with a runtime error beyond ~45k profiles
- name: Python
  role: Environment in which FiRE is installed and run for timing; also used for building
    nested cell-count subsets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:0f74e5aeea585e1cb1efdfa026b47262d1b0c29cdc43c99b12d43cef6f94d0a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - GiniClust
  - Python
  - RaceID
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# scrnaseq-runtime-benchmarking

## Summary

A benchmarking protocol that measures single-core wall-clock execution times of rare-cell detection methods (FiRE, LOF, RaceID, GiniClust) as scRNA-seq input size scales from 1k to ~68k expression profiles, demonstrating FiRE's dramatic speed advantage.

## When to use

Use when comparing computational scalability of rare-cell identification methods on scRNA-seq data, or when judging whether a method can process datasets in the tens-of-thousands-of-cells range.

## When NOT to use

- The goal is accuracy of rare-cell detection (precision/recall) rather than runtime scalability — this skill measures execution time only
- The dataset is small (hundreds of cells) where scalability differences between methods are negligible
- The candidate method cannot be run single-core or in a controlled environment, making wall-clock comparisons unfair

## Inputs

- ~68k-cell human PBMC scRNA-seq expression matrix from GEO accession GSE94820
- Nested cell-count subsets (1k, 5k, 10k, 15k, ... ~68k cells) of the expression matrix
- FiRE Python/C++ module installed in the Python environment; R environment for RaceID and GiniClust

## Outputs

- Comparison table of wall-clock runtimes with one row per method per subset size
- Log of method failures (e.g., GiniClust runtime error beyond ~45k profiles)
- Validation flags where recorded runtimes deviate from Supplementary Table 1

## How to apply

Download the ~68k-cell human PBMC dataset (GEO GSE94820) and construct nested cell-count subsets (1k, 5k, 10k, 15k, ... up to the full ~68k profiles) from the expression matrix. Run each method single-core on every subset, recording wall-clock time per run and any failures: FiRE from its Python/C++ build (installed via ./INSTALL --py), LOF, and the R-based RaceID and GiniClust. Preprocess FiRE inputs with ranger_preprocess to retain the top 1000 variable genes, then instantiate the model with FiRE.FiRE(L=100, M=50, H=1017881, seed=5489) before timing fit/score. Compile all measured runtimes into one comparison table with one row per method per subset size, and flag any cell deviating materially from the paper's Supplementary Table 1 (FiRE ~26 s on ~68k PBMC profiles, GiniClust erroring beyond ~45k profiles, RaceID ~79 h for 5k cells).

## Related tools

- **FiRE** (Benchmarked method; rareness-score algorithm whose single-core runtime is measured across subset sizes using its Python/C++ build) — github.com/princethewinner/FiRE
- **RaceID** (Baseline R-based rare-cell method whose runtime (e.g., ~79 h for 5k cells) is logged at each subset size)
- **GiniClust** (Baseline R-based rare-cell method whose runtime is logged until it terminates with a runtime error beyond ~45k profiles)
- **Python** (Environment in which FiRE is installed and run for timing; also used for building nested cell-count subsets)

## Examples

```
[sudo] ./INSTALL --py && python example/jurkat_simulation_small.py
```

## Evaluation signals

- FiRE completes the full ~68k-cell PBMC data in ~26 s (paper reports a ~31 s class runtime for ~68k expression profiles)
- GiniClust raises a runtime error / terminates when subset size exceeds ~45k profiles
- RaceID requires ~79 h (or times out) on the 5k-cell subset
- Runtime table has one row per method per subset size covering the full 1k-to-68k range with no missing cells
- All runs are single-core so wall-clock times are directly comparable across methods

## Limitations

- Existing rare-cell methods other than FiRE scale unbearably slowly or terminate as sample size grows to the order of tens of thousands, capping the achievable comparison range
- The paper's reported ~31 s FiRE runtime on ~68k profiles is hardware-dependent; reproduced values will vary with machine specifications
- FiRE gives a continuous rareness score, while RaceID and GiniClust give binary predictions, so runtime comparison is not paired with an equivalent prediction interface

## Evidence

- [discussion] FiRE runtime on full dataset: "FiRE took ~31 s to analyze a scRNA-seq dataset containing ~68 k expression profiles."
- [abstract] Motivation for runtime benchmarking: "Existing algorithms to find rare cells scale unbearably slowly or terminate, as the sample size grows to the order of tens of thousands."
- [methods] Install FiRE in python environment: "--py                       : python        : Install FiRE in python environment."
- [readme] FiRE model instantiation parameters: [evidence span withheld — anchor status: external_doc]
- [abstract] FiRE runtime claim: "We propose Finder of Rare Entities (FiRE), an algorithm that, in a matter of seconds, assigns a rareness score to every individual expression profile under study."
