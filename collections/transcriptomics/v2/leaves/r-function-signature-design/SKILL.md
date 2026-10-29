---
name: r-function-signature-design
description: Use when wrapping heterogeneous supervised methods into a single R package
  and you must unify their differing input contracts (expression matrices plus labels
  vs. marker gene lists) behind shared dispatch functions.
when_to_use_negative:
- The task is running an unsupervised or semi-supervised identification method rather
  than routing one of the integrated supervised methods through the three wrappers
- The method requires a pre-annotated reference dataset in a format other than the
  genes-by-cells count matrix plus label_train vector, so the uniform training/testing
  signature does not fit
- Only a marker list is available and no testing expression matrix exists, since markersupervised
  still requires the testing count matrix as input
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: AutomaticCellTypeIdentification
  role: R package whose three wrapper interfaces (eagersupervised, lazysupervised,
    markersupervised) the function signatures are designed for
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: Seurat
  role: Used to derive cell-type specific markers (FindAllMarkers with labels as active.ident)
    that serve as the marker-list input to markersupervised wrappers
- name: caret (R package)
  role: Provides createDataPartition and createFolds to split count matrices into
    the training and testing datasets consumed by the eager/lazy wrappers
- name: SCSA
  role: Example marker-learning method dispatched through the markersupervised(test,
    marker, method='SCSA') signature
- name: CELLBLAST
  role: Example lazy-supervised method dispatched through lazysupervised(train, test,
    label_train, method='CELLBLAST')
  repo: https://github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:895f8e6c15c4ca485dcbbbbfade306be7c87492d3524cf55092b2f35f5c0d5cd
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
  - AutomaticCellTypeIdentification
  - CELLBLAST
  - R
  - SCSA
  - Seurat
  - caret (R package)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# r-function-signature-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design a minimal set of R wrapper function signatures (eagersupervised, lazysupervised, markersupervised) so every integrated cell-type identification method resolves to exactly one wrapper with a uniform argument order, distinguishing training/testing expression-matrix inputs from marker-list inputs.

## When to use

Use when wrapping heterogeneous supervised methods into a single R package and you must unify their differing input contracts (expression matrices plus labels vs. marker gene lists) behind shared dispatch functions.

## When NOT to use

- The task is running an unsupervised or semi-supervised identification method rather than routing one of the integrated supervised methods through the three wrappers
- The method requires a pre-annotated reference dataset in a format other than the genes-by-cells count matrix plus label_train vector, so the uniform training/testing signature does not fit
- Only a marker list is available and no testing expression matrix exists, since markersupervised still requires the testing count matrix as input

## Inputs

- Training count matrix (rows = genes, columns = cells)
- Testing count matrix (rows = genes, columns = cells)
- Manually annotated cell-type labels for the training data (label_train)
- Per-cell-type marker gene list (e.g. top 5 or 15 DE genes per type from Seurat FindAllMarkers, or canonical markers from PanglaoDB/CellMarker)
- Method name string resolving to one of the integrated methods

## Outputs

- A prediction vector of cell-type labels for the testing cells, returned by the wrapper for the named method
- A single self-contained R source file exposing the three dispatch wrappers (eagersupervised, lazysupervised, markersupervised)

## How to apply

Read the count matrices (genes as rows, cells as columns) and manually annotated labels from public resources such as Tabula Muris, PBMC (SCP424), lung (GSE131907) and brain (GSE116470) to fix the shared training/testing input format. Define eagersupervised(train, test, label_train, method=...) routing eager methods (e.g. Seurat, ACTINN, CaSTLe, SciBet, SingleR) and lazysupervised with the same signature routing instance-based methods (e.g. CELLBLAST, scmapcell). Define markersupervised(test, marker, method=...) whose signature drops label_train and instead accepts a per-cell-type marker list (differentially expressed markers from Seurat FindAllMarkers on training data, or canonical markers from PanglaoDB/CellMarker), routing marker-learning methods (e.g. SCSA, Garnett, SCINA). Encode a method-to-category routing table so each method name resolves to exactly one wrapper, then verify the source file parses in R and each wrapper returns a prediction vector without argument errors.

## Related tools

- **AutomaticCellTypeIdentification** (R package whose three wrapper interfaces (eagersupervised, lazysupervised, markersupervised) the function signatures are designed for) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **Seurat** (Used to derive cell-type specific markers (FindAllMarkers with labels as active.ident) that serve as the marker-list input to markersupervised wrappers)
- **caret (R package)** (Provides createDataPartition and createFolds to split count matrices into the training and testing datasets consumed by the eager/lazy wrappers)
- **SCSA** (Example marker-learning method dispatched through the markersupervised(test, marker, method='SCSA') signature)
- **CELLBLAST** (Example lazy-supervised method dispatched through lazysupervised(train, test, label_train, method='CELLBLAST')) — https://github.com/gao-lab/Cell_BLAST

## Examples

```
eagersupervised(train,test,label_train,method='Seurat'); markersupervised(test,marker,method='SCSA')
```

## Evaluation signals

- The R source file containing the three wrapper definitions parses and loads without error (e.g. via R CMD check or source())
- eagersupervised(train, test, label_train, method='Seurat') returns a prediction vector with no argument errors
- lazysupervised(train, test, label_train, method='CELLBLAST') returns a prediction vector with no argument errors
- markersupervised(test, marker, method='SCSA') returns a prediction vector without requiring label_train
- Every integrated method name resolves to exactly one wrapper category via the routing table (no method matches two wrappers)

## Limitations

- Python-based methods dispatched through the wrappers require the corresponding conda environment to be loaded at the beginning in R
- Input to the wrappers must be a count matrix with genes as rows and cells as columns; transposed or normalized-format mismatches will break argument expectations
- marker lists and training/testing data come from different resource centers (GEO/ArrayExpress/GSA for data, PanglaoDB/CellMarker/CancerSEA for markers), so column/identifier alignment must be handled before dispatch

## Evidence

- [readme] Now, three interface of eagersupervised`` lazysupervised`` markersupervised methods supports the available automatic methods.: [evidence span withheld — anchor status: not_found]
- [readme] markersupervised signature consumes marker list plus testing data: [evidence span withheld — anchor status: external_doc]
- [readme] The input of training and testing data is count matrix, the row is gene and the column is cell.: [evidence span withheld — anchor status: external_doc]
- [readme] Note: the conda environment of python-based method is needed to load at the beginning in R.: [evidence span withheld — anchor status: external_doc]
