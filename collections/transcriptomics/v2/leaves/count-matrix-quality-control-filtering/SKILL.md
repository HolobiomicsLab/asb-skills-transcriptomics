---
name: count-matrix-quality-control-filtering
description: Use when a raw scRNA-seq count matrix (genes × cells, e.g. from GEO/ArrayExpress/GSA)
  is about to be used as a training or testing input for cell-type identification
  and needs QC, normalization and label curation first.
when_to_use_negative:
- The count matrix has already been QC-filtered and normalized for a downstream method,
  so re-applying the 200-gene / 2×SD / 20% mitochondrial cutoffs would double-filter
- You are running marker-based methods (e.g. markersupervised SCSA/SCSA-style interface)
  on an unlabeled test set where no cell-type labels exist to delete and only the
  marker list is the input
- Your dataset legitimately contains very low-gene-count or high-mitochondrial cells
  of interest, since the paper's thresholds are fixed generic cutoffs, not dataset-specific
  optima
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Provides the NormalizeData normalization and the QC pipeline (filter genes/cells,
    mitochondrial percentage, FindAllMarkers with labels as 'active.ident') that defines
    this skill's thresholds
- name: AutomaticCellTypeIdentification
  role: Consumes the filtered count matrix (genes as rows, cells as columns) as the
    train/test input for eagersupervised, lazysupervised and markersupervised cell-type
    identification
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: caret (R package)
  role: Splits or downsamples the filtered cells via createDataPartition/createFolds
    after QC is complete
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:3af8dfabdb27ff81df2300bc70e9c61f87fbe3d7badc6479a7d0d61d0cff759c
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# count-matrix-quality-control-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the paper's Seurat-compatible quality-control filters to raw gene-by-cell count matrices — removing unexpressed genes, low-complexity cells, high-mitochondrial-fraction cells and unlabeled cells — so that datasets are clean before training/testing automatic cell-type identification methods.

## When to use

Use when a raw scRNA-seq count matrix (genes × cells, e.g. from GEO/ArrayExpress/GSA) is about to be used as a training or testing input for cell-type identification and needs QC, normalization and label curation first.

## When NOT to use

- The count matrix has already been QC-filtered and normalized for a downstream method, so re-applying the 200-gene / 2×SD / 20% mitochondrial cutoffs would double-filter
- You are running marker-based methods (e.g. markersupervised SCSA/SCSA-style interface) on an unlabeled test set where no cell-type labels exist to delete and only the marker list is the input
- Your dataset legitimately contains very low-gene-count or high-mitochondrial cells of interest, since the paper's thresholds are fixed generic cutoffs, not dataset-specific optima

## Inputs

- Raw count matrix with genes as rows and cells as columns
- Manually annotated cell-type labels for each cell (e.g. from GEO accession GSE131907 or GSE116470)
- Mitochondrial gene annotation to compute per-cell mitochondrial expression fraction

## Outputs

- Filtered count matrix (unexpressed genes and low-quality cells removed)
- Normalized count matrix produced with the Seurat NormalizeData function
- Curated cell-to-cell-type label table with unlabeled cells deleted

## How to apply

Start from the raw count matrix with manually annotated labels, oriented with genes as rows and cells as columns as required by the AutomaticCellTypeIdentification package. Apply the paper's four Seurat-compatible cell filters in order: remove genes not expressed in any cells; remove cells with fewer than 200 expressed genes; remove cells whose expressed-gene count exceeds twice the standard deviation of the mean; and remove cells with more than 20% mitochondrial gene expression. When annotations exist (as in the GSE131907 lung and GSE116470 brain workflows), also delete cells lacking cell-type information, since supervised training requires labels. Normalize the filtered count matrix with the Seurat NormalizeData function before downstream steps such as marker finding (FindAllMarkers) or caret-based splitting via createDataPartition/createFolds. The rationale is that these thresholds standardize inputs across the 24+ wrapped methods so accuracy, F1-score, unlabeling-rate and runtime benchmarks are comparable.

## Related tools

- **Seurat** (Provides the NormalizeData normalization and the QC pipeline (filter genes/cells, mitochondrial percentage, FindAllMarkers with labels as 'active.ident') that defines this skill's thresholds)
- **AutomaticCellTypeIdentification** (Consumes the filtered count matrix (genes as rows, cells as columns) as the train/test input for eagersupervised, lazysupervised and markersupervised cell-type identification) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **caret (R package)** (Splits or downsamples the filtered cells via createDataPartition/createFolds after QC is complete)

## Examples

```
norm <- NormalizeData(counts)  # counts: row=gene, column=cell, after removing genes/cells failing the paper's QC filters
```

## Evaluation signals

- No row (gene) of the filtered matrix has zero total counts across all cells
- Every retained cell has at least 200 expressed genes and its expressed-gene count does not exceed twice the standard deviation of the mean
- Maximum per-cell mitochondrial gene expression fraction is at most 20%
- Every retained cell carries a non-empty cell-type label when the labeled-workflow variant is used
- The filtered matrix retains genes-as-rows / cells-as-columns orientation, the input format expected by AutomaticCellTypeIdentification's eagersupervised/lazysupervised interfaces

## Limitations

- The 200-gene, 2×SD, and 20% mitochondrial cutoffs are generic thresholds from the paper's preprocessing; they were not tuned per tissue and may over- or under-filter unusual datasets
- Deleting cells without cell-type information restricts the data to annotated populations, which is only valid for supervised training/testing designs like the GSE131907 and GSE116470 workflows
- The article provides no discussion-section caveats (no changelog/discussion text available), so edge-case guidance beyond the stated filters is not documented

## Evidence

- [readme] The input of training and testing data is count matrix, the row is gene and the column is cell.: [evidence span withheld — anchor status: external_doc]
