---
name: f1-score-evaluation-of-rare-cell-calls
description: Use when benchmarking rare-cell or doublet detection on subsampled scRNA-seq
  data with known planted cells, to compare methods like GapClust, RaceID, GiniClust,
  CellSIUS, and FiRE by F1 score.
when_to_use_negative:
- Input is a single unlabeled scRNA-seq dataset with no planted ground-truth cells,
  so precision/recall/F1 cannot be computed
- The goal is speed/memory benchmarking rather than rare-cell call accuracy — the
  paper evaluates those separately via runtime and memory profiling
- Major cell types must first be determined by clustering (e.g. CellSIUS-style workflows);
  this skill only scores calls against planted labels
tools:
- name: GapClust
  role: Primary rare-cell detection method benchmarked; identifies candidate k when
    distance-score skewness exceeds 2
  repo: github.com/fabotao/GapClust
- name: RaceID
  role: Comparator method run on every benchmark dataset with published default settings
- name: GiniClust
  role: Comparator method run on every benchmark dataset with published default settings
- name: CellSIUS
  role: Comparator method run on every benchmark dataset with published default settings
- name: FiRE
  role: 'Comparator method run twice per dataset: default threshold and relaxed IQR
    threshold'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:e18c8945c0e382a4d31a4e429142384a8331fe7bd148484673aecc189bb61578
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# f1-score-evaluation-of-rare-cell-calls

## Summary

Benchmark rare-cell detection methods (GapClust, RaceID, GiniClust, CellSIUS, FiRE) by scoring each method's rare-cell calls against planted ground-truth labels using precision, recall, and F1 on the 99 rare-cell and 140 doublet 68k PBMC subsample datasets.

## When to use

Use when benchmarking rare-cell or doublet detection on subsampled scRNA-seq data with known planted cells, to compare methods like GapClust, RaceID, GiniClust, CellSIUS, and FiRE by F1 score.

## When NOT to use

- Input is a single unlabeled scRNA-seq dataset with no planted ground-truth cells, so precision/recall/F1 cannot be computed
- The goal is speed/memory benchmarking rather than rare-cell call accuracy — the paper evaluates those separately via runtime and memory profiling
- Major cell types must first be determined by clustering (e.g. CellSIUS-style workflows); this skill only scores calls against planted labels

## Inputs

- 99 rare-cell benchmark datasets (2-100 CD14+ monocytes planted into 500 NK + 500 B cells from the 68k PBMC dataset)
- 140 doublet benchmark datasets (2 monocytes at mixing proportions 0.08%-5% from the 68k PBMC dataset)
- Rare-cell calls from GapClust, RaceID, GiniClust, CellSIUS, and FiRE (FiRE under both default and relaxed IQR thresholds)
- Ground-truth labels identifying the planted rare cells / doublets

## Outputs

- Per-method precision, recall, and F1 scores aggregated across the abundance gradient and per threshold setting
- benchmark_f1_results.csv results table
- F1 plots reproducing Fig. 2b and Supplementary Figs. 5 and 8

## How to apply

Construct 99 rare-cell benchmark datasets by embedding 2-100 CD14+ monocytes into homogeneous populations of 500 NK cells plus 500 B cells subsampled from the 68k PBMC dataset, plus 140 doublet datasets with 2 monocytes at mixing proportions from 0.08% to 5%. Run GapClust on every dataset, identifying candidate k where the skewness of the distance-score distribution exceeds 2, and run RaceID, GiniClust, and CellSIUS with published default settings; run FiRE twice, with the default threshold and with a relaxed IQR threshold. Score each method's rare-cell calls against the ground-truth labels with precision, recall, and F1, aggregating per method across the abundance gradient and per threshold setting. Verify the F1 results reproduce Fig. 2b and Supplementary Figs. 5 and 8, and write the aggregated values to benchmark_f1_results.csv.

## Related tools

- **GapClust** (Primary rare-cell detection method benchmarked; identifies candidate k when distance-score skewness exceeds 2) — github.com/fabotao/GapClust
- **RaceID** (Comparator method run on every benchmark dataset with published default settings)
- **GiniClust** (Comparator method run on every benchmark dataset with published default settings)
- **CellSIUS** (Comparator method run on every benchmark dataset with published default settings)
- **FiRE** (Comparator method run twice per dataset: default threshold and relaxed IQR threshold)

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- F1 plot reproduces the paper's Fig. 2b and Supplementary Figs. 5 and 8
- GapClust achieves higher F1 than GiniClust, RaceID, CellSIUS, and FiRE (including FiRE with relaxed IQR thresholds) across the 2-100 planted monocyte gradient
- benchmark_f1_results.csv exists with per-method precision, recall, and F1 aggregated across the abundance gradient and per threshold setting
- FiRE is evaluated under both the default and relaxed IQR threshold settings before aggregation
- F1 scores fall within the expected [0, 1] range for every method/dataset combination

## Limitations

- No version numbers given for the compared tools (GiniClust, RaceID, CellSIUS, FiRE, EDGE)
- No parameter settings for GapClust runs (e.g. K, skewness threshold) reported in the discussion section
- FiRE's default threshold can affect comparisons — relaxed IQR threshold runs are required alongside defaults
- RaceID has unrealistic running time on datasets of thousands of cells, limiting its use at full 68k scale

## Evidence

- [methods] Workflow step 6: scoring procedure: [evidence span withheld — anchor status: not_found]
- [readme] README performance statement: "In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F<sub>1</sub> score."
- [abstract] Benchmark finding: "Benchmarking on diverse experimental datasets demonstrates the superior performance of GapClust compared to other recently proposed methods"
- [intro] Comparator methods introduced: "eminent among these are rare cell-type identification (RaceID)13, GiniClust14, cell subtype identification from up-regulated gene sets (CellSIUS)15 and finder of rare entities (FiRE)16"
