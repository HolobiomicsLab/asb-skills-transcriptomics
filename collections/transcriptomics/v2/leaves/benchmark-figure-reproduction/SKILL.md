---
name: benchmark-figure-reproduction
description: Use when the goal is to reproduce or validate published rare-cell detection
  benchmarks on the 68k PBMC subsampled rare-cell (99) and doublet (140) datasets,
  or to re-derive F1, runtime, and peak-memory comparisons across RaceID, GiniClust,
  CellSIUS, FiRE and GapClust.
when_to_use_negative:
- The goal is biological discovery of new rare cell types in a novel dataset rather
  than validating the published benchmark comparisons — use GapClust directly instead
  of the reproduction workflow.
- The target publication reports only qualitative, unlinked supplementary results
  (e.g. Supplementary Figs. 14-22) with no underlying tables available in the provided
  material — a faithful reproduction cannot be validated.
- The dataset lacks ground-truth labels for planted rare cells or doublets, so precision/recall/F1
  against truth cannot be computed.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GapClust
  role: Primary rare-cell detection method being benchmarked; candidate k identified
    where skewness of the distance-score distribution exceeds 2
  repo: github.com/fabotao/GapClust
- name: RaceID
  role: Baseline method run on every dataset with published default settings for F1
    comparison
- name: GiniClust
  role: Baseline method run on every dataset with published default settings for F1
    comparison
- name: CellSIUS
  role: Baseline method run on every dataset with published default settings for F1
    comparison
- name: FiRE
  role: Baseline method run twice per dataset — default threshold and relaxed IQR
    threshold — for F1 comparison
- name: Seurat
  role: Prerequisite R package (>= 3.1.0) for the GapClust installation used in the
    benchmark runs
  repo: github.com/fabotao/GapClust
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:835fd54644a3068764a8a6718e100d9a6a51a47c09959c82663e08640fd3ce43
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Augur
  - CellSIUS
  - FiRE
  - GapClust
  - GiniClust
  - R
  - RaceID
  - Seurat
  - scRank
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# benchmark-figure-reproduction

## Summary

A reproducibility workflow for re-running rare-cell detection benchmarks from the GapClust study (Nat Commun 2021), re-executing GapClust, RaceID, GiniClust, CellSIUS and FiRE on the 68k PBMC-derived subsampled datasets and regenerating the paper's F1-score and runtime/memory figures and tables.

## When to use

Use when the goal is to reproduce or validate published rare-cell detection benchmarks on the 68k PBMC subsampled rare-cell (99) and doublet (140) datasets, or to re-derive F1, runtime, and peak-memory comparisons across RaceID, GiniClust, CellSIUS, FiRE and GapClust.

## When NOT to use

- The goal is biological discovery of new rare cell types in a novel dataset rather than validating the published benchmark comparisons — use GapClust directly instead of the reproduction workflow.
- The target publication reports only qualitative, unlinked supplementary results (e.g. Supplementary Figs. 14-22) with no underlying tables available in the provided material — a faithful reproduction cannot be validated.
- The dataset lacks ground-truth labels for planted rare cells or doublets, so precision/recall/F1 against truth cannot be computed.

## Inputs

- 68k PBMC scRNA-seq expression matrix (single-cell count data)
- 99 rare-cell benchmark subsample datasets (2-100 CD14+ monocytes embedded in 500 NK + 500 B cells)
- 140 doublet benchmark subsample datasets (2 monocytes at 0.08%-5% mixing proportions)
- 16 scalability benchmark datasets ranging from ~1,000 to ~68,000 cells
- Ground-truth cell-type labels for planted rare cells and doublets

## Outputs

- benchmark_f1_results.csv with precision, recall and F1 per method per dataset/threshold
- F1-score plots reproducing Fig. 2b and Supplementary Figs. 5 and 8
- Running-time and peak-memory tables equivalent to Supplementary Tables 3-4
- Runtime-vs-cells and memory-vs-cells plots reproducing Fig. 4a-b

## How to apply

