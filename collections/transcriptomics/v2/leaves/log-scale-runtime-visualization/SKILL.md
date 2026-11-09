---
name: log-scale-runtime-visualization
description: Use when comparing algorithm runtimes across dataset sizes that span
  orders of magnitude (e.g. 1,000–50,000 cells) and the speed advantage itself is
  the finding to visualize.
when_to_use_negative:
- The input is accuracy/f1 data rather than wall-clock timing — this skill visualizes
  runtime, not prediction performance
- All runtimes are of the same order of magnitude, so a log axis would add no interpretive
  value and could exaggerate noise
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SciBet
  role: Classifier whose runtime is benchmarked; run via SelectGene then Learn/Bet
  repo: github.com/PaulingLiu/scibet
- name: scmap
  role: Baseline classifier run with scmapCluster (unassigned cells not considered)
    for the timing comparison
- name: Seurat v3
  role: Baseline classifier run with FindTransferAnchors followed by TransferData
    for the timing comparison
- name: ggplot2
  role: R plotting package used to draw the runtime-vs-dataset-size comparison on
    a log scale (Fig. 2e)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:5843addbb02fcacc5d2213a8963d4f1cdcd15c774e7c356ae13a468c118291fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciBet
  - Seurat v3
  - ggplot2
  - scmap
  - tidyverse
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# log-scale-runtime-visualization

## Summary

Reproduces the Fig. 2e-style timing comparison from the SciBet paper by plotting classification runtimes of SciBet vs. scmap and Seurat v3 against dataset size (1,000–50,000 simulated cells) on a logarithmic axis, making order-of-magnitude speed differences visually explicit.

## When to use

Use when comparing algorithm runtimes across dataset sizes that span orders of magnitude (e.g. 1,000–50,000 cells) and the speed advantage itself is the finding to visualize.

## When NOT to use

- The input is accuracy/f1 data rather than wall-clock timing — this skill visualizes runtime, not prediction performance
- All runtimes are of the same order of magnitude, so a log axis would add no interpretive value and could exaggerate noise

## Inputs

- Simulated scRNA-seq count matrices (500 genes; 1,000, 10,000, 20,000, and 50,000 cells)
- Per-dataset wall-clock runtimes for SciBet, scmap (scmapCluster), and Seurat v3 (FindTransferAnchors + TransferData) recorded under single-CPU conditions

## Outputs

- scibet_timing_benchmark.csv with per-method runtimes and derived throughput in cells per second
- fig2e_timing_comparison.png: ggplot2 runtime-vs-dataset-size plot on logarithmic scale

## How to apply

First run each classifier (SciBet via SelectGene on 500 ΔS-ranked informative genes followed by Learn/Bet with the multinomial MLE model installed with devtools::install_github("PaulingLiu/scibet"), scmapCluster without considering unassigned cells, and Seurat v3's FindTransferAnchors + TransferData) under identical single-CPU conditions on simulated scRNA-seq count matrices of 500 genes and 1,000/10,000/20,000/50,000 cells. Record elapsed wall-clock time per dataset per method into scibet_timing_benchmark.csv, including derived throughput in cells per second. Then plot runtime versus dataset size with ggplot2, applying a logarithmic scale to the runtime axis so that SciBet's ~100,000 cells/second throughput and orders-of-magnitude gap versus scmap and Seurat v3 are legible; save the figure as fig2e_timing_comparison.png. Use the simulation scripts in the ./scripts directory of the PaulingLiu/scibet repository to regenerate inputs.

## Related tools

- **SciBet** (Classifier whose runtime is benchmarked; run via SelectGene then Learn/Bet) — github.com/PaulingLiu/scibet
- **scmap** (Baseline classifier run with scmapCluster (unassigned cells not considered) for the timing comparison)
- **Seurat v3** (Baseline classifier run with FindTransferAnchors followed by TransferData for the timing comparison)
- **ggplot2** (R plotting package used to draw the runtime-vs-dataset-size comparison on a log scale (Fig. 2e))

## Examples

```
Rscript -e 'ggplot(timings, aes(x = n_cells, y = runtime, colour = method)) + geom_line() + scale_x_log10() + scale_y_log10() + ggsave("fig2e_timing_comparison.png")'
```

## Evaluation signals

- SciBet's derived throughput is approximately 100,000 cells per second
- SciBet's runtime is orders of magnitude lower than scmap and Seurat v3 at all four simulated dataset sizes (1,000–50,000 cells)
- scibet_timing_benchmark.csv contains runtime and cells-per-second rows for all three methods × four dataset sizes
- The output figure uses a logarithmic scale and matches the qualitative shape of Fig. 2e in the paper
- All methods were timed under the same single-CPU conditions, so cross-method runtime ratios are valid

## Limitations

- Runtimes are hardware-dependent; results are only comparable when all methods are timed on the same single-CPU setup
- The benchmark uses simulated datasets of 500 genes, so absolute runtimes may not transfer to real datasets with different gene counts or preprocessing
- Exact software versions (Seurat v3 minor version, scmap) are not reported in the article, complicating exact reproduction

## Evidence

- [results] SciBet outperformed scmap and Seurat v3 by orders of magnitude in classification speed: [evidence span withheld — anchor status: not_found]
- [other] Plot the runtimes of SciBet, scmap, and Seurat v3 versus dataset size on a logarithmic scale with ggplot2: [evidence span withheld — anchor status: not_found]
- [readme] scripts directory reproduces quantitative results: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
- [methods] 500 informative genes by ΔS: "we selected genes with top ΔS as informative genes (500 by default)"
