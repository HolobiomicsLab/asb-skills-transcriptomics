---
name: scrna-seq-method-benchmarking
description: Use when evaluating which automatic scRNA-seq cell-type identification
  method (eager, lazy, or marker-based supervised) performs on a manually annotated
  dataset, measured by accuracy, F1-score, unlabeling rate and speed.
when_to_use_negative:
- The dataset lacks manually annotated cell-type labels and no marker lists are available,
  so neither reference-based nor marker-based training is possible
- The goal is de novo cell discovery or clustering rather than benchmarking of supervised
  annotation methods
- Input is already a per-method summary table of metrics — the cross-validation evaluation
  has already been performed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper providing eagersupervised, lazysupervised and markersupervised interfaces
    plus the self-projection function used to run five-fold cross-validation of each
    method
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: caret (R package)
  role: createFolds function used to split cells into five cell-type-stratified subsets
    for cross-validation
- name: Seurat
  role: Seurat pipeline (NormalizeData, FindAllMarkers) used to find cell type specific
    markers and as an evaluated eager-supervised method
- name: CELLBLAST
  role: Lazysupervised method evaluated via lazysupervised(train,test,label_train,method='CELLBLAST')
  repo: https://github.com/gao-lab/Cell_BLAST
- name: ACTINN
  role: Eager-supervised method evaluated on the stratified folds
  repo: https://github.com/mafeiyang/ACTINN
- name: CaSTLe
  role: Eager-supervised method evaluated on the stratified folds
  repo: https://github.com/yuvallb/CaSTLe
- name: scmap-cell
  role: Lazysupervised method evaluated on the stratified folds
  repo: https://github.com/hemberg-lab/scmap
- name: MarkerCount
  role: Cell-type identifier available in reference- and marker-based mode, usable
    from R via reticulate
  repo: https://github.com/combio-dku/MarkerCount
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:824170beac9998e997a4b258e6a9877b8f35a938550e79fe8273091d23e52fef
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
  - CaSTLe
  - MarkerCount
  - R
  - Seurat
  - caret
  - caret (R package)
  - scmap-cell
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# scrna-seq-method-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark automatic single-cell RNA-seq cell-type identification methods via five-fold stratified cross-validation self-projection, comparing prediction accuracy, F1-score, unlabeling rate and running time across tissues using the AutomaticCellTypeIdentification R package.

## When to use

Use when evaluating which automatic scRNA-seq cell-type identification method (eager, lazy, or marker-based supervised) performs best on a manually annotated dataset, measured by accuracy, F1-score, unlabeling rate and speed.

## When NOT to use

- The dataset lacks manually annotated cell-type labels and no marker lists are available, so neither reference-based nor marker-based training is possible
- The goal is de novo cell discovery or clustering rather than benchmarking of supervised annotation methods
- Input is already a per-method summary table of metrics — the cross-validation evaluation has already been performed

## Inputs

- Raw count matrix with manually annotated cell-type labels (Tabula Muris, PBMC SCP424, GSE131907 lung, GSE116470 brain)
- Marker lists (from PanglaoDB, CellMarker or CancerSEA) for markersupervised methods

## Outputs

- Per-method, per-tissue accuracy, mean per-cell-type F1-score, unlabeling rate and running time averaged over five cross-validation rounds
- Method-by-tissue metric matrix rendered as a heatmap (e.g. Fig. 2B layout)

## How to apply

Download count matrices with manually annotated labels (e.g. Tabula Muris from https://tabula-muris.ds.czbiohub.org/) and preprocess: filter genes not expressed in any cells, drop cells with fewer than 200 expressed genes, cells exceeding twice the standard deviation of the mean expressed-gene count, cells with more than 20% mitochondrial gene expression, and delete cells lacking cell-type information. Split cells into five cell-type-stratified folds using caret's createFolds, train each method on four subsets and test on the held-out subset, repeating over five cross-validation rounds via AutomaticCellTypeIdentification's self-projection function. Methods are grouped as eagersupervised (e.g. ACTINN, CaSTLe, CHETAH, Seurat, SciBet, SingleR), lazysupervised (CELLBLAST, scmapcell) and markersupervised (SCSA, SCINA, Markercount), invoked via the package's eagersupervised/lazysupervised/markersupervised interfaces with count matrices (rows = genes, columns = cells). Compute accuracy on labeled cells, mean per-cell-type F1-score and unlabeling rate per method and tissue, average over the five rounds, and assemble a method-by-tissue matrix for heatmap visualization.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper providing eagersupervised, lazysupervised and markersupervised interfaces plus the self-projection function used to run five-fold cross-validation of each method) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **caret (R package)** (createFolds function used to split cells into five cell-type-stratified subsets for cross-validation)
- **Seurat** (Seurat pipeline (NormalizeData, FindAllMarkers) used to find cell type specific markers and as an evaluated eager-supervised method)
- **CELLBLAST** (Lazysupervised method evaluated via lazysupervised(train,test,label_train,method='CELLBLAST')) — https://github.com/gao-lab/Cell_BLAST
- **ACTINN** (Eager-supervised method evaluated on the stratified folds) — https://github.com/mafeiyang/ACTINN
- **CaSTLe** (Eager-supervised method evaluated on the stratified folds) — https://github.com/yuvallb/CaSTLe
- **scmap-cell** (Lazysupervised method evaluated on the stratified folds) — https://github.com/hemberg-lab/scmap
- **MarkerCount** (Cell-type identifier available in reference- and marker-based mode, usable from R via reticulate) — https://github.com/combio-dku/MarkerCount

## Examples

```
devtools::install_github('xiebb123456/AutomaticCellTypeIdentification'); eagersupervised(train, test, label_train, method='Seurat')
```

## Evaluation signals

- Heatmap output exists with one row per evaluated method and one column per retained tissue (eleven Tabula Muris tissues after excluding those lacking both 10x and Smart-seq2 data)
- All metric values (accuracy, F1-score, unlabeling rate) fall within the 0–1 range
- Most methods achieve high accuracy and F1-score and low unlabeling rate across tissues except marrow, consistent with the reported self-projection finding
- Each tissue was evaluated with five cell-type-stratified folds over five cross-validation rounds, with metrics averaged across rounds
- Filtered cells/genes match the stated thresholds: genes expressed in no cells removed, <200 expressed genes, >2 SD of mean expressed genes, >20% mitochondrial expression, and cells without labels deleted

## Limitations

- Tissues lacking both 10x and Smart-seq2 data (aorta, brain myeloid, brain non-myeloid, diaphragm, fat, heart, large intestine, skin) must be excluded from cross-tissue comparison
- Marrow is an outlier where most methods do not achieve high accuracy/F1-score, so conclusions about tissue-unbiased performance do not extend there
- Python-based marker methods require the conda environment to be loaded before running in R
- Overall performance involves six dimensions (accuracy, F1-score, speed, unlabeling rate, sensitivity, specificity); a method optimal on one metric may not be optimal overall

## Evidence

- [readme] package installation: [evidence span withheld — anchor status: external_doc]
- [readme] input format: [evidence span withheld — anchor status: external_doc]