Subsample the 68k PBMC dataset to build 99 rare-cell benchmarks (2-100 CD14+ monocytes planted into 500 NK + 500 B cells) and 140 doublet benchmarks (2 monocytes at mixing proportions from 0.08% to 5%). Run GapClust on every dataset, identifying candidate cluster size k where the skewness of the distance-score distribution exceeds 2; run RaceID, GiniClust and CellSIUS with their published defaults; run FiRE twice, once with the default threshold and once with a relaxed IQR threshold. Score each method's calls against ground-truth labels using precision, recall and F1, aggregating per method across the abundance gradient and per threshold setting, then verify the plotted F1 results reproduce Fig. 2b and Supplementary Figs. 5 and 8. Separately run all five tools on 16 subsampled datasets from ~1,000 to ~68,000 cells, recording wall-clock time and peak RAM into tables equivalent to Supplementary Tables 3-4, and check the scaling patterns against Fig. 4a-b. Write results to benchmark_f1_results.csv so numeric claims can be diffed against the reported benchmarks.

## Related tools

- **GapClust** (Primary rare-cell detection method being benchmarked; candidate k identified where skewness of the distance-score distribution exceeds 2) — github.com/fabotao/GapClust
- **RaceID** (Baseline method run on every dataset with published default settings for F1 comparison)
- **GiniClust** (Baseline method run on every dataset with published default settings for F1 comparison)
- **CellSIUS** (Baseline method run on every dataset with published default settings for F1 comparison)
- **FiRE** (Baseline method run twice per dataset — default threshold and relaxed IQR threshold — for F1 comparison)
- **Seurat** (Prerequisite R package (>= 3.1.0) for the GapClust installation used in the benchmark runs) — github.com/fabotao/GapClust

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- Reproduced F1 plot matches Fig. 2b and Supplementary Figs. 5 and 8, with GapClust achieving higher F1 than GiniClust, RaceID, CellSIUS and FiRE (including FiRE with relaxed IQR thresholds) across the 2-100 monocyte abundance gradient
- On the largest ~68,000-cell subsample, GapClust finishes in seconds and is about 3x faster than FiRE
- GapClust peak memory at ~68k cells is ~3 GB, versus >20 GB for FiRE and over 78 GB for GiniClust, RaceID and CellSIUS
- benchmark_f1_results.csv exists with per-method precision, recall and F1 rows for all 99 rare-cell and 140 doublet datasets
- GapClust candidate-k step fires only where skewness of the distance-score distribution exceeds 2, consistent with the article's stated criterion

## Limitations

- No version numbers are given for the compared tools (GiniClust, RaceID, CellSIUS, FiRE, EDGE), so exact-version reproducibility is not guaranteed
- No parameter settings for GapClust runs (e.g. K, skewness threshold) are reported in all sections, requiring inference from the article text
- Results for most real datasets are deferred to unlinked supplementary figures (Supplementary Figs. 14-22, Supplementary Table 5) not included in the article text
- GiniClust, RaceID and CellSIUS need >78 GB peak memory at ~68k cells, so reproduction of the scalability benchmark requires very large RAM

## Evidence

- [other] GapClust achieved higher F1 scores than GiniClust, RaceID, CellSIUS and FiRE, including FiRE evaluated with relaxed IQR thresholds.: [evidence span withheld — anchor status: not_found]
- [abstract] Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets with state-of-the-art speed and memory efficiency: "Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets"
- [discussion] 99 subsampled datasets from the 68 k PBMC dataset, ten subsampled datasets from the 293T-Jurkat dataset and 140 subsampled datasets from the 68 k PBMC dataset: "99 subsampled datasets from the 68 k PBMC dataset, ten subsampled datasets from the 293T-Jurkat dataset and 140 subsampled datasets from the 68 k PBMC dataset"
- [other] GapClust running about three times faster than FiRE, and GapClust required only ~3 GB of memory versus >20 GB for FiRE and over 78 GB for GiniClust, RaceID and CellSIUS.: [evidence span withheld — anchor status: not_found]
- [readme] R users  can easily install GapClust by running following code in R console. devtools::install_github("fabotao/GapClust"): "devtools::install_github("fabotao/GapClust")"
