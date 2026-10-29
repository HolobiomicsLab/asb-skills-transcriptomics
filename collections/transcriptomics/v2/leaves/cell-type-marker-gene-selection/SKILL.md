---
name: cell-type-marker-gene-selection
description: Use when you have a raw/normalized scRNA-seq count matrix paired with
  manually annotated cell-type labels and need a per-cell-type positive marker gene
  list to drive marker-based or marker-learning cell-type prediction.
when_to_use_negative:
- Input is already a curated marker gene table (e.g. downloaded from PanglaoDB or
  CellMarker) — pass it directly as the marker_list to markersupervised methods instead
  of re-deriving markers
- Your target prediction method is eager-supervised or lazy-supervised (e.g. Seurat,
  CELLBLAST, scmap-cell) since these take train/test count matrices and labels rather
  than a marker list
- Training cells lack manually annotated cell-type labels (cells with no cell type
  information are deleted during preprocessing)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: AutomaticCellTypeIdentification
  role: Provides the diffmarker wrapper and the markersupervised interface that consumes
    the derived marker list
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: Seurat
  role: 'Underlying engine: NormalizeData normalizes the count matrix and FindAllMarkers
    (with labels as active.ident, only.pos=TRUE) finds cell-type-specific markers'
- name: caret (R package)
  role: createDataPartition/createFolds used to split training data before marker
    identification in the cross-validation workflow
- name: SCSA
  role: markersupervised method that receives the derived marker table as its marker
    input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:7837cf505afb8f9fbe5b5459516959d31b0c74279a2e4759e4e9bba18f4aae63
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
  - R
  - SCSA
  - Seurat
  - caret (R package)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cell-type-marker-gene-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives cell-type-specific differential marker genes from a training scRNA-seq count matrix with manually annotated labels, producing a marker table that seeds marker-learning cell-type identification methods (e.g. SCSA) in the AutomaticCellTypeIdentification package.

## When to use

Use when you have a raw/normalized scRNA-seq count matrix paired with manually annotated cell-type labels and need a per-cell-type positive marker gene list to drive marker-based or marker-learning cell-type prediction.

## When NOT to use

- Input is already a curated marker gene table (e.g. downloaded from PanglaoDB or CellMarker) — pass it directly as the marker_list to markersupervised methods instead of re-deriving markers
- Your target prediction method is eager-supervised or lazy-supervised (e.g. Seurat, CELLBLAST, scmap-cell) since these take train/test count matrices and labels rather than a marker list
- Training cells lack manually annotated cell-type labels (cells with no cell type information are deleted during preprocessing)

## Inputs

- Training scRNA-seq count matrix (10x data, e.g. GSE109774 liver; genes as rows, cells as columns)
- Manually annotated cell-type labels for the training cells (used as 'active.ident')
- Optional: pre-existing marker lists from marker resource centers (PanglaoDB, CellMarker, CancerSEA) as an alternative marker source

## Outputs

- Differential-marker table (e.g. marker_5.csv) with per-cell-type columns: cell type/cluster, gene, average log fold-change, and associated statistics
- Top-N positive marker gene list per cell type (top 5 per type in the demo) used as marker_list for markersupervised methods

## How to apply

Load the training count matrix (genes as rows, cells as columns) and the manually annotated labels into R. Within the diffmarker wrapper from AutomaticCellTypeIdentification, the count matrix is normalized with Seurat's NormalizeData, then differential markers are computed with Seurat's FindAllMarkers using the annotated labels as 'active.ident', only.pos = TRUE, and default parameters otherwise — e.g. markers = diffmarker(train, label_train, only.pos=T). In the demo workflow (GSE109774 liver 10x data), the top 5 positive marker genes per cell type are retained and written to marker_5.csv, which is then passed as the marker_list input to markersupervised methods such as SCSA (markersupervised(test, marker, method='SCSA')). Upstream QC filtering should first remove genes not expressed in any cells, cells with fewer than 200 expressed genes, cells exceeding twice the standard deviation of mean expressed genes, cells with >20% mitochondrial expression, and cells lacking cell type information.

## Related tools

- **AutomaticCellTypeIdentification** (Provides the diffmarker wrapper and the markersupervised interface that consumes the derived marker list) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **Seurat** (Underlying engine: NormalizeData normalizes the count matrix and FindAllMarkers (with labels as active.ident, only.pos=TRUE) finds cell-type-specific markers)
- **caret (R package)** (createDataPartition/createFolds used to split training data before marker identification in the cross-validation workflow)
- **SCSA** (markersupervised method that receives the derived marker table as its marker input)

## Examples

```
markers <- diffmarker(train, label_train, only.pos=T); markersupervised(test, markers, method='SCSA')
```

## Evaluation signals

- marker_5.csv exists and is non-empty after the workflow run
- marker_5.csv contains the declared differential-marker columns (cell type/cluster, gene, average log fold-change, associated statistics) for every cell type present in the training labels
- Every retained marker has a positive log fold-change (only.pos=TRUE ensures no negative markers)
- Marker table contains markers for all cell types, with up to 5 positive markers per type in the demo configuration
- Downstream markersupervised call (e.g. SCSA) accepts the marker table as marker_list without schema errors

## Limitations

- Marker identification depends on the quality of the manually annotated training labels; cells without cell type information must be removed beforehand
- Defaults rely on Seurat FindAllMarkers parameters — other parameters are left at defaults in the demo, and only positive markers are retained
- Preprocessing thresholds (min 200 genes/cell, >2 SD mean-gene cutoff, >20% mitochondrial expression removal) may not suit all tissues/datasets
- Python-based markersupervised methods (e.g. SCSA) require the corresponding conda environment to be loaded before running in R

## Evidence

- [readme] markersupervised interface consumes a marker list: [evidence span withheld — anchor status: not_found]
- [readme] Input data format for the package: [evidence span withheld — anchor status: external_doc]
