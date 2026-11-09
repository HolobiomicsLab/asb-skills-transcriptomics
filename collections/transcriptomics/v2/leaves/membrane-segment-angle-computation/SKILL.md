---
name: membrane-segment-angle-computation
description: Use when a watershed label mask contains cells whose interior was partitioned
  by internal membrane pixel-classification segments — i.e. cells where internal membranes
  occupy more than 10% of the cell area — and single-cell re-segmentation is needed.
when_to_use_negative:
- Cells already cleanly separated by the initial watershed step (no internal membrane
  segments exceeding the 10%-of-cell-area threshold) — re-segmentation would alter
  valid boundaries.
- Pipelines using alternative segmentation strategies such as Voronoi tessellation
  or dilation from nuclei, which the paper identifies as insufficient for columnar
  epithelial cells but which are different methods entirely.
- Datasets without a pixel-classification probability map — the angle computation
  requires membrane segments, not raw intensity images.
tools:
- name: MIRIAM
  role: Implements the internal membrane angle computation, endpoint location, and
    segment extension that re-segment aberrant cells; available in Matlab and Python
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Provides the pixel-classification probability maps of internal membranes that
    the angle computation operates on
  repo: https://www.ilastik.org/download.html
- name: Matlab
  role: One of the two MIRIAM implementations containing this resegmentation algorithm
    (README describes Matlab code for the pipeline)
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative MIRIAM implementation; results are largely similar to the Matlab
    version
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:1cff3576dc5314e959944012ff61623642f77fe4573f35dc2a314c69b0d5f56a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - Ilastik
  - MIRIAM
  - Matlab
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# membrane-segment-angle-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MIRIAM sub-step that computes the angles of internal membrane segments within aberrantly segmented cells, locates their endpoints, and 'connects the dots' by extending segments along those angles until they hit the cell border or another extension, thereby re-segmenting cells whose watershed boundaries were split by internal membranes.

## When to use

Use when a watershed label mask contains cells whose interior was partitioned by internal membrane pixel-classification segments — i.e. cells where internal membranes occupy more than 10% of the cell area — and single-cell re-segmentation is needed.

## When NOT to use

- Cells already cleanly separated by the initial watershed step (no internal membrane segments exceeding the 10%-of-cell-area threshold) — re-segmentation would alter valid boundaries.
- Pipelines using alternative segmentation strategies such as Voronoi tessellation or dilation from nuclei, which the paper identifies as insufficient for columnar epithelial cells but which are different methods entirely.
- Datasets without a pixel-classification probability map — the angle computation requires membrane segments, not raw intensity images.

## Inputs

- Random forest pixel-classification probability maps (e.g. from Ilastik) labeling cell compartments including internal membranes
- Initial watershed label mask from the upstream MIRIAM segmentation stage
- Mucin 2 marker channel/intensity annotations for identifying goblet cells

## Outputs

- Updated full-image label mask with flagged cells re-segmented along internal membrane extensions
- Per-cell segmentation where no flagged cell retains internal membranes over >10% of its area

## How to apply

Run this after MIRIAM's random forest pixel classification (Ilastik probability maps) and initial watershed label mask. First flag candidate cells: those whose internal membrane segments comprise more than 10% of the cell area, plus Mucin 2-expressing goblet cells. For each candidate cell, compute the angles of its internal membrane segments and locate their endpoints to determine extension directions. Iteratively extend each membrane segment from its endpoints along the computed angles until it intersects either the cell border or an extension from another internal membrane, splitting the cell into separate objects. Merge the re-segmented labels back into the full-image mask, replacing the initial watershed boundaries for flagged cells. The Matlab and Python implementations of MIRIAM give largely similar results, so either can be used.

## Related tools

- **MIRIAM** (Implements the internal membrane angle computation, endpoint location, and segment extension that re-segment aberrant cells; available in Matlab and Python) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Provides the pixel-classification probability maps of internal membranes that the angle computation operates on) — https://www.ilastik.org/download.html
- **Matlab** (One of the two MIRIAM implementations containing this resegmentation algorithm (README describes Matlab code for the pipeline)) — github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative MIRIAM implementation; results are largely similar to the Matlab version)

## Evaluation signals

- Output label mask exists and matches the input image dimensions.
- No flagged cell retains internal membrane segments covering more than 10% of its area after re-segmentation.
- Total cell count is preserved for unflagged regions — re-segmentation should only alter cells selected by the >10% internal-membrane rule and Mucin 2 goblet-cell rule.
- Matlab and Python implementations produce largely similar results (per Supplemental Figure 6), so cross-implementation diffs should be minor.
- Behavioral check against the Supplemental Fig. 2 flowchart, since no standalone benchmark is reported for this sub-step.

## Limitations

- Validation is behavioral (Supplemental Fig. 2 flowchart); no standalone benchmark for the angle-extension resegmentation step is reported.
- The README's example data omits image stacks (too large), so users must regenerate or download them separately (Dropbox link) before running the pipeline.
- No software version numbers are stated for the Matlab or Python implementations, and deep-learning-based nuclei detection was not implemented in the pipeline.

## Evidence

- [other] The resegmentation algorithm 'connects the dots' by locating the endpoints of internal membrane segments within a cell and extending them until they intersect either the cell border or an extension from another internal membrane: [evidence span withheld — anchor status: not_found]
- [other] Identify cells containing internal membrane segments comprising more than 10% of the cell area, and additionally include Mucin 2-expressing goblet cells: [evidence span withheld — anchor status: not_found]
