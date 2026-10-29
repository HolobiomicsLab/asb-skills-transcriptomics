---
name: spatial-autocorrelation-testing
description: Use when you have spot-level coordinates and per-spot gene signature
  enrichment scores from a spatial transcriptomics map and need to detect whether
  TME signatures are spatially organized and where those structures lie.
when_to_use_negative:
- Data have no spatial coordinates (e.g. bulk RNA-seq or dissociated scRNA-seq), so
  no inverse-square distance weights between spots can be computed
- You need de novo structure discovery without predefined gene signatures — this skill
  tests clustering of specific signatures' enrichment scores, not unsupervised spatial
  domain segmentation
- Only a handful of spots are available, making global autocorrelation statistics
  unstable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scCancer2
  role: R package whose spatial module implements the two-stage Global/Local Moran's
    I detection of TME spatial structures
  repo: https://doi.org/10.5281/zenodo.10477297
- name: R
  role: Implementation environment for the scCancer2 spatial workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:1e58a0e16222f33dd0214748122b09208d5cd915436233cd6c4d29008e5c2256
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - scCancer2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spatial-autocorrelation-testing

## Summary

Two-stage spatial autocorrelation testing from scCancer2: Global Moran's I (with inverse-square distance weights) flags gene signatures as spatially clustered (GMI > 0.5), then Local Moran's I localizes hotspot/coldspot structures such as TLSs and tumor capsules in spatial transcriptomics data.

## When to use

Use when you have spot-level coordinates and per-spot gene signature enrichment scores from a spatial transcriptomics map and need to detect whether TME signatures are spatially organized and where those structures lie.

## When NOT to use

- Data have no spatial coordinates (e.g. bulk RNA-seq or dissociated scRNA-seq), so no inverse-square distance weights between spots can be computed
- You need de novo structure discovery without predefined gene signatures — this skill tests clustering of specific signatures' enrichment scores, not unsupervised spatial domain segmentation
- Only a handful of spots are available, making global autocorrelation statistics unstable

## Inputs

- Spatial transcriptomic expression matrix (spot x gene)
- Spot coordinates on the spatial map
- Gene signature definitions (scCancer2 spatial module input files)
- Per-spot enrichment scores per gene signature

## Outputs

- Per-signature Global Moran's I values (GMI > 0.5 flags a clustered signature)
- Per-spot Local Moran's I labels identifying hotspot/coldspot regions
- FDR-adjusted Local Moran's I P-values after z-score-based correction
- spatial_structure_results.csv

## How to apply

Load the spatial expression matrix, spot coordinates, and gene signature definitions (e.g. from the scCancer2 spatial module input files), and compute per-spot enrichment scores for each signature across the spatial map. For each signature, calculate Global Moran's I using inverse-square distance weights between spot pairs; declare the signature spatially clustered when GMI > 0.5. For signatures passing the global test, compute Local Moran's I statistics at every spot to localize hotspot and coldspot regions. Correct Local Moran's I P-values with z-score-based correction followed by FDR adjustment, then apply the z-score-based false-positive filtering strategy to retain only robustly significant spots. Write the per-signature GMI values and filtered per-spot Local Moran's I labels and adjusted P-values to spatial_structure_results.csv.

## Related tools

- **scCancer2** (R package whose spatial module implements the two-stage Global/Local Moran's I detection of TME spatial structures) — https://doi.org/10.5281/zenodo.10477297
- **R** (Implementation environment for the scCancer2 spatial workflow)

## Examples

```
suppressMessages(library(scCancer)); suppressMessages(library(dplyr))
```

## Evaluation signals

- Each signature carries a computed Global Moran's I value; only signatures with GMI > 0.5 proceed to Local Moran's I localization
- Local Moran's I P-values have been z-score-corrected and FDR-adjusted before thresholding
- Per-spot labels survive the z-score-based false-positive filtering step, so only robustly significant spots remain
- Output file spatial_structure_results.csv exists with per-signature GMI values and filtered per-spot Local Moran's I labels/adjusted P-values
- Spatial hotspot/coldspot regions recovered for known structures (e.g. TLSs, tumor capsules) coincide with their expected locations on the spatial map

## Limitations

- The GMI > 0.5 threshold is a fixed decision rule from scCancer2; signatures near the cutoff may be missed or falsely flagged and no per-tissue calibration is described
- Detection is two-stage and signature-driven: signatures failing the global test receive no localization, and structures not captured by the provided gene signatures are invisible
- No software version numbers for R or scCancer2 are stated in the article, complicating reproducibility

## Evidence

- [abstract] Thirdly, scCancer2 integrated a module to process the spatial transcriptomic data and analyze the spatial features of TME.: "Thirdly, scCancer2 integrated a module to process the spatial transcriptomic data and analyze the spatial features of TME."
- [results] it declares samples with Global Moran's I > 0.5 to possess distinct spatial structures, then computes Local Moran's I to determine the position: [evidence span withheld — anchor status: not_found]
- [readme] We updated our R toolkit, `scCancer`, based on massive single-cell transcriptome and spatial transcriptome data.: [evidence span withheld — anchor status: external_doc]
