---
name: scaling-analysis-cell-count-vs-runtime
description: 'Use when comparing computational scalability of scRNA-seq methods: multiple
  tools must be run on size-graded subsamples of one dataset (e.g. the 68k PBMC set)
  and runtime/peak-memory curves plotted against cell number.'
when_to_use_negative:
- The goal is rare-cell detection accuracy (e.g. F1 score) rather than computational
  cost — use a detection-accuracy benchmarking skill instead
- The dataset cannot be subsampled across a size range, so no scaling relationship
  can be estimated
- Only a single tool is of interest — scalability curves require comparing multiple
  methods on identical data
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GapClust
  role: Benchmarked tool expected to finish in seconds and use ~3 GB at ~68k cells;
    reference point for scalability
  repo: github.com/fabotao/GapClust
- name: FiRE
  role: Benchmarked tool; second-fastest, >20 GB memory on the largest dataset, used
    as the speed comparison (~3x slower than GapClust)
- name: RaceID
  role: Benchmarked tool; infeasible on large data due to unrealistic running time
    and >78 GB memory at ~68k cells
- name: GiniClust
  role: Benchmarked tool; reported to fail on very large datasets and exceeds 78 GB
    memory at ~68k cells
- name: CellSIUS
  role: Benchmarked tool; exceeds 78 GB memory on the largest subsampled dataset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:bf68d5ba686c05a9604bc97c8971e722acbde3779393a55378bb52036cb71367
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellSIUS
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Scaling analysis of rare-cell detection tools: runtime and peak memory vs cell count

## Summary

A benchmarking skill that measures wall-clock running time and peak RAM of five rare-cell detection tools (RaceID, GiniClust, CellSIUS, FiRE, GapClust) across 16 subsampled scRNA-seq datasets scaled from ~1,000 to ~68,000 cells, to characterize computational scalability.

## When to use

Use when comparing computational scalability of scRNA-seq methods: multiple tools must be run on size-graded subsamples of one dataset (e.g. the 68k PBMC set) and runtime/peak-memory curves plotted against cell number.

## When NOT to use

- The goal is rare-cell detection accuracy (e.g. F1 score) rather than computational cost — use a detection-accuracy benchmarking skill instead
- The dataset cannot be subsampled across a size range, so no scaling relationship can be estimated
- Only a single tool is of interest — scalability curves require comparing multiple methods on identical data

## Inputs

- 16 subsampled scRNA-seq datasets derived from the 68k PBMC dataset, spanning ~1,000 to ~68,000 cells
- Installable implementations of RaceID, GiniClust, CellSIUS, FiRE and GapClust (R, with GapClust requiring R >= 3.4.0, Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1)

## Outputs

- Running-time table per tool and dataset (Supplementary Table 3 equivalent)
- Peak-memory (peak RAM) table per tool and dataset (Supplementary Table 4 equivalent)
- Plots of running time vs cell count and peak memory vs cell count for the five tools (Fig. 4a-b equivalent)

## How to apply

Download the 16 benchmark scRNA-seq datasets, which range from ~1,000 cells up to the full ~68,000-cell 68k PBMC dataset. Run each tool (RaceID, GiniClust, CellSIUS, FiRE, GapClust) on every dataset, recording wall-clock running time and peak RAM for every tool-dataset pair. Compile the per-tool measurements into tables equivalent to Supplementary Tables 3-4 (one for running time, one for memory). Plot running time vs number of cells and peak memory vs number of cells for all five tools to reproduce Fig. 4a-b. Decision point: verify scaling behavior against the published benchmark, where GapClust should use ~3 GB at ~68k cells, be about 3x faster than FiRE, while GiniClust, RaceID and CellSIUS exceed 78 GB on the largest data.

## Related tools

- **GapClust** (Benchmarked tool expected to finish in seconds and use ~3 GB at ~68k cells; reference point for scalability) — github.com/fabotao/GapClust
- **FiRE** (Benchmarked tool; second-fastest, >20 GB memory on the largest dataset, used as the speed comparison (~3x slower than GapClust))
- **RaceID** (Benchmarked tool; infeasible on large data due to unrealistic running time and >78 GB memory at ~68k cells)
- **GiniClust** (Benchmarked tool; reported to fail on very large datasets and exceeds 78 GB memory at ~68k cells)
- **CellSIUS** (Benchmarked tool; exceeds 78 GB memory on the largest subsampled dataset)

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- On the ~68,000-cell subsample, GapClust completes in seconds and is approximately 3x faster than FiRE
- GapClust peak memory at ~68k cells is ~3 GB, while FiRE uses >20 GB and GiniClust, RaceID and CellSIUS each exceed 78 GB
- Every tool-dataset pair has recorded wall-clock time and peak RAM, and tables match the structure of Supplementary Tables 3-4
- Plotted curves of time and memory vs cell count reproduce the qualitative scaling pattern of Fig. 4a-b across all 16 datasets
- Same input data and versions are used for all five tools so timing and memory differences are attributable to the methods

## Limitations

- Scalability conclusions rest on the 68k PBMC-derived subsamples; scaling on other tissues or technologies is not established by this benchmark
- RaceID, GiniClust and CellSIUS exceed 78 GB at ~68k cells, so runs on full-scale data may fail on typical hardware
- No version numbers for the compared tools are given in the article's discussion, and no GapClust parameter settings (e.g. K, skewness threshold with skewness<k> > 2) are reported for these runs
- Some result details are deferred to supplementary figures (e.g. Supplementary Figs. 14-22, Supplementary Table 5) that may not be linked in the main text

## Evidence

- [full_text] Scalability benchmark finding: [evidence span withheld — anchor status: not_found]
- [readme] README performance claim: "As to compuation time and memory utilization, GapClust also displays unrivaled speed and memory efficiency, in comparison with other methods."
- [intro] Intro motivation for efficiency benchmark: "scRNA-seq datasets containing thousands of cells preclude the use of RaceID due to unrealistic amount of running time"
- [readme] README prerequisites: "Seurat >=  3.1.0
rflann >=1.8.4 (GitHub: YeeJeremy/rflann)
irlba >=2.3.3
e1071 >= 1.7-0.1"
