---
name: scrna-seq-dataset-subsampling
description: Use when benchmarking rare-cell or doublet detection methods on scRNA-seq
  data and controlled, labeled datasets with known rare-cell abundances are needed
  as inputs.
when_to_use_negative:
- The goal is applying GapClust to a real, unmodified dataset to discover rare cells
  rather than evaluating methods on controlled benchmarks
- The input dataset lacks labeled cell populations (e.g., CD14+ monocytes, NK cells,
  B cells) needed to plant and score rare cells against ground truth
- A homogeneous background population is not appropriate — e.g., when realistic multi-cell-type
  composition is required for the benchmark
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GapClust
  role: Benchmarked rare-cell detection method; identifies candidate k where skewness
    of the distance-score distribution exceeds 2
  repo: github.com/fabotao/GapClust
- name: RaceID
  role: Comparator run on every subsampled dataset with published default settings
- name: GiniClust
  role: Comparator run on every subsampled dataset with published default settings
- name: CellSIUS
  role: Comparator run on every subsampled dataset with published default settings
- name: FiRE
  role: 'Comparator run on every subsampled dataset twice: once with default threshold
    and once with relaxed IQR threshold'
- name: Seurat
  role: Prerequisite R package (>= 3.1.0) for the GapClust environment used on the
    benchmarks
  repo: github.com/fabotao/GapClust
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:573aed4e3618fdca6d13f531afa7fc45e9e503c46c272a5dbc36908d1f5d45ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellSIUS
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# scrna-seq-dataset-subsampling

## Summary

Construct controlled single-cell benchmark datasets by subsampling the 68k PBMC dataset, planting known numbers of CD14+ monocytes into homogeneous NK/B-cell populations to create rare-cell and doublet detection benchmarks with ground-truth labels.

## When to use

Use when benchmarking rare-cell or doublet detection methods on scRNA-seq data and controlled, labeled datasets with known rare-cell abundances are needed as inputs.

## When NOT to use

- The goal is applying GapClust to a real, unmodified dataset to discover rare cells rather than evaluating methods on controlled benchmarks
- The input dataset lacks labeled cell populations (e.g., CD14+ monocytes, NK cells, B cells) needed to plant and score rare cells against ground truth
- A homogeneous background population is not appropriate — e.g., when realistic multi-cell-type composition is required for the benchmark

## Inputs

- 68k PBMC scRNA-seq dataset (raw expression profiles with cell-type labels for CD14+ monocytes, NK cells, B cells)
- Rare-cell abundance gradient specification (2-100 planted CD14+ monocytes)
- Doublet mixing-proportion specification (0.08% to 5%)

## Outputs

- 99 rare-cell benchmark datasets with ground-truth labels
- 140 doublet benchmark datasets with ground-truth labels
- Precision, recall, and F1 scores per method aggregated across the abundance gradient
- Results table written to benchmark_f1_results.csv

## How to apply

Subsample the 68k PBMC dataset to construct 99 rare-cell benchmark datasets by embedding 2-100 CD14+ monocytes into a homogeneous population of 500 NK cells plus 500 B cells, creating a controlled abundance gradient. Separately construct 140 doublet benchmark datasets by embedding 2 monocytes at mixing proportions from 0.08% to 5%. The planted-cell identities serve as ground-truth labels so each candidate method's rare-cell calls can be scored with precision, recall, and F1. Run the comparator methods on every dataset — RaceID, GiniClust, and CellSIUS with published default settings; FiRE twice, once with the default threshold and once with a relaxed IQR threshold; and GapClust, identifying candidate k where the skewness of the distance-score distribution exceeds 2. Aggregate F1 per method across the abundance gradient and per threshold setting to yield a fair benchmark under known rare-cell frequency.

## Related tools

- **GapClust** (Benchmarked rare-cell detection method; identifies candidate k where skewness of the distance-score distribution exceeds 2) — github.com/fabotao/GapClust
- **RaceID** (Comparator run on every subsampled dataset with published default settings)
- **GiniClust** (Comparator run on every subsampled dataset with published default settings)
- **CellSIUS** (Comparator run on every subsampled dataset with published default settings)
- **FiRE** (Comparator run on every subsampled dataset twice: once with default threshold and once with relaxed IQR threshold)
- **Seurat** (Prerequisite R package (>= 3.1.0) for the GapClust environment used on the benchmarks) — github.com/fabotao/GapClust

## Evaluation signals

- Each of the 99 rare-cell datasets contains exactly the specified number of planted CD14+ monocytes (2-100) within 500 NK plus 500 B cells
- Each of the 140 doublet datasets embeds 2 monocytes at a mixing proportion within 0.08%-5%
- Planted-cell labels match each method's rare-cell calls exactly when computing precision, recall, and F1
- F1 results reproduce Fig. 2b and Supplementary Figs. 5 and 8 of the GapClust paper
- Aggregated F1 results table is written to benchmark_f1_results.csv with one row per method per threshold setting

## Limitations

- The abundance gradient covers only CD14+ monocytes embedded in an NK/B-cell background; other rare-cell types or background compositions are untested
- FiRE requires evaluation under both default and relaxed IQR thresholds, adding a threshold sensitivity dimension to the benchmark
- No version numbers for the compared tools (GiniClust, RaceID, CellSIUS, FiRE, EDGE) are reported, complicating exact reproduction

## Evidence

- [other] Subsample the 68k PBMC dataset to construct 99 rare-cell benchmark datasets, embedding 2-100 CD14+ monocytes into 500 NK cells plus 500 B cells.: [evidence span withheld — anchor status: not_found]
- [discussion] 99 subsampled datasets from the 68 k PBMC dataset, ten subsampled datasets from the 293T-Jurkat dataset and 140 subsampled datasets: "99 subsampled datasets from the 68 k PBMC dataset, ten subsampled datasets from the 293T-Jurkat dataset and 140 subsampled datasets"
- [other] Construct 140 doublet benchmark datasets by embedding 2 monocytes at mixing proportions from 0.08% to 5%.: [evidence span withheld — anchor status: not_found]
- [readme] In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F1 score.: "In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F"
