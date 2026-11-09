---
name: weighted-marker-aggregation
description: Use when a query scRNA-seq dataset in AnnData format must be scored against
  a ranked cell-type marker list from a reference (e.g., MASI reference data) to produce
  per-cell cell-type scores and labels for downstream annotation.
when_to_use_negative:
- The goal is annotation of cell types not present in the reference — MASI cannot
  label unseen cell types without combining with SCCAF
- Only a subset of genes beyond the identified markers is required — MASI skips highly
  variable gene selection because only identified marker genes are used
- The input matrix is raw counts rather than LogNormalized data (counts divided by
  cell total, scaled by 10,000, log-transformed)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MASI
  role: Provides masi.gene2cell, which performs the PlinerScore transformation (TF-IDF
    normalization, 0.25 percentile thresholding) and weighted summation of marker
    expression into a cell-type score matrix
  repo: https://github.com/hayatlab/MASI
- name: Scanpy
  role: Used to load the processed LogNormalized expression matrix into the AnnData
    (ad) object passed to masi.gene2cell
- name: MACA
  role: Consumes the gene2cell scores/labels downstream to transfer cell-type labels
    to query data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:b54ed805ecf8e516f5a57c9d31756aa6f4a290cf5d4068cb621e69a1ca7aae6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MACA
  - MASI
  - Scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# weighted-marker-aggregation

## Summary

Converts a LogNormalized gene expression matrix into a cell-type score matrix by summing rank-weighted marker gene expression per cell type, as implemented in MASI's PlinerScore transformation (masi.gene2cell).

## When to use

Use when a query scRNA-seq dataset in AnnData format must be scored against a ranked cell-type marker list from a reference (e.g., MASI reference data) to produce per-cell cell-type scores and labels for downstream annotation.

## When NOT to use

- The goal is annotation of cell types not present in the reference — MASI cannot label unseen cell types without combining with SCCAF
- Only a subset of genes beyond the identified markers is required — MASI skips highly variable gene selection because only identified marker genes are used
- The input matrix is raw counts rather than LogNormalized data (counts divided by cell total, scaled by 10,000, log-transformed)

## Inputs

- Processed LogNormalized single-cell expression matrix as AnnData (counts/total x 10,000, log-transformed), e.g., from the MASI figshare deposit
- Ranked cell-type marker dictionary (cell_markers) from upstream marker identification with linear weight discounting from 100% (1st marker) to 50% (Nth marker)

## Outputs

- Per-dataset cell-type score matrix (one row per cell, one column per reference cell type), written to cell_type_scores.csv
- Per-cell raw scores and cell-type labels (scores, labels) suitable as input to masi.parallel with batch_size=50000

## How to apply

Load the processed LogNormalized expression matrix (counts divided by cell total, scaled by 10,000, log-transformed) from the figshare deposit using Scanpy/AnnData, along with the ranked cell-type marker list produced by robust rank aggregation of the top 20 markers per DE test. Apply linear weight discounting so the 1st marker contributes 100% of its expression to the cell-type score while the Nth marker contributes only 50%. Call masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True), which applies the PlinerScore transformation: TF-IDF normalization followed by suppressing marker expression values below the 0.25 percentile threshold across all cells to zero. Sum the weighted marker expression per cell type for every cell to obtain raw scores and labels; the returned scores/labels feed masi.parallel (batch_size=50000) for downstream annotation. Rationale: this marker-based, model-free scoring avoids training large supervised models while remaining competitive, and only marker genes are used so highly variable gene selection is skipped.

## Related tools

- **MASI** (Provides masi.gene2cell, which performs the PlinerScore transformation (TF-IDF normalization, 0.25 percentile thresholding) and weighted summation of marker expression into a cell-type score matrix) — https://github.com/hayatlab/MASI
- **Scanpy** (Used to load the processed LogNormalized expression matrix into the AnnData (ad) object passed to masi.gene2cell)
- **MACA** (Consumes the gene2cell scores/labels downstream to transfer cell-type labels to query data)

## Examples

```
scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True)
```

## Evaluation signals

- Output cell_type_scores.csv exists and has one row per cell and one column per reference cell type
- Returned scores and labels are valid inputs to downstream parallel annotation (masi.parallel with batch_size=50000)
- Marker expression values below the 0.25 percentile across all cells are suppressed to zero before score summation
- Weights follow the linear discount: 1st marker contributes 100% of its expression, Nth marker contributes 50%
- Input matrix is LogNormalized (divided by cell total, scaled by 10,000, log-transformed) before gene2cell is invoked

## Limitations

- Accurate annotation via MASI is dependent on the quality of reference data; the choice and resolution of the reference are critical to downstream analysis
- MASI alone cannot annotate unseen cell types not present in the reference; requires combining MASI with SCCAF
- Reproducibility gap: the PlinerScore threshold is described with both a variable 'X-percentile' and a default of 0.25; the applied value per analysis is not stated
- A major update of MASI (MASIv2) extends the approach to multi-modal single-cell and spatial omics data, so the original gene2cell path may not cover spatial contexts

## Evidence

- [results] Pipeline #14 (PlinerScore) applies an additional transformation and thresholding to the data before converting the gene expression matrix to a cell-type score matrix: [evidence span withheld — anchor status: not_found]
- [discussion] the 1st marker in this ranked list will contribute 100% of its expression to the cell-type score of A, while the Nth marker only contributes 50%: "the 1st marker in this ranked list will contribute 100% of its expression to the cell-type score of A, while the Nth marker only contributes 50%"
- [discussion] suppressing expression values of a marker gene to zeros if they are below the X-percentile of expression values across all cells: "suppressing expression values of a marker gene to zeros if they are below the X-percentile of expression values across all cells"
- [methods] scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True): "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [readme] step 1 transform gene expression matrix to cell-type score matrix: "##step 1 transform gene expression matrix to cell-type score matrix"
