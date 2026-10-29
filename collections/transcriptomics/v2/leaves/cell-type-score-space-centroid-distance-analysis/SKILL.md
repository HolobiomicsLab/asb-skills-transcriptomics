---
name: cell-type-score-space-centroid-distance-analysis
description: 'Use when transferring labels from a reference atlas with MASI and query
  data may contain cell types not present in the reference: cells whose closest-centroid
  distance approaches their farthest-centroid distance (ratio near 1, certainty near
  0) should be flagged as unassigned.'
when_to_use_negative:
- The query dataset is known to contain only cell types present in the reference —
  the certainty score only needs to flag unseen types
- The goal is to discover subtypes within major cell types rather than flag unassigned
  cells — the article instead recommends combining MASI with SCCAF for that
- Input is raw counts without LogNormalization — MASI's score conversion expects normalized
  expression (LogNormalized, 10,000 scale factor followed by log-transformation)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MASI
  role: Provides gene2cell (gene-to-cell-type-score conversion with use_weight=True),
    cal_centainty (centroid-distance certainty ratios), and marker_identification
    used to build the reference marker dict
  repo: https://github.com/hayatlab/MASI
- name: Scanpy
  role: Handles the AnnData objects that carry the query expression matrix and cell
    metadata into MASI's pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:c22fce12cc068eb5f1f190523cf9a501bc18bb46d4d27805752284e760e953e7
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
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-type-score-space-centroid-distance-analysis

## Summary

Flags cells whose true cell type is absent from the reference by computing MASI's certainty score C = 1 - D1st/DNth — the ratio of each cell's distance to the closest versus least-close reference cell-type centroid in the cell-type score space produced by masi.gene2cell — and labelling cells below a chosen threshold as unassigned.

## When to use

Use when transferring labels from a reference atlas with MASI and query data may contain cell types not present in the reference: cells whose closest-centroid distance approaches their farthest-centroid distance (ratio near 1, certainty near 0) should be flagged as unassigned.

## When NOT to use

- The query dataset is known to contain only cell types present in the reference — the certainty score only needs to flag unseen types
- The goal is to discover subtypes within major cell types rather than flag unassigned cells — the article instead recommends combining MASI with SCCAF for that
- Input is raw counts without LogNormalization — MASI's score conversion expects normalized expression (LogNormalized, 10,000 scale factor followed by log-transformation)

## Inputs

- Processed single-cell query dataset as an AnnData object (e.g. from the figshare deposit 18866264)
- Reference AnnData with cell-type labels in a 'cell_type' obs column
- Cell-type marker dictionary from masi.marker_identification on the reference
- Certainty threshold(s), e.g. 0.5 and 0.6

## Outputs

- certainty_scores.csv: per-cell table with cell barcode, assigned label, and certainty ratio (D1st/DNth)
- Per-cell unassigned flag for cells whose certainty falls below the chosen threshold

## How to apply

Load the processed single-cell dataset (e.g. the figshare AnnData deposit for the MASI paper) and identify cell-type markers from the labeled reference with masi.marker_identification. Convert the query's gene expression matrix to a cell-type score matrix with masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True), applying MASI's linear marker discount (100% for the 1st marker to 50% for the Nth) and the 0.25-percentile PlinerScore suppression of sub-threshold marker expression. Compute reference cell-type centroids in this score space and run MASI's cal_centainty to obtain, per cell, the ratio of distance to the closest centroid over distance to the least-close centroid. Write the per-cell table (cell barcode, assigned label, certainty ratio) to certainty_scores.csv, then apply a threshold — 0.5 and 0.6 were validated on the Oetjen et al. data — to flag cells with low certainty as unassigned. Rationale: a query cell of a type unseen in the reference sits nearly equidistant from all centroids, so its D1st/DNth ratio approaches 1 and its certainty C = 1 - D1st/DNth drops toward 0.

## Related tools

- **MASI** (Provides gene2cell (gene-to-cell-type-score conversion with use_weight=True), cal_centainty (centroid-distance certainty ratios), and marker_identification used to build the reference marker dict) — https://github.com/hayatlab/MASI
- **Scanpy** (Handles the AnnData objects that carry the query expression matrix and cell metadata into MASI's pipeline)

## Examples

```
scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True)
```

## Evaluation signals

- All cells whose true cell type is absent from the reference have certainty ratios below both tested thresholds (0.5 and 0.6) and are labelled unassigned
- certainty_scores.csv exists with a certainty ratio entry for every cell (no missing barcodes)
- Every certainty ratio lies in [0, 1], since it is a closest-centroid distance divided by a least-close centroid distance
- Assigned labels in the output match the labels returned by masi.gene2cell for the same AnnData
- Cells whose true type IS present in the reference score above the threshold and are not flagged unassigned

## Limitations

- MASI alone cannot annotate unseen cell types not present in the reference; the article suggests combining MASI with SCCAF to identify subtypes within major cell types
- The 0.5–0.6 threshold range was validated for retrieving the 4 extra cell types in Oetjen et al. with decent accuracy; other datasets may require re-tuning the threshold
- Accurate annotation via MASI is dependent on the quality of reference data; the choice and resolution of the reference are critical to downstream analysis

## Evidence

- [discussion] Unseen-type detection rationale: "If query data has unseen cell types not in reference, MASI in combination with SCCAF can be used to identify subtypes within major cell types."
- [methods] Score-matrix conversion step: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [discussion] Reference-quality dependency: "accurate annotation via MASI is also dependent on the quality of reference data. Thus, the choice and resolution of the reference are critical to downstream analysis."
- [readme] Marker input format: "##source is Anndata of reference data that has cell-type labels named 'cell_type'"
