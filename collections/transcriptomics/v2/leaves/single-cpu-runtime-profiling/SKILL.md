---
name: single-cpu-runtime-profiling
description: Use when a benchmark requires measuring and comparing classification/annotation
  runtime of single-cell label-transfer tools as dataset size scales, and elapsed
  wall-clock time and throughput are the primary metrics.
when_to_use_negative:
- The question under test is prediction accuracy or false positive rate rather than
  runtime — use the stratified 7:3 hold-out validation skill instead (50 repeats)
  rather than timing runs
- Input datasets are real experimental datasets with annotation labels rather than
  the simulated scaling matrices; timing comparisons across heterogeneous references
  confound dataset difficulty with speed
- Compute resources are multi-core or GPU-based and cannot be pinned to a single CPU,
  breaking the like-for-like runtime comparison
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SciBet
  role: Classifier under test; SelectGene/Learn/Bet pipeline timed per simulated dataset
  repo: github.com/PaulingLiu/scibet
- name: scmap
  role: Baseline classifier timed via scmapCluster without considering unassigned
    cells
- name: Seurat v3
  role: Baseline classifier timed via FindTransferAnchors followed by TransferData
- name: ggplot2
  role: Plots runtimes vs. dataset size on a logarithmic scale to reproduce Fig. 2e
- name: tidyverse
  role: Aggregates per-dataset runtimes into scibet_timing_benchmark.csv with derived
    throughput
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:5a3d4c86d48f52b00619720ea48fed818aa2fec3a026e07571cf7df944bac7af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciBet
  - Seurat v3
  - ggplot2
  - scmap
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cpu-runtime-profiling

## Summary

Benchmarks the wall-clock classification runtime and throughput (cells per second) of single-cell annotation methods — SciBet vs. scmap and Seurat v3 — on simulated scRNA-seq datasets of increasing size (1,000 to 50,000 cells), under fixed single-CPU conditions, to demonstrate SciBet's order-of-magnitude speed advantage (~100,000 cells per second).

## When to use

Use when a benchmark requires measuring and comparing classification/annotation runtime of single-cell label-transfer tools as dataset size scales, and elapsed wall-clock time and throughput are the primary metrics.

## When NOT to use

- The question under test is prediction accuracy or false positive rate rather than runtime — use the stratified 7:3 hold-out validation skill instead (50 repeats) rather than timing runs
- Input datasets are real experimental datasets with annotation labels rather than the simulated scaling matrices; timing comparisons across heterogeneous references confound dataset difficulty with speed
- Compute resources are multi-core or GPU-based and cannot be pinned to a single CPU, breaking the like-for-like runtime comparison

## Inputs

- Simulated scRNA-seq count matrices with 500 genes and 1,000 / 10,000 / 20,000 / 50,000 cells (from the PaulingLiu/scibet ./scripts simulation scripts)
- Single-CPU compute environment common to all compared methods
- Installed R packages: SciBet (devtools::install_github), scmap, Seurat v3, tidyverse, ggplot2

## Outputs

- scibet_timing_benchmark.csv with per-dataset runtimes for SciBet, scmap, and Seurat v3 plus derived throughput (cells/second)
- fig2e_timing_comparison.png — runtime vs. dataset size plotted on a logarithmic scale with ggplot2

## How to apply

Generate scRNA-seq count matrices (500 genes) of 1,000, 10,000, 20,000, and 50,000 cells using the simulation scripts shipped in the ./scripts directory of the PaulingLiu/scibet repository. Run each classifier on a single CPU under identical conditions: SciBet via SelectGene (top 500 informative genes by ΔS) then Learn/Bet using the multinomial MLE model installed with devtools::install_github("PaulingLiu/scibet"); scmap via scmapCluster ignoring unassigned cells; Seurat v3 via FindTransferAnchors followed by TransferData. Record elapsed wall-clock time per dataset for each method, then derive throughput in cells per second. The rationale is to expose algorithmic scaling differences that accuracy benchmarks hide — SciBet's multinomial likelihood over a compact marker set avoids the expensive anchor/search procedures of competitors.

## Related tools

- **SciBet** (Classifier under test; SelectGene/Learn/Bet pipeline timed per simulated dataset) — github.com/PaulingLiu/scibet
- **scmap** (Baseline classifier timed via scmapCluster without considering unassigned cells)
- **Seurat v3** (Baseline classifier timed via FindTransferAnchors followed by TransferData)
- **ggplot2** (Plots runtimes vs. dataset size on a logarithmic scale to reproduce Fig. 2e)
- **tidyverse** (Aggregates per-dataset runtimes into scibet_timing_benchmark.csv with derived throughput)

## Examples

```
devtools::install_github("PaulingLiu/scibet"); genes <- SelectGene(ref, k = 500); model <- Learn(ref, genes, k = 1); Bet(query, model)
```

## Evaluation signals

- SciBet attains a throughput of approximately 100,000 cells per second on the simulated datasets
- SciBet's runtime is orders of magnitude lower than scmap and Seurat v3 across all four simulated dataset sizes (1,000–50,000 cells)
- scibet_timing_benchmark.csv contains runtime entries for all three methods at all four dataset sizes, each with a non-negative derived throughput value
- All three methods were run under identical single-CPU conditions so the wall-clock times are directly comparable
- The ggplot2 figure reproduces the log-scale Fig. 2e timing comparison showing runtime versus dataset size

## Limitations

- Timings reflect the authors' single-CPU conditions and simulation settings (500 genes); throughput on other hardware or real data will differ
- scmap was run without considering unassigned cells, which removes a rejection step that in practice adds runtime
- Exact software versions were not reported for scmap and Seurat v3, so timing comparisons may not reproduce exactly
- Simulation scripts live in the repository's ./scripts directory; if unavailable, the scaling series cannot be regenerated as published

## Evidence

- [abstract] order-of-magnitude speed advantage: "We present SciBet, a supervised cell type identifier that accurately predicts cell identity for newly sequenced cells with order-of-magnitude speed advantage."
- [discussion] scmapCluster without unassigned cells: "we performed the classification using the function scmapCluster of scmap without considering the unassigned cells"
- [discussion] Seurat v3 anchors-based classification: "For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions"
- [methods] installation via devtools: "devtools::install_github("PaulingLiu/scibet")"
- [readme] simulation scripts location: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
