---
name: cell-type-classification-runtime-benchmarking
description: Use when comparing or benchmarking the running time of automatic scRNA-seq
  cell-type identification methods on subsets of varying training/testing sizes, e.g.
  reproducing the Fig. 4 speed ranking on the mouse brain dataset GSE116470.
when_to_use_negative:
- The goal is accuracy, F1-score, unlabeling rate, or new cell-type identification
  performance rather than running-time scaling
- The dataset is not a single scRNA-seq expression matrix with labels (e.g. bulk RNA-seq,
  proteomics) — cell-type identification classifiers do not apply
- Only one fixed training/testing size is available, so no runtime scaling curve over
  dataset sizes can be constructed
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper that runs the evaluated cell-type identification methods (eagersupervised/lazysupervised/markersupervised)
    so their prediction times can be recorded uniformly
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: caret (R package)
  role: createDataPartition is used to generate training and testing subsets of varying
    cell counts for the runtime scaling experiment
- name: scmap-cluster
  role: lazy supervised method expected to rank fastest in the benchmark, serving
    as the expected top of the runtime ranking
  repo: https://github.com/hemberg-lab/scmap
- name: Seurat
  role: eager supervised method in the benchmark and the source of the preprocessing/marker-identification
    pipeline conventions used for the data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:4ba875e5391c2d56bbec9f9881cd3db3670630c35d568cbd9d9bf8cf152b0389
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
  - scmap-cell
  - scmap-cluster
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# cell-type-classification-runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the computation time of automatic cell-type identification methods for scRNA-seq across varying training-set (500–10,000 cells) and testing-set (1,000–50,000 cells) sizes, using the AutomaticCellTypeIdentification R wrapper, to rank methods by prediction speed (as in Fig. 4 of the paper).

## When to use

Use when comparing or benchmarking the running time of automatic scRNA-seq cell-type identification methods on subsets of varying training/testing sizes, e.g. reproducing the Fig. 4 speed ranking on the mouse brain dataset GSE116470.

## When NOT to use

- The goal is accuracy, F1-score, unlabeling rate, or new cell-type identification performance rather than running-time scaling
- The dataset is not a single scRNA-seq expression matrix with labels (e.g. bulk RNA-seq, proteomics) — cell-type identification classifiers do not apply
- Only one fixed training/testing size is available, so no runtime scaling curve over dataset sizes can be constructed

## Inputs

- mouse brain scRNA-seq count matrix (GSE116470) with manually annotated cell-type labels
- training-set subsets of 500–10,000 cells generated via caret::createDataPartition
- testing-set subsets of 1,000–50,000 cells generated via caret::createDataPartition

## Outputs

- runtime_scaling.csv with prediction computing time per method per training/testing size combination
- speed performance score (reciprocal of base-10 log of computing time) for the reference 20,000-cell prediction run
- Fig.-4-style ranking of the 24 evaluated methods from fastest (scmap-cluster) to slowest (neural-network methods)

## How to apply

Download the mouse brain scRNA-seq dataset (GSE116470) and apply the Seurat-compatible filtering: remove genes not expressed in any cells, cells with fewer than 200 expressed genes, cells exceeding twice the standard deviation of the mean expressed genes, and cells with more than 20% mitochondrial gene expression. Generate training/testing subsets with the 'createDataPartition' function of the caret R package, varying training size from 500 to 10,000 cells and testing size from 1,000 to 50,000 cells. Run each method through the AutomaticCellTypeIdentification package (eagersupervised methods such as ACTINN, CaSTLe, CHETAH, SciBet, scmap-cluster, scVI, Seurat, SingleCellNet, SingleR; lazysupervised methods CELLBLAST and scmap-cell; markersupervised methods SCINA, SCSA, scTyper, DigitalCellSorter, MarkerCount), recording prediction computing time for each training/testing combination. Compute a speed performance score as the reciprocal of the base-10 logarithm of computing time for the reference run predicting 20,000 cells, compile runtimes into runtime_scaling.csv, and check the expected ranking: scmap-cluster fastest, SCINA/SingleCellNet/SciBet/SingleR/scHPL fast, and neural-network-based methods (scVI, MARS, CellAssign, CELLBLAST) slowest, with more than half of methods finishing within 100 s.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper that runs the evaluated cell-type identification methods (eagersupervised/lazysupervised/markersupervised) so their prediction times can be recorded uniformly) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **caret (R package)** (createDataPartition is used to generate training and testing subsets of varying cell counts for the runtime scaling experiment)
- **scmap-cluster** (lazy supervised method expected to rank fastest in the benchmark, serving as the expected top of the runtime ranking) — https://github.com/hemberg-lab/scmap
- **Seurat** (eager supervised method in the benchmark and the source of the preprocessing/marker-identification pipeline conventions used for the data)

## Examples

```
eagersupervised(train,test,label_train,method='Seurat')
```

## Evaluation signals

- runtime_scaling.csv is produced with one runtime entry per method per training (500–10,000) and testing (1,000–50,000) size combination
- scmap-cluster ranks as the fastest method across size combinations
- Neural-network-based methods (scVI, MARS, CellAssign, CELLBLAST) rank among the slowest
- More than half of the evaluated methods complete prediction within 100 s
- Speed performance score equals 1/log10(computing time) for the reference prediction of 20,000 brain cells

## Limitations

- Runtime depends on the specific dataset (mouse brain GSE116470) and hardware; rankings may shift on other datasets
- The R interfaces must be matched to method category: markersupervised methods take marker lists (test, marker) rather than train matrices, so timing comparisons require category-specific calls
- Python-based methods within the wrapper require a conda environment loaded in R before timing runs

## Evidence

- [methods] createDataPartition: "by the ‘createDataPartition’ function in the ‘caret’ R package"
- [methods] mitochondrial filter threshold: "the expression of mitochondrial genes in the cells is more than 20%"
- [methods] overall performance metrics: "Overall performance of automatic methods consists of accuracy, F1-score, speed, unlabeling rate, sensitivity and specificity"
- [readme] wrapper method interfaces: [evidence span withheld — anchor status: not_found]
