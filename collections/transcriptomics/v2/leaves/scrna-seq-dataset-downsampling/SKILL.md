---
name: scrna-seq-dataset-downsampling
description: Use when benchmarking how a single-cell method's CPU time and memory
  scale with dataset size, requiring size-stratified subsets drawn from a large atlas
  such as MOCA (10k to 900k cells).
when_to_use_negative:
- The goal is a biological or quality comparison (e.g. metacell purity, compactness,
  separation, INV) rather than computational cost — downsampling changes cell composition
  and is not a like-for-like biological control.
- The dataset is already small enough that all candidate methods run tractably, so
  size scaling is not the question.
- The input is not single-cell count data (e.g. a precomputed feature table or bulk
  data), where cell-count stratification is not meaningful.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MC2 (tanaylab/metacells)
  role: Metacell construction method benchmarked across subset sizes at graining level
    75; divide-and-conquer variant extends to the largest subsets
  repo: https://github.com/tanaylab/metacells
- name: SuperCell
  role: Metacell construction method benchmarked across subset sizes at graining level
    75, including the projection-based variant
  repo: https://github.com/GfellerLab/SuperCell
- name: SEACells
  role: Metacell construction method run only on the subset sizes tractable for it
    during the scaling benchmark
  repo: https://github.com/dpeerlab/SEACells
- name: MetacellAnalysisToolkit
  role: Integrated pipeline environment providing MC2, SuperCell, and SEACells with
    a joined R package for metacell QC used in the benchmark runs
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: Seurat
  role: R framework for loading and preparing single-cell count matrices for the size-stratified
    subsets
- name: ggplot2
  role: Plots CPU time and peak memory versus number of cells for each method and
    variant (fig5c_scaling_curves.png)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:cb1488c39dae0ea7a919c51264c9a6cc82b0ab21f2198dfbf6ac35eba9a5d64e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tanaylab/metacells
  tools:
  - MC2 (tanaylab/metacells)
  - MetacellAnalysisToolkit
  - SEACells
  - Seurat
  - SuperCell
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# scrna-seq-dataset-downsampling

## Summary

Prepare size-stratified single-cell RNA-seq subsets (e.g. MOCA 10k–900k cells) so that the computational cost of metacell construction tools (MC2, SuperCell, SEACells) at a fixed graining level can be benchmarked for CPU time and peak memory scaling.

## When to use

Use when benchmarking how a single-cell method's CPU time and memory scale with dataset size, requiring size-stratified subsets drawn from a large atlas such as MOCA (10k to 900k cells).

## When NOT to use

- The goal is a biological or quality comparison (e.g. metacell purity, compactness, separation, INV) rather than computational cost — downsampling changes cell composition and is not a like-for-like biological control.
- The dataset is already small enough that all candidate methods run tractably, so size scaling is not the question.
- The input is not single-cell count data (e.g. a precomputed feature table or bulk data), where cell-count stratification is not meaningful.

## Inputs

- scRNA-seq count matrices from a large atlas (MOCA mouse organogenesis, GEO accession GSE114953)
- Size-stratification grid spanning dataset sizes (10,000 to 900,000 cells)
- Metacell construction methods and variants to benchmark (MC2, divide-and-conquer MC2, SuperCell, projection SuperCell, SEACells) at graining level 75

## Outputs

- Size-stratified subset count matrices (10k to 900k cells)
- scaling_benchmark.csv with elapsed CPU time and peak memory per method × variant × dataset size
- fig5c_scaling_curves.png — ggplot2 CPU-time and peak-memory curves versus number of cells for each method and variant

## How to apply

Download the raw scRNA-seq count matrices (for the metacell scaling benchmark, the MOCA mouse organogenesis atlas from GEO accession GSE114953, Cao et al 2019) and prepare size-stratified subsets spanning the target range, e.g. 10,000 to 900,000 cells, keeping cell composition representative of the full atlas. Run the target method on every subset size at a fixed parameterization — here graining level 75 for MC2 (tanaylab/metacells) and SuperCell (GfellerLab/SuperCell), and SEACells only on the subset sizes tractable for that method — including scalable variants such as the divide-and-conquer MC2 variant and the projection-based SuperCell variant on the largest subsets. Instrument every construction run to record elapsed CPU time and peak memory per method, variant, and dataset size, including a per-sample parallel MOCA run. Assemble the measurements into a tidy table (scaling_benchmark.csv) with one row per method × variant × dataset-size combination, then plot CPU time and peak memory versus number of cells per method and variant with ggplot2.

## Related tools

- **MC2 (tanaylab/metacells)** (Metacell construction method benchmarked across subset sizes at graining level 75; divide-and-conquer variant extends to the largest subsets) — https://github.com/tanaylab/metacells
- **SuperCell** (Metacell construction method benchmarked across subset sizes at graining level 75, including the projection-based variant) — https://github.com/GfellerLab/SuperCell
- **SEACells** (Metacell construction method run only on the subset sizes tractable for it during the scaling benchmark) — https://github.com/dpeerlab/SEACells
- **MetacellAnalysisToolkit** (Integrated pipeline environment providing MC2, SuperCell, and SEACells with a joined R package for metacell QC used in the benchmark runs) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **Seurat** (R framework for loading and preparing single-cell count matrices for the size-stratified subsets)
- **ggplot2** (Plots CPU time and peak memory versus number of cells for each method and variant (fig5c_scaling_curves.png))

## Evaluation signals

- scaling_benchmark.csv covers every method × variant × dataset-size combination defined for the benchmark
- Plotted CPU-time and memory curves reproduce the expected Fig 5C scaling behavior, e.g. divide-and-conquer MC2 and projection SuperCell extend to the 900k-cell dataset while the naive variants do not
- Peak memory and CPU time increase monotonically with the number of cells for the non-divide-and-conquer methods
- All methods complete on the smallest (10k-cell) subsets, and only the scalable variants (divide-and-conquer MC2, projection SuperCell) complete on the 900k-cell subset
- The per-sample parallel MOCA run is recorded as a separate measurement in the benchmark table

## Limitations

- Except for the divide-and-conquer MC2 method, building metacells is time- and memory-consuming on large datasets, so naive MC2, SuperCell, and SEACells runs cannot cover the full 10k–900k range
- SEACells is only run on the subset sizes tractable for that method, leaving gaps in its scaling curve at large sizes
- Graining level is fixed at 75; scaling results at other graining levels are not covered by this benchmark

## Evidence

- [methods] MOCA subsets and GEO source: [evidence span withheld — anchor status: not_found]
- [results] Benchmark finding: [evidence span withheld — anchor status: not_found]
- [methods] Plotting output: [evidence span withheld — anchor status: not_found]
- [readme] Tool scalability: [evidence span withheld — anchor status: external_doc]
- [abstract] Toolkit pipeline: "a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods"
