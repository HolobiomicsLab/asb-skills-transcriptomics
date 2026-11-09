---
name: cross-platform-label-transfer
description: Use when a labeled training count matrix (genes as rows, cells as columns)
  plus an unlabeled testing matrix must be annotated by routing each method to one
  of three wrappers by its category.
when_to_use_negative:
- The input is raw clustering output with no reference labels and no marker list —
  unsupervised clustering plus manual annotation of differentially expressed cluster
  markers is the traditional route described instead
- Only an unlabeled query and no training data are available and marker-based annotation
  is intended — call the markersupervised wrapper directly rather than the eager/lazy
  wrappers that require labeled training matrices
- Python-based methods are invoked without the required conda environment loaded,
  or the method name does not resolve to exactly one of the three wrappers in the
  routing table
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper package exposing the eagersupervised, lazysupervised and markersupervised
    interface functions that route each integrated method
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: Seurat
  role: Derives cell-type-specific markers from the training data (FindAllMarkers
    with labels as active.ident) and is itself an eager-supervised method
- name: CELLBLAST v0.3.8
  role: Lazy-supervised method invoked via lazysupervised(train,test,label_train,method='CELLBLAST')
  repo: https://github.com/gao-lab/Cell_BLAST
- name: scmap-cell v1.6.0
  role: Lazy-supervised instance-based method routed through the lazysupervised wrapper
  repo: https://github.com/hemberg-lab/scmap
- name: ACTINN (master)
  role: Eager-supervised method routed through the eagersupervised wrapper
  repo: https://github.com/mafeiyang/ACTINN
- name: CaSTLe v1.0.0.2
  role: Eager-supervised method routed through the eagersupervised wrapper
  repo: https://github.com/yuvallb/CaSTLe
- name: SciBet v0.1.0
  role: Eager-supervised method routed through the eagersupervised wrapper
  repo: https://github.com/zwj-tina/scibetR
- name: Garnett v0.1.19
  role: Marker-supervised method invoked via markersupervised(test,marker,...) with
    a marker gene list
  repo: https://github.com/cole-trapnell-lab/garnett
- name: caret (R package)
  role: Splits the annotated dataset into training/testing sets via createDataPartition
    and five-fold createFolds
- name: MarkerCount
  role: Cell-type identifier usable in both marker-based and reference-based modes,
    routed in R via reticulate
  repo: https://github.com/combio-dku/MarkerCount
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:75abdb66746d8180cab3ff8996c007c6e19cd646902be72355ac8aff0ba78c2a
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
  - ACTINN (master)
  - AutomaticCellTypeIdentification
  - CELLBLAST v0.3.8
  - CaSTLe v1.0.0.2
  - Garnett v0.1.19
  - MarkerCount
  - R
  - SciBet v0.1.0
  - Seurat
  - caret (R package)
  - scmap-cell v1.6.0
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# cross-platform-label-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Wraps automatic scRNA-seq cell-type identification methods into the three interface functions (eagersupervised, lazysupervised, markersupervised) of the AutomaticCellTypeIdentification R package, so predictions can be transferred across platforms with a uniform call signature per method category.

## When to use

Use when a labeled training count matrix (genes as rows, cells as columns) plus an unlabeled testing matrix must be annotated by routing each method to one of three wrappers by its category.

## When NOT to use

- The input is raw clustering output with no reference labels and no marker list — unsupervised clustering plus manual annotation of differentially expressed cluster markers is the traditional route described instead
- Only an unlabeled query and no training data are available and marker-based annotation is intended — call the markersupervised wrapper directly rather than the eager/lazy wrappers that require labeled training matrices
- Python-based methods are invoked without the required conda environment loaded, or the method name does not resolve to exactly one of the three wrappers in the routing table

## Inputs

- Training count matrix (rows = genes, columns = cells)
- Manually annotated cell-type labels for the training cells
- Testing count matrix (rows = genes, columns = cells)
- Per-cell-type marker gene list (top 5 or 15 genes per type, or canonical markers from PanglaoDB/CellMarker/CancerSEA) for marker-supervised wrappers
- Named method string (e.g. 'Seurat', 'CELLBLAST', 'SCSA')

