---
name: runtime-memory-profiling-scrnaseq-tools
description: 'Use when comparing computational scalability of scRNA-seq rare-cell
  detection methods: you need running time and peak memory measured across subsampled
  datasets spanning ~1,000 to ~68,000 cells.'
when_to_use_negative:
- The goal is rare-cell detection accuracy (e.g. F1 score) rather than computational
  scalability — use an accuracy benchmarking workflow instead
- Datasets are far smaller than ~1,000 cells or are not single-cell expression profiles,
  so the runtime/memory scaling signal of interest does not exist
- Only one tool is of interest and no cross-method comparison of speed or memory is
  needed
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GapClust
  role: Light-weight rare-cell detection tool benchmarked as the reference for speed
    and memory efficiency; expected to use ~3 GB at 68k cells and run ~3x faster than
    FiRE
  repo: github.com/fabotao/GapClust
- name: FiRE
  role: Rare-cell detection tool benchmarked; the speed reference point GapClust is
    compared against on the largest dataset (>20 GB peak memory)
- name: RaceID
  role: Rare-cell detection tool benchmarked; reported to have unrealistic running
    time on datasets with thousands of cells (>78 GB peak memory at ~68k cells)
- name: GiniClust
  role: Rare-cell detection tool benchmarked; reported to fail on very large datasets
    (>78 GB peak memory at ~68k cells)
- name: CellSIUS
  role: Rare-cell detection tool benchmarked; requires major cell types to be determined
    and exceeds 78 GB peak memory at ~68k cells
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:5095e21117a7f6db43a9112afd1ccb28ce4b031f74b1de78525177eb8c7330a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellSIUS
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# runtime-memory-profiling-scrnaseq-tools

## Summary

Benchmarks wall-clock running time and peak RAM of rare-cell detection tools (RaceID, GiniClust, CellSIUS, FiRE, GapClust) as input size grows, using 16 subsampled datasets derived from the 68k PBMC scRNA-seq dataset to reproduce Fig. 4a-b and Supplementary Tables 3-4.

## When to use

Use when comparing computational scalability of scRNA-seq rare-cell detection methods: you need running time and peak memory measured across subsampled datasets spanning ~1,000 to ~68,000 cells.

## When NOT to use

- The goal is rare-cell detection accuracy (e.g. F1 score) rather than computational scalability — use an accuracy benchmarking workflow instead
- Datasets are far smaller than ~1,000 cells or are not single-cell expression profiles, so the runtime/memory scaling signal of interest does not exist
- Only one tool is of interest and no cross-method comparison of speed or memory is needed

## Inputs

- 16 subsampled scRNA-seq datasets derived from the 68k PBMC dataset, spanning ~1,000 to ~68,000 cells
- Installable versions of RaceID, GiniClust, CellSIUS, FiRE and GapClust (GapClust requires R >= 3.4.0, Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1)

## Outputs

- Per tool-dataset pair wall-clock running time measurements (Supplementary Table 3 equivalent)
- Per tool-dataset pair peak RAM (memory) measurements (Supplementary Table 4 equivalent)
- Scaling plots: running time vs. number of cells and peak memory vs. number of cells for all five tools (Fig. 4a-b equivalent)

## How to apply

Obtain 16 benchmark scRNA-seq datasets subsampled from the 68k PBMC dataset, ranging from ~1,000 to ~68,000 cells. Run each of the five tools (RaceID, GiniClust, CellSIUS, FiRE, GapClust) on every dataset, recording wall-clock running time and peak RAM for every tool-dataset pair. Compile measurements into two tables (running time and peak memory) equivalent to Supplementary Tables 3-4, then plot running time vs. number of cells and peak memory vs. number of cells to reproduce Fig. 4a-b. Validate against the reported benchmarks: at ~68,000 cells GapClust should use ~3 GB of memory and complete ~3x faster than FiRE, while FiRE exceeds 20 GB and GiniClust, RaceID and CellSIUS exceed 78 GB.

## Related tools

- **GapClust** (Light-weight rare-cell detection tool benchmarked as the reference for speed and memory efficiency; expected to use ~3 GB at 68k cells and run ~3x faster than FiRE) — github.com/fabotao/GapClust
- **FiRE** (Rare-cell detection tool benchmarked; the speed reference point GapClust is compared against on the largest dataset (>20 GB peak memory))
- **RaceID** (Rare-cell detection tool benchmarked; reported to have unrealistic running time on datasets with thousands of cells (>78 GB peak memory at ~68k cells))
- **GiniClust** (Rare-cell detection tool benchmarked; reported to fail on very large datasets (>78 GB peak memory at ~68k cells))
- **CellSIUS** (Rare-cell detection tool benchmarked; requires major cell types to be determined and exceeds 78 GB peak memory at ~68k cells)

## Examples

```
Rscript -e 'devtools::install_github("fabotao/GapClust"); t0<-Sys.time(); gc.out<-GapClust::GapClust(expr); rt<-Sys.time()-t0; mem<-sum(gc()[,""])'
```

## Evaluation signals

- On the largest ~68,000-cell subsample, GapClust completes in seconds and is approximately three times faster than FiRE
- GapClust peak memory at ~68k cells is approximately 3 GB
- FiRE peak memory at ~68k cells exceeds 20 GB; GiniClust, RaceID and CellSIUS each exceed 78 GB
- Every tool-dataset pair has both a running-time and a peak-memory record, and tables match the structure of Supplementary Tables 3-4
- Plotted curves reproduce the monotonic scaling pattern of running time and peak memory with cell count as in Fig. 4a-b

## Limitations

- Comparative tool versions are not specified in the article, so exact reproductions may vary with software versions
- No parameter settings for the tool runs (e.g. GapClust's K or skewness threshold) are reported, which affects exact runtime/memory values
- RaceID is impractical on large inputs due to unrealistic running time, and GiniClust was reported to fail on very large datasets, so those cells of the benchmark may not complete
- FiRE does not discriminate outliers from minor cell types, so results reflect scalability rather than detection correctness

## Evidence

- [results] Scalability benchmark finding from source card: [evidence span withheld — anchor status: not_found]
- [discussion] Speed comparison with FiRE: "GapClust is currently the fastest rare-cell detection method to analyse such large datasets with less memory resources"
- [intro] Scalability limitations of existing tools: "scRNA-seq datasets containing thousands of cells preclude the use of RaceID due to unrealistic amount of running time"
- [abstract] GapClust design goal of efficiency: "a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets with state-of-the-art speed and memory efficiency"
- [readme] README performance claims: "As to compuation time and memory utilization, GapClust also displays unrivaled speed and memory efficiency, in comparison with other methods."
