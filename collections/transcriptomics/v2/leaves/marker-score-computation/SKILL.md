---
name: marker-score-computation
description: Use when a pre-annotated single-cell RNA-seq reference dataset (e.g.
  .h5ad count matrix with cell type labels) must be converted into per-cell-type marker
  gene lists and occurrence-frequency statistics before MarkerCount-based annotation.
when_to_use_negative:
- Only an external marker database (e.g. Panglao DB, CellMarker DB) is available with
  no annotated reference dataset — use marker-based mode (MarkerCount/HiCAT) instead
  of deriving markers from reference data
- The reference cell labels are unreliable or absent, since the marker score depends
  on the pre-annotated target vs non-target cell split
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MarkerCount
  role: Reference-based cell type identifier whose MarkerCount_Ref() function consumes
    the learned marker lists and thresholds; the scoring workflow mirrors its 'Finding
    Markers From Reference Datasets' step
  repo: https://github.com/combio-dku/MarkerCount
- name: HiCAT
  role: Improved marker-based successor released after MarkerCount; relevant only
    when external marker lists are used instead of reference-derived markers
  repo: https://github.com/combio-dku/HiCAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:4477d8fba3785b2dc11f15a390332b4595cd5b17f03dbc657dae3c9d6414bb59
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/combio-dku/MarkerCount
  repo_url: https://github.com/combio-dku/MarkerCount
  tools:
  - HiCAT
  - MarkerCount
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# marker-score-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs MarkerCount's reference-phase marker discovery: binarize expression, score genes by target vs non-target occurrence, and select the top 18 markers per cell type. This produces the marker lists and rejection thresholds consumed by test-phase marker counting.

## When to use

Use when a pre-annotated single-cell RNA-seq reference dataset (e.g. .h5ad count matrix with cell type labels) must be converted into per-cell-type marker gene lists and occurrence-frequency statistics before MarkerCount-based annotation.

## When NOT to use

- Only an external marker database (e.g. Panglao DB, CellMarker DB) is available with no annotated reference dataset — use marker-based mode (MarkerCount/HiCAT) instead of deriving markers from reference data
- The reference cell labels are unreliable or absent, since the marker score depends on the pre-annotated target vs non-target cell split

## Inputs

- Pre-annotated reference gene expression count matrix (e.g. .h5ad) with per-cell cell type labels
- Hyper-parameters: occurrence frequency threshold fth=0.9, score exponent n=2, number of markers N=18, target FPR p

## Outputs

- Per-cell-type marker gene lists (up to 18 markers each, overlaps allowed)
- Target and non-target occurrence frequencies per marker
- Learned rejection thresholds tm within [0,1] of normalized marker counts (markers_and_thresholds file)

## How to apply

Load the pre-annotated reference gene expression matrix and binarize it, setting bij=1 if xij>0, else 0. For each annotated cell type m, pre-filter candidate genes requiring target-cell occurrence frequency (1/Cm)Σbij >= fth=0.9, then compute the marker score sm,j = (1/Cm)Σbij * (1 - (1/C̄m)Σbij)^n with exponent n=2. Sort sm,j descending per cell type and select the top N=18 genes as markers (N=18 chosen as best-performing across the paper's datasets; overlaps between cell types are allowed). Compute normalized marker counts yi,m on reference cells and sort them descending to find the minimum threshold tm such that FPR(t) <= p via the non-parametric true-positive-set intersection definition. Save per-cell-type marker lists, target/non-target occurrence frequencies, and learned thresholds for the test-phase identifier (MarkerCount_Ref).

## Related tools

- **MarkerCount** (Reference-based cell type identifier whose MarkerCount_Ref() function consumes the learned marker lists and thresholds; the scoring workflow mirrors its 'Finding Markers From Reference Datasets' step) — https://github.com/combio-dku/MarkerCount
- **HiCAT** (Improved marker-based successor released after MarkerCount; relevant only when external marker lists are used instead of reference-derived markers) — https://github.com/combio-dku/HiCAT

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref; df_res = mkrcnt$MarkerCount_Ref(..arguments..)  # R: mkrcnt <- import("MarkerCount.marker_count")
```

## Evaluation signals

- markers_and_thresholds output file exists and contains an entry for every reference cell type
- Each cell type has up to 18 markers, with overlaps between cell types permitted
- Every learned rejection threshold tm lies within the [0,1] range of normalized marker counts
- Every retained marker has target-cell occurrence frequency >= fth=0.9 after the pre-filter
- Recomputing scores with sm,j = (1/Cm)Σbij * (1 - (1/C̄m)Σbij)^2 on binarized data reproduces the top-18 ranking

## Limitations

- Identification through normalized marker counts alone yields considerable errors for closely related sub-populations (beta/delta/epsilon/gamma pancreas cells, dendritic cells in blood, B cells in tumors, T cells in lung), so thresholds must be conservative
- Proper identification requires well-clustered cell populations, which is not always the case
- Cell type taxonomy is not standardized, so the reference annotation itself drives marker quality

## Evidence
