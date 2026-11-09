---
name: multiple-testing-fdr-correction
description: Use when per-spot significance tests (e.g., Local Moran's I across many
  spots and gene signatures) yield thousands of unadjusted P-values, and hotspots
  must be declared only after multiple-testing correction.
when_to_use_negative:
- When only a single hypothesis is being tested (e.g., one signature's Global Moran's
  I gate itself) — no multiple-testing correction is needed for the GMI > 0.5 declaration
- When the goal is cell subtype annotation or malignant cell identification (predCellType,
  runCellSubtypeClassify, xgboost classifier) rather than spatial significance testing
- When spatial coordinates are unavailable and only per-cell expression annotation
  is required
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scCancer2
  role: R package implementing the spatial module that computes Global/Local Moran's
    I and applies z-score-based correction and FDR adjustment to Local Moran's I P-values
  repo: http://lifeome.net/software/sccancer2/
- name: R
  role: Language in which the scCancer2 spatial structure detection and correction
    pipeline is implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:128748f03cfddf15913eac494998047519a013ded1ab7257f705e58df9949d97
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - scCancer2
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multiple-testing-fdr-correction

## Summary

Statistical correction workflow for controlling false discoveries when many spatial significance tests are run simultaneously, as in scCancer2's spatial structure detection where Local Moran's I P-values are computed per spot per gene signature. It applies z-score-based correction followed by FDR adjustment so only robustly significant spots are reported.

## When to use

Use when per-spot significance tests (e.g., Local Moran's I across many spots and gene signatures) yield thousands of unadjusted P-values, and hotspots must be declared only after multiple-testing correction.

## When NOT to use

- When only a single hypothesis is being tested (e.g., one signature's Global Moran's I gate itself) — no multiple-testing correction is needed for the GMI > 0.5 declaration
- When the goal is cell subtype annotation or malignant cell identification (predCellType, runCellSubtypeClassify, xgboost classifier) rather than spatial significance testing
- When spatial coordinates are unavailable and only per-cell expression annotation is required

## Inputs

- Per-spot Local Moran's I statistics and raw P-values for gene signatures passing the GMI > 0.5 global test
- Per-signature Global Moran's I values computed with inverse-square distance weights between spot pairs
- Spot-level z-scores of signature enrichment scores across the spatial map

## Outputs

- FDR-adjusted Local Moran's I P-values per spot per signature
- Filtered hotspot/coldspot labels per spot after z-score-based false-positive filtering
- Per-signature GMI values and corrected results written to spatial_structure_results.csv

## How to apply

First gate the tests: compute Global Moran's I per gene signature with inverse-square distance weights between spot pairs and only proceed to localization for signatures with GMI > 0.5, since a non-clustered signature produces no meaningful local tests to correct. For signatures passing this gate, compute Local Moran's I at each spot to label hotspot and coldspot regions, then correct the raw per-spot P-values in two stages: a z-score-based correction followed by FDR adjustment. Finally, apply the z-score-based false-positive filtering strategy to retain only robustly significant spots, writing the per-signature GMI values and the filtered per-spot Local Moran's I labels and adjusted P-values to spatial_structure_results.csv. The rationale is that naive per-spot P-values across a spatial map massively inflate false positives, so unfiltered Local Moran's I calls should never be interpreted directly.

## Related tools

- **scCancer2** (R package implementing the spatial module that computes Global/Local Moran's I and applies z-score-based correction and FDR adjustment to Local Moran's I P-values) — http://lifeome.net/software/sccancer2/
- **R** (Language in which the scCancer2 spatial structure detection and correction pipeline is implemented)

## Evaluation signals

- Only signatures with Global Moran's I > 0.5 have Local Moran's I tests and corrections performed; signatures below the threshold should be absent from the localization output
- Adjusted P-values in spatial_structure_results.csv must be monotonically >= their raw counterparts, consistent with FDR adjustment
- Spots retained as hotspot/coldspot must pass the z-score-based false-positive filtering strategy, not merely raw P-value significance
- Output CSV contains both per-signature GMI values and per-spot labels/adjusted P-values in the expected schema

## Limitations

- The Global Moran's I > 0.5 threshold is a hard gate: spatial structures in signatures falling below it are never localized, potentially missing weakly clustered patterns
- The z-score-based false-positive filtering strategy may discard genuine weak signals, as it is designed to retain only robustly significant spots
- The article does not state software version numbers for R or scCancer, affecting reproducibility of the correction pipeline

## Evidence

- [methods] Correct Local Moran's I P-values with z-score-based correction followed by FDR adjustment, and apply the z-score-based false-positive filtering strategy to retain robustly significant spots: [evidence span withheld — anchor status: not_found]
- [abstract] scCancer2 spatial module finding: "scCancer2 integrated a module to process the spatial transcriptomic data and analyze the spatial features of TME"
- [methods] GMI threshold decision point: [evidence span withheld — anchor status: not_found]
- [methods] Output artifact schema: [evidence span withheld — anchor status: not_found]
