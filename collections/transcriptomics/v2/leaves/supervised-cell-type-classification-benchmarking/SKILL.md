---
name: supervised-cell-type-classification-benchmarking
description: Use when you need to compare classification speed or accuracy of cell
  type identifiers (e.g. SciBet vs scmap vs Seurat v3) across scRNA-seq datasets of
  increasing size, or to reproduce the Fig. 2e timing benchmark.
when_to_use_negative:
- The goal is clustering or unsupervised annotation — the benchmark is specifically
  for supervised classifiers given annotated reference labels
- The task is model accuracy on real annotated atlases (e.g. Tabula Muris, MOCA) rather
  than runtime scaling on simulated count matrices
- Data require cross-species mapping (human-mouse HomoloGene conversion) or null-dataset
  false-positive control, which are separate SciBet capabilities not exercised here
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: SciBet
  role: Primary classifier under benchmark; SelectGene picks 500 informative genes
    by ΔS, Learn/Bet classify via multinomial MLE
  repo: https://github.com/PaulingLiu/scibet
- name: scmap
  role: Baseline classifier run via scmapCluster without considering unassigned cells,
    timed under identical single-CPU conditions
- name: Seurat v3
  role: Baseline classifier using FindTransferAnchors followed by TransferData, timed
    under identical conditions
- name: ggplot2
  role: Plots runtimes of the three methods versus dataset size on a logarithmic scale
    for the Fig. 2e comparison
- name: tidyverse
  role: Dependency package for data aggregation of per-dataset runtimes into the benchmark
    CSV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:3fb201bea8c84bb58446cddad3dd14e69804eaf6f11b623ebd4feb8ee1ce4233
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - SciBet
  - Seurat v3
  - ggplot2
  - scmap
  - tidyverse
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# supervised-cell-type-classification-benchmarking

## Summary

Benchmarks supervised single-cell type classifiers (SciBet, scmap, Seurat v3) for classification speed and accuracy across dataset sizes, from 1,000 to 50,000 simulated cells, using the multinomial-MLE SciBet model as the reference method.

## When to use

Use when you need to compare classification speed or accuracy of cell type identifiers (e.g. SciBet vs scmap vs Seurat v3) across scRNA-seq datasets of increasing size, or to reproduce the Fig. 2e timing benchmark.

## When NOT to use

- The goal is clustering or unsupervised annotation — the benchmark is specifically for supervised classifiers given annotated reference labels
- The task is model accuracy on real annotated atlases (e.g. Tabula Muris, MOCA) rather than runtime scaling on simulated count matrices
- Data require cross-species mapping (human-mouse HomoloGene conversion) or null-dataset false-positive control, which are separate SciBet capabilities not exercised here

## Inputs

- Simulated scRNA-seq count matrices with 500 genes and 1,000 / 10,000 / 20,000 / 50,000 cells
- Annotated reference training set for supervised classification
- SciBet R package installed from github.com/PaulingLiu/scibet
- scmap and Seurat v3 R packages

## Outputs

- scibet_timing_benchmark.csv with per-dataset runtimes and derived throughput (cells/second) for SciBet, scmap, and Seurat v3
- fig2e_timing_comparison.png — log-scale runtime vs dataset size plot (ggplot2) reproducing Fig. 2e
- Per-method classification predictions on each test dataset

## How to apply

Generate simulated scRNA-seq count matrices (500 genes) with 1,000, 10,000, 20,000, and 50,000 cells using the scripts in ./scripts of the PaulingLiu/scibet repository. Install SciBet with devtools::install_github('PaulingLiu/scibet'), select 500 informative genes via SelectGene (top ΔS statistic), then classify with Learn/Bet on a single CPU and record elapsed wall-clock time. Run scmapCluster (without considering unassigned cells) and Seurat v3 FindTransferAnchors followed by TransferData under identical single-CPU conditions. Aggregate per-dataset runtimes into a CSV with derived throughput (cells/second) and plot runtimes versus dataset size on a log scale with ggplot2; validate that SciBet reaches ~100,000 cells/second and is orders of magnitude faster than both competitors.

## Related tools

- **SciBet** (Primary classifier under benchmark; SelectGene picks 500 informative genes by ΔS, Learn/Bet classify via multinomial MLE) — https://github.com/PaulingLiu/scibet
- **scmap** (Baseline classifier run via scmapCluster without considering unassigned cells, timed under identical single-CPU conditions)
- **Seurat v3** (Baseline classifier using FindTransferAnchors followed by TransferData, timed under identical conditions)
- **ggplot2** (Plots runtimes of the three methods versus dataset size on a logarithmic scale for the Fig. 2e comparison)
- **tidyverse** (Dependency package for data aggregation of per-dataset runtimes into the benchmark CSV)

## Examples

```
devtools::install_github("PaulingLiu/scibet"); genes <- SelectGene(ref, k=500); model <- Learn(ref, genes); pred <- Bet(query, model)
```

## Evaluation signals

- SciBet attains a throughput of approximately 100,000 cells per second on the simulated datasets
- SciBet runtime is orders of magnitude lower than scmapCluster and Seurat v3 TransferData across all four dataset sizes (1,000–50,000 cells)
- Benchmark CSV contains runtime and throughput rows for all three methods at each of the four simulated sizes
- Timing plot reproduces the log-scale Fig. 2e relationship between runtime and dataset size
- All methods were run under the same single-CPU wall-clock timing conditions

## Limitations

- Seurat v3 minor version, scmap version, and random seeds for the benchmark splits are not reported in the article
- Benchmark relies on simulated count matrices (500 genes) as generated by the repository scripts; runtime scaling on real datasets may differ
- scmap was evaluated without considering unassigned cells, so its effective runtime excludes rejection handling

## Evidence

- [results] SciBet outperformed scmap and Seurat v3 by orders of magnitude in classification speed: [evidence span withheld — anchor status: not_found]
- [discussion] Seurat v3 anchors-based baseline: "For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions"
- [discussion] scmapCluster baseline without unassigned cells: "we performed the classification using the function scmapCluster of scmap without considering the unassigned cells"
- [discussion] Feature selection of 500 informative genes by ΔS: "we selected genes with top ΔS as informative genes (500 by default)"
- [readme] SciBet installation via devtools: "devtools::install_github("PaulingLiu/scibet")"
