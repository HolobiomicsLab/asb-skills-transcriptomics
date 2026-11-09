---
name: spatial-transcriptomics-data-processing
description: Use when spatial transcriptomics data (expression matrix with spot coordinates)
  must be processed to detect and localize TME spatial structures such as TLSs or
  tumor capsules using gene-signature spatial autocorrelation.
when_to_use_negative:
- Input is plain scRNA-seq data without spot coordinates — use the scCancer2 single-cell
  annotation modules (predCellType, runCellSubtypeClassify) instead
- The goal is malignant cell identification from bulk/scRNA-seq rather than spatial
  structure localization — use the xgboost-based malignant classifier module
- No gene signature definitions are available, since per-spot signature enrichment
  scores are the required input to the Moran's I statistics
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCancer2
  role: R package whose spatial module implements signature enrichment scoring, Global/Local
    Moran's I detection, P-value correction, and result export
  repo: czythu/scCancer
- name: R
  role: runtime environment for scCancer2 spatial analyses
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:3451bdcf39d4142ef9492fd8f9ee130a0f11feb6bf4636ef8bfbe1478c35474f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/czythu/scCancer
  tools:
  - R
  - scCancer2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# spatial-transcriptomics-data-processing

## Summary

scCancer2's spatial transcriptomics module processes spot-level expression matrices plus spot coordinates and detects/localizes TME spatial structures (e.g., TLSs, tumor capsules) via Global and Local Moran's I statistics on gene-signature enrichment maps.

## When to use

Use when spatial transcriptomics data (expression matrix with spot coordinates) must be processed to detect and localize TME spatial structures such as TLSs or tumor capsules using gene-signature spatial autocorrelation.

## When NOT to use

- Input is plain scRNA-seq data without spot coordinates — use the scCancer2 single-cell annotation modules (predCellType, runCellSubtypeClassify) instead
- The goal is malignant cell identification from bulk/scRNA-seq rather than spatial structure localization — use the xgboost-based malignant classifier module
- No gene signature definitions are available, since per-spot signature enrichment scores are the required input to the Moran's I statistics

## Inputs

- Spatial transcriptomic expression matrix
- Spot coordinates
- Gene signature definitions (scCancer2 spatial module input files)

## Outputs

- Per-signature Global Moran's I values
- Filtered per-spot Local Moran's I labels and FDR-adjusted P-values
- spatial_structure_results.csv

## How to apply

Load the spatial transcriptomic expression matrix together with spot coordinates and gene signature definitions from the scCancer2 spatial module input files, then compute per-spot enrichment scores for each gene signature across the spatial map. For each signature, calculate Global Moran's I using inverse-square distance weights between spot pairs and declare the signature spatially clustered when GMI > 0.5 — this two-stage design first decides whether a distinct spatial structure exists at all. Only for signatures passing the global test, compute Local Moran's I statistics at each spot to localize hotspot and coldspot regions corresponding to the structure's position. Correct Local Moran's I P-values with z-score-based correction followed by FDR adjustment, and apply a z-score-based false-positive filtering strategy so only robustly significant spots are retained. Write the per-signature GMI values and the filtered per-spot Local Moran's I labels/adjusted P-values to spatial_structure_results.csv.

## Related tools

- **scCancer2** (R package whose spatial module implements signature enrichment scoring, Global/Local Moran's I detection, P-value correction, and result export) — czythu/scCancer
- **R** (runtime environment for scCancer2 spatial analyses)

## Examples

```
suppressMessages(library(scCancer))
```

## Evaluation signals

- Samples are declared to possess distinct spatial structures only when Global Moran's I > 0.5; verify per-signature GMI values against this threshold
- Local Moran's I is computed only for signatures that passed the global test (GMI > 0.5), not for all signatures
- Local Moran's I P-values must be z-score corrected and FDR adjusted before filtering; retained spots are those surviving the z-score-based false-positive filter
- Output file spatial_structure_results.csv exists and contains per-signature GMI values plus filtered per-spot Local Moran's I labels/adjusted P-values

## Limitations

- The Global Moran's I threshold of 0.5 is a fixed declaration cutoff, so borderline structures with GMI just below it will not be localized
- No software version numbers are stated for R, scCancer, Seurat, or xgboost in the article's methods, complicating exact reproduction
- Detection depends on the supplied gene signature definitions; structure calls are only as good as the signature-to-spot enrichment maps

## Evidence

- [abstract] scCancer2 includes a spatial transcriptomics module for TME spatial structure detection: "Thirdly, scCancer2 integrated a module to process the spatial transcriptomic data and analyze the spatial features of TME."
- [readme] scCancer2 was updated on massive single-cell and spatial transcriptome data: [evidence span withheld — anchor status: external_doc]
- [methods] The spatial module is part of the scCancer2 R toolkit loaded in R: "suppressMessages(library(scCancer))
suppressMessages(library(dplyr))"
