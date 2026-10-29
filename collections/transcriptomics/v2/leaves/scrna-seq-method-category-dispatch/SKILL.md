---
name: scrna-seq-method-category-dispatch
description: Use when calling AutomaticCellTypeIdentification on count-matrix inputs
  and you must route a named method (e.g. Seurat, scmap-cell, SCSA) to the correct
  wrapper with the correct argument order.
when_to_use_negative:
- No training labels AND no marker lists are available (method is unsupervised; these
  wrappers only cover supervised categories)
- Input is already a marker list but the target method is eager or lazy supervised
  — those wrappers require the full training expression matrix and labels, not markers
- The goal is to benchmark method accuracy (accuracy, F1-score, unlabeling rate) rather
  than to route and execute a single named method
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0091
tools:
- name: AutomaticCellTypeIdentification
  role: Provides the three dispatch wrappers (eagersupervised, lazysupervised, markersupervised)
    that route named methods to their implementations
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: Seurat
  role: Pipeline used to find cell-type-specific markers (FindAllMarkers with manually
    annotated labels as active.ident) that feed the markersupervised wrapper
- name: scmap-cell
  role: Instance-based lazy-supervised method routed through lazysupervised(train,
    test, label_train, method='scmapcell')
  repo: https://github.com/hemberg-lab/scmap
- name: CELLBLAST
  role: Instance-based lazy-supervised method routed through lazysupervised(train,
    test, label_train, method='CELLBLAST')
  repo: https://github.com/gao-lab/Cell_BLAST
- name: ACTINN
  role: Eager-supervised method routed through eagersupervised with training matrix,
    labels, and testing matrix
  repo: https://github.com/mafeiyang/ACTINN
- name: CaSTLe
  role: Eager-supervised method routed through eagersupervised with training matrix,
    labels, and testing matrix
  repo: https://github.com/yuvallb/CaSTLe
- name: SciBet
  role: Eager-supervised method routed through eagersupervised with training matrix,
    labels, and testing matrix
  repo: https://github.com/zwj-tina/scibetR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:4f3b8ce42c1113e9de4b6092414ebe04e21b05c1ebdf2d5b4b1d0ac230fbba56
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
  - R
  - SciBet
  - Seurat
  - scmap-cell
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# scrna-seq-method-category-dispatch

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Dispatches scRNA-seq cell-type identification methods to one of three wrapper interfaces in the AutomaticCellTypeIdentification R package — eagersupervised, lazysupervised, and markersupervised — each with a distinct argument signature matched to the method's learning category.

## When to use

Use when calling AutomaticCellTypeIdentification on count-matrix inputs and you must route a named method (e.g. Seurat, scmap-cell, SCSA) to the correct wrapper with the correct argument order.

## When NOT to use

- No training labels AND no marker lists are available (method is unsupervised; these wrappers only cover supervised categories)
- Input is already a marker list but the target method is eager or lazy supervised — those wrappers require the full training expression matrix and labels, not markers
- The goal is to benchmark method accuracy (accuracy, F1-score, unlabeling rate) rather than to route and execute a single named method

## Inputs

- Training count matrix (rows = genes, columns = cells)
- Testing count matrix (rows = genes, columns = cells)
- Manually annotated cell-type labels for the training data (label_train)
- Per-cell-type marker gene list (top 5 or 15 DE markers per type, or canonical markers from PanglaoDB/CellMarker/CancerSEA) for markersupervised methods
- Named method string (e.g. 'Seurat', 'CELLBLAST', 'SCSA')

## Outputs

- Predicted cell-type label vector for the testing cells
- Method-to-category routing resolution (eager / lazy / marker)

## How to apply

Read the training and testing count matrices (genes as rows, cells as columns) with manually annotated labels. Build the method-to-category routing table: eager-supervised methods (ACTINN, CaSTLe, CHETAH, clustifyr, Garnett, Markercount, MARS, scClassifR, scHPL, SciBet, scID, scLearn, scmapcluster, scPred, scVI, Seurat, SingleCellNet, SingleR) and lazy-supervised instance-based methods (CELLBLAST, scmap-cell) both take (train, test, label_train), while marker-supervised methods (scTyper, Markercount, SCSA, DigitalCellSorter, SCINA) take only (test, marker) because they consume per-cell-type marker gene lists (e.g. top 5 or 15 differentially expressed markers per type derived with the Seurat FindAllMarkers pipeline) instead of expression matrices. Dispatch each named method to exactly one wrapper with the correct argument order. For python-based methods (e.g. SCSA), load the required conda environment at the beginning in R. Wrap the dispatch functions in a single self-contained R source file that loads without error.

## Related tools

- **AutomaticCellTypeIdentification** (Provides the three dispatch wrappers (eagersupervised, lazysupervised, markersupervised) that route named methods to their implementations) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **Seurat** (Pipeline used to find cell-type-specific markers (FindAllMarkers with manually annotated labels as active.ident) that feed the markersupervised wrapper)
- **scmap-cell** (Instance-based lazy-supervised method routed through lazysupervised(train, test, label_train, method='scmapcell')) — https://github.com/hemberg-lab/scmap
- **CELLBLAST** (Instance-based lazy-supervised method routed through lazysupervised(train, test, label_train, method='CELLBLAST')) — https://github.com/gao-lab/Cell_BLAST
- **ACTINN** (Eager-supervised method routed through eagersupervised with training matrix, labels, and testing matrix) — https://github.com/mafeiyang/ACTINN
- **CaSTLe** (Eager-supervised method routed through eagersupervised with training matrix, labels, and testing matrix) — https://github.com/yuvallb/CaSTLe
- **SciBet** (Eager-supervised method routed through eagersupervised with training matrix, labels, and testing matrix) — https://github.com/zwj-tina/scibetR

## Examples

```
eagersupervised(train,test,label_train,method='Seurat'); markersupervised(test,marker,method='SCSA')
```

## Evaluation signals

- The self-contained R dispatch source file parses and loads without error
- eagersupervised(train, test, label_train, method='Seurat') returns a prediction vector for the testing cells without argument errors
- lazysupervised accepts the same (train, test, label_train) signature and routes CELLBLAST and scmapcell correctly
- markersupervised(test, marker, method='SCSA') runs with only a marker list plus testing matrix and returns predictions
- Every integrated method resolves to exactly one wrapper category (eager, lazy, or marker) with the correct argument order

## Limitations

- Python-based methods require the matching conda environment to be loaded at the beginning in R; otherwise the call fails
- Some methods (e.g. Markercount) appear in more than one category, so the routing table must encode the intended mode per call
- Wrapper installation pulls dependencies listed in the package Description file, which can be heavy since it wraps many published methods

## Evidence

- [readme] Now, three interface of eagersupervised`` lazysupervised`` markersupervised methods supports the available automatic methods.: [evidence span withheld — anchor status: not_found]
- [readme] lazysupervised methods include CELLBLAST and scmapcell.: [evidence span withheld — anchor status: not_found]
- [readme] markersupervised methods include scTyper, Markercount, SCSA, DigitalCellSorter and SCINA.: [evidence span withheld — anchor status: not_found]
- [readme] The input of training and testing data is count matrix, the row is gene and the column is cell.: [evidence span withheld — anchor status: external_doc]