## Outputs

- Prediction vector of cell-type labels for each testing cell
- Performance metrics (accuracy, F1-score, unlabeling rate, running time) when evaluated in five-fold self-projection cross-validation

## How to apply

Install the package via devtools::install_github('xiebb123456/AutomaticCellTypeIdentification') and confirm all dependencies in the Description file are present (or use the provided docker image). Read the count matrices and manually annotated labels from public resources (Tabula Muris, PBMC SCP424, GSE131907 lung, GSE116470 brain), then apply the documented filters: drop genes not expressed in any cells, drop cells with fewer than 200 expressed genes, cells exceeding twice the standard deviation of mean expressed genes, mitochondrial expression above 20%, and cells lacking cell-type information. Route eager methods (ACTINN, CaSTLe, CHETAH, SciBet, Seurat, SingleCellNet, etc.) and lazy instance-based methods (CELLBLAST, scmapcell) through calls that take training matrix, testing matrix, and training labels. Route marker-learning methods (SCSA, Garnett, scClassify, scTyper, DigitalCellSorter, SCINA, Markercount) through a call that takes only the testing matrix and a marker list derived from the training data via the Seurat FindAllMarkers pipeline or downloaded from PanglaoDB/CellMarker/CancerSEA. For python-based methods, load the required conda environment at the beginning in R before invoking the wrapper.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper package exposing the eagersupervised, lazysupervised and markersupervised interface functions that route each integrated method) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **Seurat** (Derives cell-type-specific markers from the training data (FindAllMarkers with labels as active.ident) and is itself an eager-supervised method)
- **CELLBLAST v0.3.8** (Lazy-supervised method invoked via lazysupervised(train,test,label_train,method='CELLBLAST')) — https://github.com/gao-lab/Cell_BLAST
- **scmap-cell v1.6.0** (Lazy-supervised instance-based method routed through the lazysupervised wrapper) — https://github.com/hemberg-lab/scmap
- **ACTINN (master)** (Eager-supervised method routed through the eagersupervised wrapper) — https://github.com/mafeiyang/ACTINN
- **CaSTLe v1.0.0.2** (Eager-supervised method routed through the eagersupervised wrapper) — https://github.com/yuvallb/CaSTLe
- **SciBet v0.1.0** (Eager-supervised method routed through the eagersupervised wrapper) — https://github.com/zwj-tina/scibetR
- **Garnett v0.1.19** (Marker-supervised method invoked via markersupervised(test,marker,...) with a marker gene list) — https://github.com/cole-trapnell-lab/garnett
- **caret (R package)** (Splits the annotated dataset into training/testing sets via createDataPartition and five-fold createFolds)
- **MarkerCount** (Cell-type identifier usable in both marker-based and reference-based modes, routed in R via reticulate) — https://github.com/combio-dku/MarkerCount

## Examples

```
markersupervised(test,marker,method='SCSA')
```

## Evaluation signals

- The package source loads in R without error and each of the three wrappers resolves a named method from its category without argument errors
- Each wrapper returns a prediction vector with one label per testing cell (number of predictions equals number of testing columns)
- Every integrated method name resolves to exactly one wrapper category in the routing table (e.g. SCSA only via markersupervised, CELLBLAST only via lazysupervised)
- In five-fold self-projection, accuracy, F1-score, unlabeling rate and running time are computed for each method and are finite values
- Marker-supervised calls succeed when markers are supplied as top 5 or 15 genes per cell type derived from Seurat FindAllMarkers or canonical marker resources

## Limitations

- Python-based methods require the corresponding conda environment to be loaded at the beginning in R before the wrapper call
- AutomaticCellTypeIdentification is a wrapper of published methods, so all dependencies listed in the Description file must be installed separately (or use the docker image)
- Training/testing inputs must be count matrices with genes as rows and cells as columns; cells with no cell-type information must be deleted before splitting

## Evidence

- [readme] Now, three interface of eagersupervised`` lazysupervised`` markersupervised methods supports the available automatic methods.: [evidence span withheld — anchor status: not_found]
- [readme] The input of training and testing data is count matrix, the row is gene and the column is cell.: [evidence span withheld — anchor status: external_doc]
