---
name: marker-gene-weight-assignment
description: Use when you have an upstream ranked marker list (e.g., MASI's marker_identification
  output) per cell type and need rank-discounted marker weights before computing a
  cell-type score matrix for annotation or integration.
when_to_use_negative:
- Input is an unranked or significant-marker-only marker set with no per-cell-type
  rank ordering, since the linear discount is defined over rank position N
- Query data contains cell types absent from the reference; MASI weighting cannot
  annotate unseen cell types (combine MASI with SCCAF instead)
- The reference data are low quality or poorly resolved — annotation accuracy depends
  critically on reference quality and resolution
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MASI
  role: Provides masi.gene2cell (use_weight=True) to apply the linear rank discount
    to cell_markers and produce the cell-type score matrix
  repo: github.com/hayatlab/MASI
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:e6d85f067a020a7a84db73b75d36d24ec96cfba2414748603a9fee786932111a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MASI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# marker-gene-weight-assignment

## Summary

Assigns linearly discounted weights to a ranked cell-type marker list following MASI's weighing scheme, where the 1st-ranked marker contributes 100% and the Nth-ranked marker contributes 50% of its expression to the cell-type score. These weighted markers drive MASI's conversion of the gene expression matrix into the cell-type score matrix used for annotation and integration.

## When to use

Use when you have an upstream ranked marker list (e.g., MASI's marker_identification output) per cell type and need rank-discounted marker weights before computing a cell-type score matrix for annotation or integration.

## When NOT to use

- Input is an unranked or significant-marker-only marker set with no per-cell-type rank ordering, since the linear discount is defined over rank position N
- Query data contains cell types absent from the reference; MASI weighting cannot annotate unseen cell types (combine MASI with SCCAF instead)
- The reference data are low quality or poorly resolved — annotation accuracy depends critically on reference quality and resolution

## Inputs

- Ranked cell-type marker list (cell_markers dict of cell type -> ranked marker genes) from MASI marker_identification or ensemble rank aggregation
- LogNormalized gene expression matrix in AnnData (.h5ad) format (raw counts normalized to 10,000 per cell then log-transformed)

## Outputs

- Cell-type score matrix (scores) and per-cell label assignments (labels) from masi.gene2cell
- Weighted marker table (cell type, marker gene, rank, weight), e.g., weighted_markers.csv

## How to apply

Load the ranked marker list (cell_markers) produced by MASI's marker_identification/ensemble aggregation for each cell type. For each cell type's ranked list, compute linearly discounted weights under MASI's weighing rule: the 1st marker contributes 100% (weight 1.0), the Nth marker contributes 50% (weight 0.5), and intermediate markers are discounted linearly between these endpoints (equivalent to weight = 1 - (i/N)*(1/2) at rank i). Attach the weight to each (cell type, marker, rank) record via the use_weight=True pathway of masi.gene2cell, which then transforms the LogNormalized gene expression matrix (scale factor 10,000, log-transformed) into the cell-type score matrix by summing weighted marker expression. Optionally write the weighted marker table (cell type, marker gene, rank, weight) to weighted_markers.csv for auditing. Rationale: rank discounting ensures high-confidence (top-ranked) markers dominate the cell-type score, improving downstream label transfer through MACA.

## Related tools

- **MASI** (Provides masi.gene2cell (use_weight=True) to apply the linear rank discount to cell_markers and produce the cell-type score matrix) — github.com/hayatlab/MASI

## Examples

```
import MASI as masi; scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True)
```

## Evaluation signals

- weighted_markers.csv exists and contains marker gene, rank, and weight fields
- All weight values fall in the range 0.5–1.0
- Weights are monotonically non-increasing with rank
- Weight equals exactly 1.0 at rank 1 and 0.5 at rank N (the last marker)
- Downstream scores/labels from masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True) are produced for every cell type in cell_markers

## Limitations

- The PlinerScore threshold applied after scoring is described with both a variable 'X-percentile' and a default of 0.25; the exact per-analysis value is not stated, complicating reproduction of downstream scores
- The weighting scheme is only meaningful for ranked marker lists; MASI's annotation accuracy also depends on the quality and resolution of the reference data used to derive the markers
- No software version numbers are reported in the article for MASI or related tools, so exact behavior of the weighting implementation may vary across releases

## Evidence

- [discussion] linear discount endpoints: "the 1st marker in this ranked list will contribute 100% of its expression to the cell-type score of A, while the Nth marker only contributes 50%"
- [methods] use_weight pathway: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [discussion] rank aggregation upstream: "the top 20 marker genes from each DE test were compiled together"
- [discussion] PlinerScore threshold downstream: "The default value for PlinerScore threshold is 0.25 as the percentile threshold"
- [readme] marker identification input: "cell_markers = masi.marker_identification(source_data=source)"
