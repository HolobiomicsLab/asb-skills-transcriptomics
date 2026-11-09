---
name: scrna-dataset-downsampling
description: Use when benchmarking cell-type identification methods on scRNA-seq count
  matrices and you need training sets of 500–10,000 cells and testing sets of 1,000–50,000
  cells, or ~10,000-cell stratified subsets.
when_to_use_negative:
- Input data has not yet been quality-filtered (genes not expressed in any cells,
  cells with <200 genes, >20% mitochondrial expression, or unlabeled cells still present)
  — filtering must precede partitioning
- The analysis goal is marker-based (markersupervised) methods such as SCSA, SCINA
  or DigitalCellSorter that require a marker list rather than training/testing splits
- Data are bulk RNA-seq or already a precomputed feature/signature table with no per-cell
  labels to stratify on
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: caret (R package)
  role: Provides createDataPartition for stratified downsampling and createFolds for
    five-fold cross-validation splits
- name: AutomaticCellTypeIdentification
  role: Consumes the downsampled training/testing count matrices to run the 24+ evaluated
    cell-type identification methods and record prediction time
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: R
  role: Execution environment for the downsampling and benchmarking scripts
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:f0ec379442032ec6ab15b2175a62ba1d96a1a6e61fb149d1b635f82196bc16c1
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 31
---

# scrna-dataset-downsampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates stratified scRNA-seq training/testing subsets of controlled size with the caret R package's createDataPartition (and createFolds for five-fold cross-validation), enabling benchmarks where computation time and accuracy are measured as dataset size varies.

## When to use

Use when benchmarking cell-type identification methods on scRNA-seq count matrices and you need training sets of 500–10,000 cells and testing sets of 1,000–50,000 cells, or ~10,000-cell stratified subsets.

## When NOT to use

- Input data has not yet been quality-filtered (genes not expressed in any cells, cells with <200 genes, >20% mitochondrial expression, or unlabeled cells still present) — filtering must precede partitioning
- The analysis goal is marker-based (markersupervised) methods such as SCSA, SCINA or DigitalCellSorter that require a marker list rather than training/testing splits
- Data are bulk RNA-seq or already a precomputed feature/signature table with no per-cell labels to stratify on

## Inputs

- Preprocessed scRNA-seq count matrix (rows = genes, columns = cells) with manually annotated cell-type labels, e.g. GSE116470 brain or GSE131907 lung data
- Target training-set sizes (e.g. 500 to 10,000 cells) and testing-set sizes (e.g. 1,000 to 50,000 cells)
- caret R package installed alongside the AutomaticCellTypeIdentification package

## Outputs

- Stratified training/testing subset pairs with preserved cell-type proportions (runtime_scaling.csv rows when used for the timing benchmark)
- ~10,000-cell downsampled datasets with equal proportion of each cell type per dataset
- Five cross-validation folds from createFolds for self-projection evaluation

## How to apply

After preprocessing the count matrix (removing genes not expressed in any cells, cells with <200 expressed genes, cells exceeding twice the standard deviation of the mean expressed genes, cells with >20% mitochondrial expression, and cells lacking labels), apply caret::createDataPartition to the cell-type labels so each subset retains an equal proportion of every cell type. For the computation-time benchmark on the mouse brain dataset (GSE116470), vary training-set size from 500 to 10,000 cells and testing-set size from 1,000 to 50,000 cells, then feed each training/testing pair into the AutomaticCellTypeIdentification package (e.g. eagersupervised(train, test, label_train, method='Seurat')) and record prediction runtime. For the lung-tumor task (GSE131907), downsample both normal-lung and tumor datasets to ~10,000 cells with equal cell-type proportions before training. For accuracy evaluation without explicit downsampling, use caret::createFolds to split data into five subsets for five-fold cross-validation.

## Related tools

- **caret (R package)** (Provides createDataPartition for stratified downsampling and createFolds for five-fold cross-validation splits)
- **AutomaticCellTypeIdentification** (Consumes the downsampled training/testing count matrices to run the 24+ evaluated cell-type identification methods and record prediction time) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **R** (Execution environment for the downsampling and benchmarking scripts)

## Examples

```
Rscript -e "devtools::install_github('xiebb123456/AutomaticCellTypeIdentification'); library(caret); idx <- createDataPartition(labels, p=10000/nrow(data), list=FALSE)"
```

## Evaluation signals

- Each downsampled subset contains an equal proportion of every cell type relative to the parent dataset (stratification check via label frequency table)
- Training-set sizes span 500–10,000 cells and testing-set sizes span 1,000–50,000 cells, matching the Fig. 4 scaling grid
- No training cells appear in the testing partition (disjointness check on cell IDs)
- All subsets inherit the preprocessing filters: no genes expressed in zero cells, no cells with <200 genes, >2 SD mean genes, or >20% mitochondrial expression, and no unlabeled cells
- Resulting runtimes compiled per training/testing size combination reproduce the ordering where scmap-cluster ranks fastest and neural-network-based methods (scVI, MARS, CellAssign, CELLBLAST) rank slowest

## Limitations

- Downsampling caps datasets at ~10,000 cells in some analyses, discarding cells that might change accuracy estimates on very large atlases
- Runtime scaling results are dataset-specific (mouse brain GSE116470); timing conclusions may not transfer to other tissues or sequencing protocols
- The caret-based partitioning assumes every cell has a manual cell-type label, since cells with no cell type information were deleted before partitioning

## Evidence

- [readme] devtools::install_github('xiebb123456/AutomaticCellTypeIdentification'): [evidence span withheld — anchor status: external_doc]
