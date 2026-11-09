---
name: cell-type-score-computation
description: Use when a scRNA-seq analysis requires scoring cells against known cell-type
  markers (e.g., producing per-cell scores/labels for MASI-based annotation or for
  pipeline variants in integration benchmarks).
when_to_use_negative:
- Query data contains unseen cell types not present in the reference — MASI scores
  alone cannot annotate those; combine MASI with SCCAF to identify subtypes.
- Reference data quality or resolution does not match the query — MASI annotation
  accuracy depends on reference quality, so poor reference choice invalidates the
  scores.
- Input is raw counts without LogNormalization (scale factor 10,000 + log transform)
  — the PlinerScore transformation expects normalized expression.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MASI
  role: Provides masi.gene2cell, which converts the gene expression matrix to a cell-type
    score matrix via PlinerScore (TF-IDF + 0.25-percentile thresholding) with weighted
    marker summation
  repo: github.com/hayatlab/MASI
- name: Scanpy
  role: Loads the LogNormalized expression matrix from the figshare deposit as an
    AnnData object for gene2cell input
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:4970fa3a48e79c78f7c7f4d6d09a22a42336aecc5c5fa79e4c693551939d4cc6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MASI
  - Scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cell-type-score-computation

## Summary

Converts a LogNormalized gene expression matrix into a cell-type score matrix using MASI's gene2cell, applying the PlinerScore transformation (TF-IDF normalization plus 0.25-percentile thresholding) and weighted summation of ranked marker genes per cell type.

## When to use

Use when a scRNA-seq analysis requires scoring cells against known cell-type markers (e.g., producing per-cell scores/labels for MASI-based annotation or for pipeline variants in integration benchmarks).

## When NOT to use

- Query data contains unseen cell types not present in the reference — MASI scores alone cannot annotate those; combine MASI with SCCAF to identify subtypes.
- Reference data quality or resolution does not match the query — MASI annotation accuracy depends on reference quality, so poor reference choice invalidates the scores.
- Input is raw counts without LogNormalization (scale factor 10,000 + log transform) — the PlinerScore transformation expects normalized expression.

## Inputs

- Processed LogNormalized gene expression matrix in AnnData (ad) format (counts / cell total × 10,000, log-transformed)
- Ranked cell-type marker list/dict (cell_markers) from upstream marker identification, with linear weight discounting from 100% to 50%

## Outputs

- Cell-type score matrix (one row per cell, one column per reference cell type)
- Assigned cell-type labels per cell
- Optionally cell_type_scores.csv per dataset

## How to apply

Load the LogNormalized expression matrix (counts divided by cell total, scaled by 10,000, log-transformed) as an AnnData object, and load the ranked cell-type marker list with linear weight discounting from 100% for the 1st marker to 50% for the Nth. Call masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True): this applies the PlinerScore transformation — TF-IDF normalization followed by suppressing marker expression values below the 0.25 percentile threshold across all cells to zero — then sums weighted marker expression per cell type for every cell to obtain raw scores and assigned labels. No highly variable gene selection is needed because only identified marker genes are used. The resulting score matrix feeds downstream annotation via masi.parallel(scores=scores, labels=labels, batch_size=50000) and can be exported as cell_type_scores.csv.

## Related tools

- **MASI** (Provides masi.gene2cell, which converts the gene expression matrix to a cell-type score matrix via PlinerScore (TF-IDF + 0.25-percentile thresholding) with weighted marker summation) — github.com/hayatlab/MASI
- **Scanpy** (Loads the LogNormalized expression matrix from the figshare deposit as an AnnData object for gene2cell input)

## Examples

```
scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True)
```

## Evaluation signals

- Output score matrix has one row per cell and one column per reference cell type
- Returned scores/labels are valid inputs for downstream parallel annotation via masi.parallel with batch_size=50000
- PlinerScore threshold applied is 0.25 as the percentile threshold across all cells
- In benchmark context, pipeline #14 (PlinerScore) shows substantially better integration scores than pipelines #1–#7 by paired t-tests (p-value < 0.05)

## Limitations

- PlinerScore threshold is described as both a variable 'X-percentile' and a default of 0.25; the applied value per analysis is not always stated (reproducibility gap).
- Marker expression values below the 0.25 percentile across all cells are suppressed to zero, which can drop lowly expressed markers.
- Scores depend on the ranked marker list quality; MASI alone cannot annotate unseen cell types absent from the reference.

## Evidence

- [full_text] Pipeline #14 (PlinerScore) applies an additional transformation and thresholding to the data before converting the gene expression matrix to a cell-type score matrix by summing up all marker genes for a given cell type.: [evidence span withheld — anchor status: not_found]
- [methods] scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True): "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [discussion] The default value for PlinerScore threshold is 0.25 as the percentile threshold: "The default value for PlinerScore threshold is 0.25 as the percentile threshold"
- [discussion] the 1st marker in this ranked list will contribute 100% of its expression to the cell-type score of A, while the Nth marker only contributes 50%: "the 1st marker in this ranked list will contribute 100% of its expression to the cell-type score of A, while the Nth marker only contributes 50%"
- [discussion] Raw gene expression counts data were ‘LogNormalized’, which divides the total count in that cell and multiplies it by a scale factor of 10,000: "Raw gene expression counts data were ‘LogNormalized’, which divides the total count in that cell and multiplies it by a scale factor of 10,000"
