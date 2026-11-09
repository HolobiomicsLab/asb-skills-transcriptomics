---
name: tfidf-gene-expression-transformation
description: Use when a LogNormalized single-cell expression matrix (AnnData) and
  a ranked marker list must be converted into a per-cell, per-reference-cell-type
  score matrix for marker-based label transfer in MASI.
when_to_use_negative:
- Query data contains unseen cell types not present in the reference — MASI (and hence
  this score conversion) alone cannot annotate those; combine MASI with SCCAF instead
- Input matrix is raw counts rather than LogNormalized data (scale factor 10,000 +
  log-transformation) — the PlinerScore/TF-IDF transformation presumes normalized
  input
- The goal is a trained probabilistic or neural-network annotation (e.g. scNym, scArches,
  Symphony reference mapping) rather than fast model-free marker-based scoring
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MASI
  role: Provides masi.gene2cell, which implements the PlinerScore TF-IDF transformation,
    0.25-percentile thresholding, and weighted marker summation to build the cell-type
    score matrix
  repo: https://github.com/hayatlab/MASI
- name: Scanpy
  role: Used to load the LogNormalized expression matrix into the AnnData object passed
    to gene2cell
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:7217fc1e3dd6ca6af84e31b0747d61f3748c16e4c9e39de915f375c6fffbc737
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# tfidf-gene-expression-transformation

## Summary

The PlinerScore transformation inside MASI's gene2cell step converts a LogNormalized gene expression matrix into a cell-type score matrix via TF-IDF normalization, thresholding, and weighted summation of marker genes, enabling fast model-free cell-type annotation.

## When to use

Use when a LogNormalized single-cell expression matrix (AnnData) and a ranked marker list must be converted into a per-cell, per-reference-cell-type score matrix for marker-based label transfer in MASI.

## When NOT to use

- Query data contains unseen cell types not present in the reference — MASI (and hence this score conversion) alone cannot annotate those; combine MASI with SCCAF instead
- Input matrix is raw counts rather than LogNormalized data (scale factor 10,000 + log-transformation) — the PlinerScore/TF-IDF transformation presumes normalized input
- The goal is a trained probabilistic or neural-network annotation (e.g. scNym, scArches, Symphony reference mapping) rather than fast model-free marker-based scoring

## Inputs

- Processed LogNormalized gene expression matrix in AnnData format (counts divided by cell total, scaled by 10,000, log-transformed), e.g. from the MASI figshare deposit
- Ranked cell-type marker list/dict with linear weight discounting (100% for 1st marker to 50% for Nth marker), produced by masi.marker_identification

## Outputs

- Cell-type score matrix with one row per cell and one column per reference cell type (cell_type_scores.csv)
- Assigned cell-type labels per cell, valid inputs for masi.parallel(scores=scores, labels=labels, batch_size=50000)

## How to apply

Load the processed LogNormalized expression matrix (counts divided by cell total, scaled by 10,000, log-transformed) as AnnData and load the ranked cell-type marker list (linear weight discount from 100% for the 1st marker to 50% for the Nth marker). Call masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True), which applies TF-IDF normalization then suppresses marker expression values below the 0.25 percentile threshold across all cells to zero (PlinerScore). For every cell, sum the weighted marker expression per cell type to obtain raw scores and assigned labels. Write the per-dataset score matrix to cell_type_scores.csv and pass scores/labels to masi.parallel (batch_size=50000) for downstream annotation.

## Related tools

- **MASI** (Provides masi.gene2cell, which implements the PlinerScore TF-IDF transformation, 0.25-percentile thresholding, and weighted marker summation to build the cell-type score matrix) — https://github.com/hayatlab/MASI
- **Scanpy** (Used to load the LogNormalized expression matrix into the AnnData object passed to gene2cell)

## Examples

```
scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True)
```

## Evaluation signals

- cell_type_scores.csv exists and has one row per cell and one column per reference cell type
- Returned scores and labels are accepted downstream by masi.parallel(scores=scores, labels=labels, batch_size=50000) without error
- Marker expression values below the 0.25 percentile across all cells are zeroed before summation (verify thresholding was applied)
- Weights follow the linear discount: 1st marker contributes 100% of its expression to the cell-type score, Nth marker contributes 50%
- Scores are per-cell sums of weighted marker expression for each reference cell type, suitable for visualization and downstream annotation

## Limitations

- MASI annotation accuracy depends on the quality of the reference data; choice and resolution of the reference are critical to downstream analysis
- MASI alone cannot annotate unseen cell types not in the reference; it must be combined with SCCAF to identify subtypes within major cell types
- Reproducibility gap: the PlinerScore threshold is described with both a variable 'X-percentile' and a default of 0.25; the applied value per analysis is not stated, and no software version numbers are reported

## Evidence

- [discussion] PlinerScore 0.25 default threshold: "The default value for PlinerScore threshold is 0.25 as the percentile threshold"
- [discussion] Suppressing low-expression markers to zero: "suppressing expression values of a marker gene to zeros if they are below the X-percentile of expression values across all cells"
- [discussion] Marker weight discounting: "the 1st marker in this ranked list will contribute 100% of its expression to the cell-type score of A, while the Nth marker only contributes 50%"
- [methods] gene2cell invocation: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [readme] README gene-to-score step: "##step 1 transform gene expression matrix to cell-type score matrix"
