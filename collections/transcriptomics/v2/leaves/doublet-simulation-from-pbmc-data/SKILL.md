---
name: doublet-simulation-from-pbmc-data
description: 'Use when constructing controlled doublet-detection benchmarks: you have
  the 68k PBMC dataset with NK, B and CD14+ monocyte labels and need synthetic doublets
  at known abundance gradients (e.g. 2 monocytes at 0.08%-5%) to evaluate F1 of rare-cell
  detectors.'
when_to_use_negative:
- The goal is to detect naturally occurring rare cells in a real dataset rather than
  to benchmark detectors against synthetic planted doublets
- The input dataset lacks cell-type labels (e.g. NK, B, CD14+ monocytes) needed to
  construct homogeneous subsamples and ground truth
- Only rare-cell (single-population) benchmarks are needed — the 99 rare-cell datasets
  with 2-100 planted monocytes are the appropriate construction instead
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: GapClust
  role: Primary method evaluated on the simulated doublet datasets; identifies candidate
    k where skewness of the distance-score distribution exceeds 2
  repo: github.com/fabotao/GapClust
- name: RaceID
  role: Comparison method run on every doublet dataset with published default settings
- name: GiniClust
  role: Comparison method run on every doublet dataset with published default settings
- name: CellSIUS
  role: Comparison method run on every doublet dataset with published default settings
- name: FiRE
  role: 'Comparison method run twice per dataset: default threshold and relaxed IQR
    threshold'
- name: Seurat
  role: R prerequisite (>= 3.1.0) for installing and running GapClust on the simulated
    datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:945d253a3ae962fde433b0904f404159bb14a18a3898a38dffc14ff48d436bef
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# doublet-simulation-from-pbmc-data

## Summary

A benchmark-construction skill that builds 140 doublet detection datasets by embedding 2 monocytes into subsampled 68k PBMC data at mixing proportions from 0.08% to 5%, so that rare-cell/doublet methods like GapClust, RaceID, GiniClust, CellSIUS and FiRE can be scored by precision, recall and F1 against known ground truth.

## When to use

Use when constructing controlled doublet-detection benchmarks: you have the 68k PBMC dataset with NK, B and CD14+ monocyte labels and need synthetic doublets at known abundance gradients (e.g. 2 monocytes at 0.08%-5%) to evaluate F1 of rare-cell detectors.

## When NOT to use

- The goal is to detect naturally occurring rare cells in a real dataset rather than to benchmark detectors against synthetic planted doublets
- The input dataset lacks cell-type labels (e.g. NK, B, CD14+ monocytes) needed to construct homogeneous subsamples and ground truth
- Only rare-cell (single-population) benchmarks are needed — the 99 rare-cell datasets with 2-100 planted monocytes are the appropriate construction instead

## Inputs

- 68k PBMC scRNA-seq dataset with cell-type labels (NK cells, B cells, CD14+ monocytes)
- Ground-truth labels of planted rare monocytes per simulated dataset
- List of target mixing proportions spanning 0.08% to 5%

## Outputs

- 140 doublet benchmark datasets with planted 2-monocyte ground truth
- Per-method precision, recall and F1 scores aggregated per method across the abundance gradient and per threshold setting
- benchmark_f1_results.csv results table and F1 plots reproducing Fig. 2b and Supplementary Figs. 5 and 8

## How to apply

Subsample the 68k PBMC dataset to homogeneous populations (e.g. 500 NK cells plus 500 B cells) and plant 2 CD14+ monocytes into each subsample, sweeping the mixing proportion from 0.08% up to 5% to generate 140 doublet benchmark datasets. Run GapClust on every dataset, identifying candidate k where the skewness of the distance-score distribution exceeds 2, and run RaceID, GiniClust and CellSIUS with their published defaults; run FiRE twice, once with the default threshold and once with a relaxed IQR threshold. Score each method's calls against the planted-monocyte ground-truth labels with precision, recall and F1, then aggregate per method across the abundance gradient and per threshold setting. The rationale is that embedding known doublets at controlled proportions isolates each method's sensitivity to rare populations without the confounding of naturally occurring rare cells.

## Related tools

- **GapClust** (Primary method evaluated on the simulated doublet datasets; identifies candidate k where skewness of the distance-score distribution exceeds 2) — github.com/fabotao/GapClust
- **RaceID** (Comparison method run on every doublet dataset with published default settings)
- **GiniClust** (Comparison method run on every doublet dataset with published default settings)
- **CellSIUS** (Comparison method run on every doublet dataset with published default settings)
- **FiRE** (Comparison method run twice per dataset: default threshold and relaxed IQR threshold)
- **Seurat** (R prerequisite (>= 3.1.0) for installing and running GapClust on the simulated datasets)

## Examples

```
devtools::install_github("fabotao/GapClust"); library(GapClust); GapClust(data = doublet_benchmark_dataset)
```

## Evaluation signals

- Exactly 140 doublet datasets are generated, each containing 2 planted monocytes at a mixing proportion within 0.08%-5%
- GapClust identifies candidate k only where the skewness of the distance-score distribution exceeds 2
- GapClust achieves higher F1 scores than GiniClust, RaceID, CellSIUS and FiRE (including FiRE with relaxed IQR thresholds) across the abundance gradient
- F1 plots reproduce Fig. 2b and Supplementary Figs. 5 and 8
- Results table benchmark_f1_results.csv is written with per-method precision, recall and F1 per threshold setting

## Limitations

- Compared tool versions (GiniClust, RaceID, CellSIUS, FiRE, EDGE) are not specified, complicating exact reproduction
- Results for most real datasets are deferred to unlinked supplementary figures (Supplementary Figs. 14-22, Supplementary Table 5) not included in the main text
- GapClust's design exploits the gap between a minor cluster and its neighbouring abundant cluster, so performance depends on planted doublets forming a separable minor cluster

## Evidence

- [other] 140 doublet benchmark datasets by embedding 2 monocytes at mixing proportions from 0.08% to 5%.: [evidence span withheld — anchor status: not_found]
- [discussion] GapClust candidate k via skewness threshold.: "Candidate k can be identified if the skewness<k> > 2."
- [abstract] GapClust as light-weight rare-cell detection algorithm.: "Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets"
- [readme] README performance claim on F1 score.: "In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F<sub>1</sub> score."
- [discussion] Doublet benchmark datasets listed in discussion.: "ten subsampled datasets from the 293T-Jurkat dataset and 140 subsampled datasets from the 68 k PBMC dataset for doublets identification"
