---
name: marker-table-validation
description: Use when a marker_5.csv or similar differential-marker table has just
  been generated from a training count matrix (e.g. GSE109774 liver 10x data) and
  must be verified before being passed to markersupervised methods.
when_to_use_negative:
- The input is already a curated marker list from a marker resource center (PanglaoDB,
  CellMarker, CancerSEA) rather than a diffmarker()-derived table — it may not carry
  the log fold-change/statistics schema to validate.
- The pipeline uses eager- or lazy-supervised methods (eagersupervised/lazysupervised),
  which take count matrices and labels directly rather than a marker_list.
- The marker table is empty for some cell types due to failed FindAllMarkers runs
  — validation should fail rather than be forced through.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutomaticCellTypeIdentification
  role: Provides diffmarker() and markersupervised(); diffmarker wraps Seurat to derive
    the marker table being validated
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: Seurat
  role: Underlying engine for NormalizeData and FindAllMarkers used inside diffmarker
    with active.ident set to the manual labels
- name: R
  role: Execution environment for the diffmarker and validation workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:64c918683f721a8118a8fe6167975d7c79ceb51ee51ba072a9ed155ff0adecd1
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
  - Seurat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# marker-table-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates the differential-marker table (e.g. marker_5.csv) produced by the AutomaticCellTypeIdentification diffmarker() workflow, which seeds marker-learning cell-type prediction methods such as SCSA. Ensures the marker list is complete, non-empty, and schema-conformant before downstream prediction.

## When to use

Use when a marker_5.csv or similar differential-marker table has just been generated from a training count matrix (e.g. GSE109774 liver 10x data) and must be verified before being passed to markersupervised methods.

## When NOT to use

- The input is already a curated marker list from a marker resource center (PanglaoDB, CellMarker, CancerSEA) rather than a diffmarker()-derived table — it may not carry the log fold-change/statistics schema to validate.
- The pipeline uses eager- or lazy-supervised methods (eagersupervised/lazysupervised), which take count matrices and labels directly rather than a marker_list.
- The marker table is empty for some cell types due to failed FindAllMarkers runs — validation should fail rather than be forced through.

## Inputs

- Differential-marker table (marker_5.csv) with columns: cell type/cluster, gene, average log fold-change, associated statistics
- Training count matrix (gene rows x cell columns, e.g. preprocessed GSE109774 liver 10x data)
- Manually annotated cell-type labels for the training cells

## Outputs

- Validation verdict: marker file exists, is non-empty, and contains required columns for every cell type
- Per-cell-type marker set (top 5 positive markers) ready as marker_list input to markersupervised()

## How to apply

After running markers = diffmarker(train, label_train, only.pos=T), check that the output marker file (marker_5.csv) exists and is non-empty. Verify the table contains the declared differential-marker columns for every cell type in the training labels: cell type/cluster, gene, average log fold-change, and associated statistics. Confirm the table retains the top 5 positive marker genes per cell type, since only.pos = TRUE restricts output to positive markers and the demo workflow seeds marker-learning prediction (e.g. SCSA via markersupervised) with this marker_list. The markers are derived by wrapping Seurat's NormalizeData and FindAllMarkers, with the manually annotated labels set as 'active.ident' and all other FindAllMarkers parameters left at their defaults.

## Related tools

- **AutomaticCellTypeIdentification** (Provides diffmarker() and markersupervised(); diffmarker wraps Seurat to derive the marker table being validated) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **Seurat** (Underlying engine for NormalizeData and FindAllMarkers used inside diffmarker with active.ident set to the manual labels)
- **R** (Execution environment for the diffmarker and validation workflow)

## Examples

```
markers <- diffmarker(train, label_train, only.pos=T); markersupervised(test, marker = markers, method='SCSA')
```

## Evaluation signals

- marker_5.csv exists on disk and is non-empty
- Header contains the declared columns: cell type/cluster, gene, average log fold-change, and associated statistics
- Every cell type present in the training labels has at least one marker row in the table
- Each cell type contributes the expected top 5 positive markers (only.pos = TRUE was honored — no negative fold-change genes present)
- marker table can be passed without error as the marker argument to markersupervised(test, marker, method='SCSA')

## Limitations

- Validation confirms schema and coverage, not biological correctness — FindAllMarkers runs at default parameters and the review does not benchmark marker quality itself.
- Only.pos = TRUE means the table contains no negative markers; cell types lacking strong positive markers may be under-represented.
- Python-based markersupervised methods (e.g. SCSA) require the conda environment to be loaded at the beginning in R, which is outside the validation step.

## Evidence

- [methods] markers derived by applying diffmarker directly to the training count matrix and labels with only.pos=T: [evidence span withheld — anchor status: not_found]
- [results] Validation criterion from the task card: [evidence span withheld — anchor status: not_found]
- [readme] markersupervised interface from README: [evidence span withheld — anchor status: external_doc]
