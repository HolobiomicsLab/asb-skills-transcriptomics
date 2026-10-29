---
name: cross-method-speed-comparison
description: Use when you need to quantify and compare prediction runtime of automatic
  cell-type identification methods on scRNA-seq data, especially to assess how runtime
  scales with training/testing set size or to select a fast method for large prediction
  tasks.
when_to_use_negative:
- When the goal is prediction accuracy, F1-score, or unlabeling-rate comparison rather
  than runtime scaling (use the accuracy-evaluation workflow instead)
- When the dataset is not labeled — training-based speed measurement requires annotated
  training data, and the article deletes cells with no cell type information during
  preprocessing
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper that runs all evaluated cell-type identification methods uniformly
    (eagersupervised, lazysupervised, markersupervised interfaces) and is used to
    time prediction
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: caret (R package)
  role: Provides 'createDataPartition' to generate training/testing subsets of varying
    sizes for the runtime scaling experiment
- name: scmap-cluster
  role: Fastest method in the benchmark; reference point for the speed ranking
  repo: https://github.com/hemberg-lab/scmap
- name: Seurat
  role: Preprocessing pipeline used to filter low-quality cells and find cell-type
    specific markers
- name: MarkerCount
  role: One of the evaluated methods (available via eagersupervised and markersupervised
    interfaces)
  repo: https://github.com/combio-dku/MarkerCount
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:f5333079befe01cc1514b2384a89df20797330f327048383164ee580c39575e7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiebb123456/AutomaticCellTypeIdentification
  repo_url: https://github.com/xiebb123456/AutomaticCellTypeIdentification
  tools:
  - ACTINN
  - AutomaticCellTypeIdentification
  - CELLBLAST
  - CHETAH
  - CaSTLe
  - CellAssign
  - CellFishing.jl
  - DigitalCellSorter
  - Garnett
  - MARS
  - MarkerCount
  - R
  - SCINA
  - SCSA
  - SciBet
  - Seurat
  - SingleCellNet
  - SingleR
  - caret (R package)
  - clustifyr
  - scClassifR
  - scHPL
  - scID
  - scLearn
  - scPred
  - scTypER
  - scVI
  - scmap-cluster
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# cross-method-speed-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the computation time of automatic single-cell RNA-seq cell-type identification methods across varying training and testing dataset sizes, ranking methods from fastest (scmap-cluster) to slowest (neural-network-based methods such as scVI, MARS, CellAssign, CELLBLAST), reproducing Fig. 4 of the Xie et al. 2021 evaluation.

## When to use

Use when you need to quantify and compare prediction runtime of automatic cell-type identification methods on scRNA-seq data, especially to assess how runtime scales with training/testing set size or to select a fast method for large prediction tasks.

## When NOT to use

- When the goal is prediction accuracy, F1-score, or unlabeling-rate comparison rather than runtime scaling (use the accuracy-evaluation workflow instead)
- When the dataset is not labeled — training-based speed measurement requires annotated training data, and the article deletes cells with no cell type information during preprocessing

## Inputs

- Mouse brain scRNA-seq count matrix (GSE116470) with manually annotated cell-type labels
- Training-set subsets of 500–10,000 cells generated with caret::createDataPartition
- Testing-set subsets of 1,000–50,000 cells generated with caret::createDataPartition
- Marker lists (e.g., from PanglaoDB/CellMarker) for marker-based methods such as SCSA, SCINA, DigitalCellSorter

## Outputs

- Per-method computing times for prediction across training/testing size combinations (e.g., runtime_scaling.csv)
- Speed performance scores (reciprocal of log10 computing time for predicting 20,000 cells)
- Ranked speed comparison of methods reproducing Fig. 4

## How to apply

Download the mouse brain scRNA-seq dataset (GSE116470) and apply the Seurat-compatible filtering: remove genes not expressed in any cells, cells with fewer than 200 expressed genes, cells exceeding twice the standard deviation of the mean expressed genes, and cells with more than 20% mitochondrial gene expression. Generate subsets with caret's 'createDataPartition' function, varying training-set size from 500 to 10,000 cells and testing-set size from 1,000 to 50,000 cells. Run each evaluated method via the AutomaticCellTypeIdentification R package (eagersupervised, lazysupervised, or markersupervised interfaces) on each training/testing size combination, recording computing time for prediction only (not training). Compute a speed performance score as the reciprocal of the base 10 logarithm of the computing time for the reference task of predicting 20,000 cells from the brain dataset. Compile runtimes (e.g., runtime_scaling.csv) and verify that scmap-cluster ranks fastest, that SCINA, SingleCellNet, SciBet, SingleR and scHPL are also fast, and that neural-network-based methods (scVI, MARS, CellAssign, CELLBLAST) rank slowest; more than half of the methods should need no more than 100 s.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper that runs all evaluated cell-type identification methods uniformly (eagersupervised, lazysupervised, markersupervised interfaces) and is used to time prediction) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **caret (R package)** (Provides 'createDataPartition' to generate training/testing subsets of varying sizes for the runtime scaling experiment)
- **scmap-cluster** (Fastest method in the benchmark; reference point for the speed ranking) — https://github.com/hemberg-lab/scmap
- **Seurat** (Preprocessing pipeline used to filter low-quality cells and find cell-type specific markers)
- **MarkerCount** (One of the evaluated methods (available via eagersupervised and markersupervised interfaces)) — https://github.com/combio-dku/MarkerCount

## Examples

```
eagersupervised(train,test,label_train,method='Seurat')
```

## Evaluation signals

- scmap-cluster should rank as the fastest automatic cell-type identification method in the compiled runtimes
- More than half of the evaluated methods should complete prediction in no more than 100 s
- Neural-network-based methods (scVI, MARS, CellAssign, CELLBLAST) should rank slowest in the speed comparison
- Runtime should be recorded for the prediction step only, per training/testing size combination spanning 500–10,000 training cells and 1,000–50,000 testing cells
- Speed performance score equals 1 / log10(computing time) for the reference prediction of 20,000 brain cells

## Limitations

- Runtime measurements are hardware- and implementation-dependent; relative rankings from this benchmark may not transfer to other systems
- Python-based methods require the corresponding conda environment to be loaded before timing in R
- Methods needing pre-annotated references cannot be timed in marker-only mode; marker-based methods (SCSA, SCINA, DigitalCellSorter, scTyper, MarkerCount) use the markersupervised interface with a marker input instead of a training matrix

## Evidence

- [methods] Speed score definition: [evidence span withheld — anchor status: not_found]
- [readme] Package interfaces: [evidence span withheld — anchor status: not_found]
- [results] Runtime finding: [evidence span withheld — anchor status: not_found]
