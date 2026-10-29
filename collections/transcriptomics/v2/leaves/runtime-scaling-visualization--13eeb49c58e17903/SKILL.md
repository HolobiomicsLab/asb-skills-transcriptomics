---
name: runtime-scaling-visualization
description: Use when comparing the computational cost of cell-type annotation methods
  on scRNA-seq data, or when reproducing a runtime scaling benchmark across varying
  training-set (e.g. 500–10,000 cells) and testing-set (e.g. 1,000–50,000 cells) sizes.
when_to_use_negative:
- The analysis goal is prediction accuracy, F1-score, or unlabeling rate rather than
  computational cost — this skill records runtime only.
- The input is unlabeled data with no reference annotations, since the benchmark requires
  a pre-annotated training set for each method run.
- The methods under comparison are not implemented in (or wrappable by) the AutomaticCellTypeIdentification
  package, so timing would not be measured under comparable conditions.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper used to run each cell-type identification method and record prediction
    computing time under uniform conditions
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: caret (R package)
  role: createDataPartition generates training/testing subsets of varying sizes for
    the runtime scaling experiment
- name: R
  role: Environment in which the benchmark, timing, and score computation are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:a54fc8d7ccc6aeeef2ff4f4bc7367e8b414d48893f16b8c577a460eb64798400
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 34
---

# runtime-scaling-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking skill for measuring how the prediction runtime of automatic cell-type identification methods scales with training and testing dataset sizes, and converting computing time into a comparable speed performance score (as done for the 24+ methods evaluated in Fig. 4 of the scRNA-seq cell-type identification review).

## When to use

Use when comparing the computational cost of cell-type annotation methods on scRNA-seq data, or when reproducing a runtime scaling benchmark across varying training-set (e.g. 500–10,000 cells) and testing-set (e.g. 1,000–50,000 cells) sizes.

## When NOT to use

- The analysis goal is prediction accuracy, F1-score, or unlabeling rate rather than computational cost — this skill records runtime only.
- The input is unlabeled data with no reference annotations, since the benchmark requires a pre-annotated training set for each method run.
- The methods under comparison are not implemented in (or wrappable by) the AutomaticCellTypeIdentification package, so timing would not be measured under comparable conditions.

## Inputs

- Preprocessed scRNA-seq count matrix (genes as rows, cells as columns) with manually annotated cell-type labels for the training set
- Training and testing subsets generated via caret::createDataPartition at multiple size combinations (500–10,000 training cells; 1,000–50,000 testing cells)
- Reference benchmark dataset, e.g. mouse brain GSE116470 filtered for <200 genes/cell, >2 SD mean gene count, and >20% mitochondrial expression

## Outputs

- runtime_scaling.csv: table of computing times per method per training/testing size combination
- Speed performance scores (reciprocal of log10 of computing time) per method for a 20,000-cell reference prediction
- Runtime ranking suitable for a speed comparison figure (e.g. Fig. 4-style bar/scatter plot)

## How to apply

Download the benchmark scRNA-seq dataset (e.g. mouse brain GSE116470) and preprocess it with the Seurat-compatible filters: remove genes not expressed in any cells, cells with fewer than 200 expressed genes, cells exceeding twice the standard deviation of the mean expressed genes, and cells with more than 20% mitochondrial gene expression. Generate subsets with the 'createDataPartition' function of the caret R package, varying training-set size from 500 to 10,000 cells and testing-set size from 1,000 to 50,000 cells. Run each method through the AutomaticCellTypeIdentification R package (eagersupervised/lazysupervised/markersupervised interfaces) on each train/test size combination, recording only the prediction (not training) computing time. Compute a speed performance score as the reciprocal of the base-10 logarithm of computing time for a reference run predicting 20,000 brain cells, so faster methods get higher scores. Validate by confirming the expected ordering: scmap-cluster fastest (with SCINA, SingleCellNet, SciBet, SingleR and scHPL also fast) and neural-network-based methods (scVI, MARS, CellAssign, CELLBLAST) slowest.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper used to run each cell-type identification method and record prediction computing time under uniform conditions) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **caret (R package)** (createDataPartition generates training/testing subsets of varying sizes for the runtime scaling experiment)
- **R** (Environment in which the benchmark, timing, and score computation are executed)

## Examples

```
eagersupervised(train,test,label_train,method='Seurat')
```

## Evaluation signals

- runtime_scaling.csv exists and contains one runtime entry per method per train/test size combination
- scmap-cluster ranks as the fastest method in the compiled runtimes
- Neural-network-based methods (scVI, MARS, CellAssign, CELLBLAST) rank among the slowest
- More than half of the evaluated methods complete prediction in no more than 100 s
- Speed performance score equals the reciprocal of log10(computing time) for the 20,000-cell reference prediction, with faster methods receiving higher scores

## Limitations

- Runtime results are hardware- and implementation-dependent; the ranking reflects the benchmark environment, not absolute speed.
- The benchmark records computing time for prediction, so training cost is not captured by the speed score.
- Runtime scaling conclusions (e.g. scmap-cluster fastest) were established on specific datasets such as mouse brain GSE116470 and may not transfer to other tissues or platforms.

## Evidence

- [results] scmap-cluster is the fastest automatic cell-type identification method: [evidence span withheld — anchor status: not_found]
- [readme] Package interfaces: [evidence span withheld — anchor status: not_found]
