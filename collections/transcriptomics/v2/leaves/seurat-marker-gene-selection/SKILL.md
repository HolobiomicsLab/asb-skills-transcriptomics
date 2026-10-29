---
name: seurat-marker-gene-selection
description: Use when you have a preprocessed count matrix plus manually annotated
  cell-type labels and need top positive marker genes per cell type to run marker-supervised
  methods (SCSA, MarkerCount, SCINA) or evaluate marker-based identification.
when_to_use_negative:
- No manually annotated labels exist for the training cells — FindAllMarkers requires
  labels set as 'active.ident'; use eager- or lazy-learning methods instead.
- Markers are already available from marker resource centers (PanglaoDB, CellMarker,
  CancerSEA) and no dataset-derived markers are needed.
- The goal is prediction from reference data rather than marker-based identification
  — use eagersupervised or lazysupervised interfaces of AutomaticCellTypeIdentification
  instead.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Provides NormalizeData for count-matrix normalization and FindAllMarkers for
    identifying cell-type-specific positive marker genes using the manual labels as
    'active.ident'
- name: AutomaticCellTypeIdentification
  role: R wrapper whose diffmarker function encapsulates the Seurat normalization
    plus FindAllMarkers workflow and whose markersupervised interface consumes the
    resulting marker list (e.g. SCSA)
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: caret (R package)
  role: createFolds is used to split datasets into five folds for cross-validation
    in the evaluation workflows that consume these markers
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:dccb0438902cbcc89fba386a32693688bf43e1d09156ab9155d8f1177b443fc6
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
  - CellFishing.jl
  - Garnett
  - R
  - SciBet
  - Seurat
  - caret
  - caret (R package)
  - scID
  - scLearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# Seurat marker gene selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives cell-type-specific marker genes from a count matrix with manually annotated labels using the Seurat pipeline (NormalizeData followed by FindAllMarkers with only.pos=TRUE), seeding marker-learning cell-type identification methods.

## When to use

Use when you have a preprocessed count matrix plus manually annotated cell-type labels and need top positive marker genes per cell type to run marker-supervised methods (SCSA, MarkerCount, SCINA) or evaluate marker-based identification.

## When NOT to use

- No manually annotated labels exist for the training cells — FindAllMarkers requires labels set as 'active.ident'; use eager- or lazy-learning methods instead.
- Markers are already available from marker resource centers (PanglaoDB, CellMarker, CancerSEA) and no dataset-derived markers are needed.
- The goal is prediction from reference data rather than marker-based identification — use eagersupervised or lazysupervised interfaces of AutomaticCellTypeIdentification instead.

## Inputs

- Raw or filtered count matrix (rows = genes, columns = cells), e.g. 10x GSE109774 liver or PBMC SCP424 data
- Manually annotated cell-type labels, one label per cell, used as Seurat 'active.ident'

## Outputs

- Marker table (e.g. marker_5.csv) with columns for cell type/cluster, gene, average log fold-change and associated statistics
- Top-N positive marker gene list per cell type (e.g. top 5 and 15 genes) used as marker_list for markersupervised prediction

## How to apply

Load the count matrix (genes as rows, cells as columns) and the manually annotated cell-type labels into R, then normalize the count matrix with the Seurat NormalizeData function (in the AutomaticCellTypeIdentification package this is wrapped by diffmarker). Set the manually annotated labels as the Seurat 'active.ident' and run FindAllMarkers with only.pos = TRUE, leaving all other parameters at their defaults. Retain the top 5 positive marker genes per cell type (the PBMC cross-platform evaluation also used top 5 and 15 genes supplemented by canonical PBMC markers) to build the marker list. Pass the resulting marker table as the marker_list input to markersupervised methods via the AutomaticCellTypeIdentification package, e.g. markersupervised(test, marker, method='SCSA').

## Related tools

- **Seurat** (Provides NormalizeData for count-matrix normalization and FindAllMarkers for identifying cell-type-specific positive marker genes using the manual labels as 'active.ident')
- **AutomaticCellTypeIdentification** (R wrapper whose diffmarker function encapsulates the Seurat normalization plus FindAllMarkers workflow and whose markersupervised interface consumes the resulting marker list (e.g. SCSA)) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **caret (R package)** (createFolds is used to split datasets into five folds for cross-validation in the evaluation workflows that consume these markers)

## Examples

```
markers = diffmarker(train, label_train, only.pos=T); markersupervised(test, marker = markers, method='SCSA')
```

## Evaluation signals

- marker_5.csv exists, is non-empty, and contains the declared differential-marker columns (cell type/cluster, gene, average log fold-change, associated statistics) for every cell type in the training labels
- Every marker gene is positive (only.pos = TRUE), i.e. no negative log-fold-change entries are retained
- Exactly the top 5 positive marker genes per cell type are retained when seeding markersupervised predictions
- Downstream markersupervised calls (e.g. SCSA) run without missing-gene errors, confirming the marker list covers the cell types in the test data

## Limitations

- Markers are dataset-specific: FindAllMarkers is run on the training count matrix with manual labels, so marker sets may not transfer across platforms (the PBMC cross-platform evaluation showed marker-learning methods slightly trail top eager methods in accuracy and F1-score)
- Requires high-quality manual annotations; cells with no cell type information must be deleted beforehand, and cells with fewer than 200 expressed genes or more than 20% mitochondrial gene expression were filtered in the published workflows
- The count matrix must be normalized (NormalizeData) before marker selection, as done inside the diffmarker wrapper

## Evidence
