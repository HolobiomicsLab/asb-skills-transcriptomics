---
name: percentile-threshold-suppression
description: Use when converting a LogNormalized single-cell gene expression matrix
  into a cell-type score matrix with MASI's gene2cell/PlinerScore, and low-level marker
  expression must be zeroed before summing weighted markers per cell type.
when_to_use_negative:
- When the query data contains cell types not present in the reference — MASI alone
  cannot annotate unseen cell types and requires combination with SCCAF
- When raw counts have not been LogNormalized (total-count division, 10,000 scale
  factor, log-transformation) — the transformation assumes normalized input
- When a model-based integration/annotation approach is required (e.g. scNym, scArches,
  Symphony pipelines) rather than a marker-based model-free method
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MASI
  role: Provides the gene2cell/PlinerScore function that performs TF-IDF normalization
    and percentile-threshold suppression before cell-type scoring
  repo: https://github.com/hayatlab/MASI
- name: Scanpy
  role: Used to load the processed LogNormalized expression matrix from the figshare
    deposit as an AnnData object
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:8f627b95d52ad3224ab01fd289b57c4677701c4040622c25668318291057b837
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MASI
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# percentile-threshold-suppression

## Summary

Within the MASI PlinerScore transformation, marker expression values below a percentile threshold (default 0.25) across all cells are suppressed to zero before the gene expression matrix is converted to a cell-type score matrix, reducing noise from low marker expression.

## When to use

Use when converting a LogNormalized single-cell gene expression matrix into a cell-type score matrix with MASI's gene2cell/PlinerScore, and low-level marker expression must be zeroed before summing weighted markers per cell type.

## When NOT to use

- When the query data contains cell types not present in the reference — MASI alone cannot annotate unseen cell types and requires combination with SCCAF
- When raw counts have not been LogNormalized (total-count division, 10,000 scale factor, log-transformation) — the transformation assumes normalized input
- When a model-based integration/annotation approach is required (e.g. scNym, scArches, Symphony pipelines) rather than a marker-based model-free method

## Inputs

- Processed LogNormalized gene expression matrix in AnnData format (from the figshare deposit or user data)
- Ranked cell-type marker dictionary (cell_markers) with linear weight discounting from 100% (1st marker) to 50% (Nth marker)
- Percentile threshold parameter (default 0.25 for PlinerScore)

## Outputs

- Cell-type score matrix (one row per cell, one column per reference cell type), e.g. exported to cell_type_scores.csv
- Per-cell assigned labels (scores/labels valid as inputs to masi.parallel with batch_size=50000)

## How to apply

Load the processed LogNormalized expression matrix (counts divided by cell total, scaled by 10,000, then log-transformed) as AnnData and the ranked cell-type marker list with linear weight discounting (100% for the 1st marker to 50% for the Nth marker). Call masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True), which applies TF-IDF normalization followed by suppressing expression values of each marker gene to zeros if they fall below the percentile threshold across all cells; the default PlinerScore threshold is the 0.25 percentile. After suppression, weighted marker expression is summed per cell type for every cell to produce raw scores and assigned labels. Note the article describes the threshold with a variable 'X-percentile' and a default of 0.25, but the applied value per analysis is not explicitly stated per dataset.

## Related tools

- **MASI** (Provides the gene2cell/PlinerScore function that performs TF-IDF normalization and percentile-threshold suppression before cell-type scoring) — https://github.com/hayatlab/MASI
- **Scanpy** (Used to load the processed LogNormalized expression matrix from the figshare deposit as an AnnData object)

## Examples

```
scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True)
```

## Evaluation signals

- Output cell_type_scores.csv exists with one row per cell and one column per reference cell type
- Suppressed values are exactly zero for marker expression below the 0.25 percentile across all cells
- Returned scores and labels are valid inputs to downstream parallel annotation (masi.parallel with batch_size=50000)
- Raw cell-type scores equal the sum of weighted marker expression per cell type after TF-IDF and thresholding

## Limitations

- PlinerScore threshold is described with both a variable 'X-percentile' and a default of 0.25; the applied value per analysis is not stated, a reproducibility gap
- No software version numbers are reported for tools used (Scanpy, MASI, etc.)
- Annotation accuracy via MASI is dependent on the quality of reference data; choice and resolution of the reference are critical

## Evidence

- [discussion] Suppressing marker expression values below the 0.25 percentile across all cells to zero before cell-type score conversion: "suppressing expression values of a marker gene to zeros if they are below the X-percentile of expression values across all cells"
- [discussion] PlinerScore default threshold: "The default value for PlinerScore threshold is 0.25 as the percentile threshold"
- [methods] gene2cell invocation: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [readme] README gene-to-score step: "##step 1 transform gene expression matrix to cell-type score matrix"
