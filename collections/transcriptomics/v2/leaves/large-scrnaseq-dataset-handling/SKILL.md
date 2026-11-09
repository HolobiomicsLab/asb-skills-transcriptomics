---
name: large-scrnaseq-dataset-handling
description: Use when input is a large single-cell expression matrix (e.g. 1,000 to
  ~68,000 cells, such as the 68k PBMC dataset) and the goal is rare cell detection
  under tight runtime/memory budgets, or a scaling benchmark of time and peak RAM
  across rare-cell tools.
when_to_use_negative:
- Input is a very small dataset (hundreds of cells) where heavier tools like RaceID
  or CellSIUS are still computationally feasible and may be preferred for their accuracy
  trade-offs
- The goal is comprehensive major cell-type characterisation — use toolkits such as
  Seurat or Scater instead, since GapClust is specialised for rare cell detection
- CellSIUS-specific workflows are required that presuppose major cell types already
  determined
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GapClust
  role: Primary light-weight tool for rare cell detection on ultra-large datasets
    with state-of-the-art speed and memory efficiency
  repo: github.com/fabotao/GapClust
- name: FiRE
  role: Benchmarked competitor; only feasible alternative at ~68k cells besides GapClust
    (>20 GB peak memory, ~3x slower than GapClust)
- name: RaceID
  role: Benchmarked competitor; computationally precluded on thousands-of-cell datasets
    due to unrealistic running time (>78 GB at 68k cells)
- name: GiniClust
  role: Benchmarked competitor; reported to fail on very large datasets (>78 GB peak
    memory at 68k cells)
- name: CellSIUS
  role: Benchmarked competitor; requires major cell types to be determined and exceeds
    78 GB peak memory at 68k cells
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:787d8e03662641e0482c68b3865866c9dc14578fc4969c113158025a90fa794a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellSIUS
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# large-scrnaseq-dataset-handling

## Summary

A skill for detecting rare cell types in ultra-large scRNA-seq datasets (thousands to ~68,000 cells) while keeping runtime and peak memory tractable, and for benchmarking that efficiency against RaceID, GiniClust, CellSIUS and FiRE. It is grounded in GapClust, a light-weight R tool that exploits the inter-cluster gap via a second-order derivative type distance score instead of expensive iterative clustering.

## When to use

Use when input is a large single-cell expression matrix (e.g. 1,000 to ~68,000 cells, such as the 68k PBMC dataset) and the goal is rare cell detection under tight runtime/memory budgets, or a scaling benchmark of time and peak RAM across rare-cell tools.

## When NOT to use

- Input is a very small dataset (hundreds of cells) where heavier tools like RaceID or CellSIUS are still computationally feasible and may be preferred for their accuracy trade-offs
- The goal is comprehensive major cell-type characterisation — use toolkits such as Seurat or Scater instead, since GapClust is specialised for rare cell detection
- CellSIUS-specific workflows are required that presuppose major cell types already determined

## Inputs

- Single-cell RNA-seq expression profile (count matrix) with up to ~68,000 cells, e.g. the 68k PBMC dataset
- Subsampled versions of large datasets (e.g. 16 subsamples spanning ~1,000 to ~68,000 cells) for scalability benchmarking
- R environment (R >= 3.4.0) with devtools

## Outputs

- Rare cell type assignments / detected minor-cluster cells
- Per-tool wall-clock running time and peak memory (peak RAM) measurements per dataset (Supplementary Tables 3-4 equivalents)
- Scaling plots of running time vs. number of cells and peak memory vs. number of cells (Fig. 4a-b equivalents)

## How to apply

Install GapClust from GitHub with devtools (R >= 3.4.0, with Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1) and run it on the expression profile; GapClust skips rare-informative-gene search and instead learns cluster size and rare cells arithmetically using the gap between a minor cluster Cn of size n and its neighbouring abundant cluster. Candidate k is identified when the skewness score exceeds a threshold (skewness<k> > 2, per the article's rule of thumb). For scalability benchmarking, run each tool (RaceID, GiniClust, CellSIUS, FiRE, GapClust) on a series of subsampled datasets ranging from ~1,000 to ~68,000 cells, recording wall-clock time and peak RAM per tool-dataset pair, then compile Supplementary-Table-style tables and plot time and memory versus cell number. Decision point: if the input is tens of thousands of cells, RaceID is impractical (unrealistic running time) and GiniClust was reported to fail, so rely on FiRE or GapClust, preferring GapClust for memory. Validate by checking that on the ~68k-cell subsample GapClust uses ~3 GB of peak memory, runs about three times faster than FiRE, and the heavier tools (GiniClust, RaceID, CellSIUS) exceed 20-78 GB.

## Related tools

- **GapClust** (Primary light-weight tool for rare cell detection on ultra-large datasets with state-of-the-art speed and memory efficiency) — github.com/fabotao/GapClust
- **FiRE** (Benchmarked competitor; only feasible alternative at ~68k cells besides GapClust (>20 GB peak memory, ~3x slower than GapClust))
- **RaceID** (Benchmarked competitor; computationally precluded on thousands-of-cell datasets due to unrealistic running time (>78 GB at 68k cells))
- **GiniClust** (Benchmarked competitor; reported to fail on very large datasets (>78 GB peak memory at 68k cells))
- **CellSIUS** (Benchmarked competitor; requires major cell types to be determined and exceeds 78 GB peak memory at 68k cells)

## Examples

```
install.packages("devtools"); devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- On the largest ~68,000-cell subsample of the 68k PBMC dataset, GapClust's peak memory should be ~3 GB, not >20 GB
- GapClust's wall-clock runtime on the largest dataset should be in seconds and about three times faster than FiRE
- GiniClust, RaceID and CellSIUS runs on the ~68k-cell dataset should show peak memory exceeding 20 GB (reported >78 GB), confirming the scaling trend
- Reproduced time-vs-cells and memory-vs-cells plots should match the monotonic scaling patterns of Fig. 4a-b and Supplementary Tables 3-4
- GapClust installation succeeds with the stated prerequisites: R >= 3.4.0, Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1

## Limitations

- No version numbers are given for the compared tools (GiniClust, RaceID, CellSIUS, FiRE, EDGE), so exact reproducibility of the benchmark numbers is uncertain
- No parameter settings for GapClust runs (e.g. K, skewness threshold) are reported in the results/discussion sections, requiring users to rely on the skewness<k> > 2 heuristic
- FiRE does not discriminate outliers from minor cell types, so its calls need downstream interpretation; results for many real datasets are deferred to unlinked supplementary figures (Supplementary Figs. 14-22, Supplementary Table 5)

## Evidence

- [abstract] GapClust as light-weight algorithm with speed and memory efficiency: "Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets"
- [discussion] GapClust fastest with least memory on large datasets: "GapClust is currently the fastest rare-cell detection method to analyse such large datasets with less memory resources"
- [intro] RaceID infeasible at scale, motivating light-weight handling: "In growing cases, scRNA-seq datasets containing thousands of cells preclude the use of RaceID due to unrealistic amount of running time"
- [readme] README states GapClust's runtime/memory advantage over other methods: "As to compuation time and memory utilization, GapClust also displays unrivaled speed and memory efficiency, in comparison with other methods."
