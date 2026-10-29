---
name: single-cell-annotation-scoring
description: Use when you have reference data with cell-type labels ('cell_type' column
  in AnnData) and need to annotate a query single-cell dataset by scoring cells against
  ranked marker genes rather than training a model.
when_to_use_negative:
- When the query dataset contains cell types absent from the reference — MASI alone
  cannot annotate unseen types; combine with SCCAF for subtype identification
- When reference data quality or resolution is poor — MASI annotation accuracy is
  dependent on the quality of reference data, so a resolution-matched reference is
  required
- When inputs are not raw counts suitable for LogNormalization (scale factor 10,000
  followed by log-transformation) or the marker lists have not yet been ranked from
  the reference
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MASI
  role: Performs marker_identification, weighted gene2cell score conversion, cal_centainty,
    and parallel MACA label transfer
  repo: https://github.com/hayatlab/MASI
- name: Scanpy
  role: Handles AnnData I/O and preprocessing (pp.highly_variable_genes is skipped
    because only marker genes are used)
- name: MACA
  role: Maps cell-type labels to query data from the MASI score matrix using multiple
    clustering parameters (resolution 3, 5, 7; neighborhood sizes 5, 10, 15)
- name: SCCAF
  role: Used in combination with MASI to identify subtypes when query data has unseen
    cell types
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:4b78f819854f71cd64cf3ed3d2a7817f0081d16de2caff6e3e0ffed4b9434381
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MACA
  - MASI
  - SCCAF
  - Scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-annotation-scoring

## Summary

Converts a gene expression matrix into a cell-type score matrix by ranking reference-derived marker genes, applying MASI's linear weight discount (100% for the 1st marker to 50% for the Nth), and applying a PlinerScore transformation so per-cell scores can be assigned labels and used for integration.

## When to use

Use when you have reference data with cell-type labels ('cell_type' column in AnnData) and need to annotate a query single-cell dataset by scoring cells against ranked marker genes rather than training a model.

## When NOT to use

- When the query dataset contains cell types absent from the reference — MASI alone cannot annotate unseen types; combine with SCCAF for subtype identification
- When reference data quality or resolution is poor — MASI annotation accuracy is dependent on the quality of reference data, so a resolution-matched reference is required
- When inputs are not raw counts suitable for LogNormalization (scale factor 10,000 followed by log-transformation) or the marker lists have not yet been ranked from the reference

## Inputs

- Reference AnnData (raw or LogNormalized counts, cell-type labels in adata.obs['cell_type'])
- Query AnnData expression matrix (cells x genes)
- Ranked cell-type marker list (cell_markers dict) from masi.marker_identification or ensemble aggregation
- Processed single-cell data from the figshare deposit (https://figshare.com/articles/dataset/Fast_model-free_integration_and_transfer_learning_via_MASI_for_single-cell_expression_data/18866264)

## Outputs

- Cell-type score matrix (cells x cell types) from masi.gene2cell (scores, labels)
- Per-cell assigned cell-type labels
- Weighted marker table (cell type, marker gene, rank, weight) e.g. weighted_markers.csv
- Per-cell certainty score table (cell barcode, assigned label, certainty ratio) e.g. certainty_scores.csv
- Unassigned flags for cells with certainty below the 0.5–0.6 threshold

## How to apply

Load the reference AnnData with a 'cell_type' column and run masi.marker_identification to obtain ranked cell_markers (top 20 markers per DE test via robust rank aggregation, or significant markers via Lancaster combination). Attach linearly discounted weights via use_weight=True so the 1st marker contributes 100% of its expression and the Nth contributes 50%, with intermediate markers discounted linearly. Call masi.gene2cell to transform the gene expression matrix into a cell-type score matrix; inside this step, marker expression values below the 0.25 percentile threshold across all cells are suppressed to zero (PlinerScore transformation with TF-IDF). Downstream, feed the scores into masi.parallel for label transfer through MACA, or compute certainty scores C = 1 - D1st/DNth via cal_centainty with a threshold of 0.5–0.6 to flag cells whose type is absent from the reference as unassigned. Validation: weights must lie in [0.5, 1.0] and be monotonically non-increasing with rank; all cells of unseen types should fall below the certainty threshold.

## Related tools

- **MASI** (Performs marker_identification, weighted gene2cell score conversion, cal_centainty, and parallel MACA label transfer) — https://github.com/hayatlab/MASI
- **Scanpy** (Handles AnnData I/O and preprocessing (pp.highly_variable_genes is skipped because only marker genes are used))
- **MACA** (Maps cell-type labels to query data from the MASI score matrix using multiple clustering parameters (resolution 3, 5, 7; neighborhood sizes 5, 10, 15))
- **SCCAF** (Used in combination with MASI to identify subtypes when query data has unseen cell types)

## Examples

```
import MASI as masi; cell_markers = masi.marker_identification(source_data=source); scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True)
```

## Evaluation signals

- weighted_markers.csv exists with marker, rank, and weight fields; all weights fall in 0.5–1.0, equal 1.0 at rank 1, and are monotonically non-increasing with rank
- gene2cell returns both scores and labels objects with one score/label per query cell
- PlinerScore threshold check: marker expression values below the 0.25 percentile across all cells are suppressed to zero before score summation
- Certainty validation: cells whose true cell type is not present in the reference fall below certainty thresholds in [0.5, 0.6] and are labelled unassigned
- certainty_scores.csv contains a certainty ratio column for every cell, where the ratio is distance to closest centroid over distance to least-close centroid

## Limitations

- MASI alone cannot annotate unseen cell types not present in the reference; it must be combined with SCCAF to identify subtypes within major cell types
- Accurate annotation via MASI is dependent on the quality of reference data; choice and resolution of the reference are critical to downstream analysis
- The PlinerScore threshold is described with both a variable 'X-percentile' and a default of 0.25, and the applied value per analysis is not always stated

## Evidence

- [discussion] weighing rule: "the 1st marker in this ranked list will contribute 100% of its expression to the cell-type score of A, while the Nth marker only contributes 50%"
- [discussion] PlinerScore threshold: "The default value for PlinerScore threshold is 0.25 as the percentile threshold"
- [discussion] percentile suppression: "suppressing expression values of a marker gene to zeros if they are below the X-percentile of expression values across all cells"
- [methods] gene2cell call: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [discussion] uncertainty handling: "If query data has unseen cell types not in reference, MASI in combination with SCCAF can be used to identify subtypes within major cell types."
- [discussion] reference dependence: "accurate annotation via MASI is also dependent on the quality of reference data. Thus, the choice and resolution of the reference are critical to downstream analysis."
- [readme] marker identification API: "cell_markers = masi.marker_identification(source_data=source)"
