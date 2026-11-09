---
name: ranked-list-linear-discounting
description: Use when you have a ranked, per-cell-type marker list (e.g., MASI's cell_markers
  dict from marker_identification) and need to weight markers by rank before summing
  marker expression into cell-type scores.
when_to_use_negative:
- Input markers are not ranked per cell type (e.g., only a flat marker set) — the
  discount scheme requires an explicit rank order per cell type
- Only a single marker per cell type is available — with N=1 the 100%-to-50% endpoints
  are degenerate and weighting has no effect
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MASI
  role: Implements the ranked-list weighting via masi.gene2cell with use_weight=True,
    converting the gene expression matrix to a cell-type score matrix
  repo: https://github.com/hayatlab/MASI
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:ac12eef19eb9992b5871fdd3e9207b1277e2f83d092c99a81f8a68228ea660ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MASI
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Ranked-list linear discounting

## Summary

Applies MASI's linear marker-weighing scheme, in which the 1st marker in a cell type's ranked marker list contributes 100% of its expression to the cell-type score and the Nth marker contributes 50%, with intermediate markers discounted linearly. This weighting is the bridge from robust-rank-aggregated marker lists to MASI's cell-type score matrix.

## When to use

Use when you have a ranked, per-cell-type marker list (e.g., MASI's cell_markers dict from marker_identification) and need to weight markers by rank before summing marker expression into cell-type scores.

## When NOT to use

- Input markers are not ranked per cell type (e.g., only a flat marker set) — the discount scheme requires an explicit rank order per cell type
- Only a single marker per cell type is available — with N=1 the 100%-to-50% endpoints are degenerate and weighting has no effect

## Inputs

- Ranked per-cell-type marker list (cell_markers dict) from masi.marker_identification
- Gene expression matrix in AnnData format (LogNormalized: total-count normalized with scale factor 10,000 followed by log-transformation)

## Outputs

- Cell-type score matrix (scores) with per-cell, per-cell-type scores
- Cell-type labels (labels) used for downstream label transfer via MACA
- Optionally a weighted marker table (cell type, marker gene, rank, weight) e.g. weighted_markers.csv

## How to apply

Load the ranked marker list (cell_markers) produced by MASI's marker_identification, which compiles the top 20 marker genes per differential-expression test via robust rank aggregation. For each cell type's ranked list, compute linearly discounted weights: the 1st-ranked marker gets weight 1.0, the Nth gets 0.5, and intermediate markers are discounted linearly between these endpoints. Attach the weights via the use_weight=True pathway of masi.gene2cell, which converts the gene expression matrix to a cell-type score matrix by summing weighted marker expression per cell type. The rationale is that top-ranked markers are more reliable cell-type signatures, so their expression should dominate the score, while lower-ranked markers still contribute half weight. The resulting cell-type score matrix is the critical component for MASI's downstream annotation (via MACA) and integrative analyses.

## Related tools

- **MASI** (Implements the ranked-list weighting via masi.gene2cell with use_weight=True, converting the gene expression matrix to a cell-type score matrix) — https://github.com/hayatlab/MASI

## Examples

```
import MASI as masi; scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True)
```

## Evaluation signals

- All weight values fall in the range 0.5–1.0
- Weights are monotonically non-increasing with rank
- The rank-1 marker for each cell type has weight exactly 1.0
- The last (Nth) ranked marker for each cell type has weight 0.5
- The resulting cell-type score matrix (from masi.gene2cell) contains one score per cell for every cell type present in cell_markers

## Limitations

- The PlinerScore transformation downstream uses a percentile threshold (default 0.25) whose applied value per analysis is not consistently stated in the paper, affecting reproducibility of the final scores
- Annotation accuracy downstream depends on reference data quality; the choice and resolution of the reference are critical
- The exact list length N that defines the 50% endpoint depends on the upstream aggregation (top 20 markers per DE test), so weights differ if marker list length changes

## Evidence

- [discussion] Linear discount endpoints: "the 1st marker in this ranked list will contribute 100% of its expression to the cell-type score of A, while the Nth marker only contributes 50%"
- [methods] gene2cell invocation: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [discussion] Ranked list provenance: "the top 20 marker genes from each DE test were compiled together"
- [discussion] Thresholding before score conversion: "suppressing expression values of a marker gene to zeros if they are below the X-percentile of expression values across all cells"
